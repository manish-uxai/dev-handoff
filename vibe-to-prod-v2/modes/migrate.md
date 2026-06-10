# Mode: JSX to TSX Migration

> Load this file when the user confirms they want to migrate their JavaScript React codebase to TypeScript. Follow these steps in order. Do not skip steps. Do not introduce `any` types as a shortcut.

---

## Pre-migration checklist

- [ ] User has explicitly confirmed they want to migrate
- [ ] Run `git status` — working tree must be clean before starting
- [ ] Note existing JSDoc typedefs in `domain.js` — these become TypeScript interfaces directly
- [ ] Check if TypeScript is already installed: `cat package.json | grep typescript`

---

## Step 1: Install TypeScript

```bash
npm install --save-dev typescript @types/react @types/react-dom @types/node
```

---

## Step 2: Create `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["src"]
}
```

---

## Step 3: Update Vite config

Rename `vite.config.js` → `vite.config.ts`:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
});
```

---

## Step 4: Migrate `domain.js` → `domain.ts`

Every JSDoc typedef becomes a TypeScript interface.

```js
// Before
/** @typedef {Object} Patient @property {string} id @property {string} name */
```
```ts
// After
export interface Patient {
  id: string;
  name: string;
}

export interface ApiResponse<T> {
  data: T;
  meta: { total?: number; page?: number };
}
```

Rules:
- `@property {string} [field]` (optional) → `field?: string`
- Every field the UI actually uses must be present — fill gaps now
- Delete `domain.js` after `domain.ts` is complete

---

## Step 5: Migrate `api/index.js` → `api/index.ts`

Add return types using interfaces from `domain.ts`:

```ts
// @backend GET /api/patients
export async function getPatients(): Promise<ApiResponse<Patient[]>> {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

---

## Step 6: Rename component files one at a time

`Component.jsx` → `Component.tsx`. Start with leaf components (no children), work upward.

**Do not rename everything at once.** Fix TypeScript errors per file before moving to the next.

---

## Step 7: Add prop interfaces to every component

```tsx
interface PatientCardProps {
  patient: Patient;
  onSelect: (id: string) => void;
  children?: React.ReactNode;
}

function PatientCard({ patient, onSelect }: PatientCardProps) { ... }
```

Never use `any` as a prop type. Use `unknown` and narrow it if needed.

---

## Step 8: Type React hooks

```ts
const [patients, setPatients] = useState<Patient[]>([]);
const inputRef = useRef<HTMLInputElement>(null);
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => { ... };
```

---

## Step 9: Convert relative imports to path aliases

```ts
// Before
import { Patient } from '../../types/domain';

// After
import { Patient } from '@/domain';
```

---

## Step 10: Add Zod schemas

```ts
// src/schemas/patient.ts
import { z } from 'zod';
export const PatientSchema = z.object({
  id: z.string(),
  name: z.string(),
  status: z.enum(['active', 'inactive']),
});
export type Patient = z.infer<typeof PatientSchema>;
```

---

## Step 11: Verify

```bash
npx tsc --noEmit
```

Fix every error. No `// @ts-ignore`. No `as any`.

| Error | Fix |
| :--- | :--- |
| Object is possibly undefined | Add `?.` optional chaining |
| Type string not assignable to never | Add explicit type to `useState` |
| Parameter implicitly has any type | Add explicit parameter type |

---

## Step 12: Run the app

```bash
npm run dev
```

The app must look and behave identically to before migration. If anything changed visually, a bug was introduced — find and fix it before running the 19-dimension audit.

---

## Hard rules

- No `any` types. Ever.
- No `// @ts-ignore`.
- No mass renaming then fixing — one file at a time.
- No new features during migration — migration only.
