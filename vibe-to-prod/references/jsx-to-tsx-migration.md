# JSX to TSX Migration Guide

> Read this file when the user has confirmed they want to migrate their JavaScript React codebase to TypeScript before handoff. Follow these steps in order. Do not skip steps. Do not introduce `any` types as a shortcut — a migration full of `any` is worse than staying in JavaScript.

---

## Pre-migration checklist

Before writing a single line of TypeScript:

- [ ] Confirm the user wants to migrate (don't assume from stack detection alone)
- [ ] Check `package.json` — TypeScript may already be installed
- [ ] Run `git status` — ensure the working tree is clean so the migration is its own commit
- [ ] Note any existing JSDoc typedefs in `domain.js` — these become TypeScript interfaces directly

---

## Step 1: Install TypeScript

```bash
npm install --save-dev typescript @types/react @types/react-dom
```

If the project uses Vite (check `vite.config.js`):
```bash
npm install --save-dev @types/node
```

---

## Step 2: Create `tsconfig.json`

Create at the project root:

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
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

If using Vite, also create `tsconfig.node.json`:

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

---

## Step 3: Update Vite config

Rename `vite.config.js` → `vite.config.ts` and add path alias:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

---

## Step 4: Migrate `domain.js` → `domain.ts`

This is the most important file. Every JSDoc typedef becomes a TypeScript interface.

**Before (JSDoc):**
```js
/**
 * @typedef {Object} Patient
 * @property {string} id
 * @property {string} name
 * @property {'active' | 'inactive'} status
 * @property {string} [notes]
 */
```

**After (TypeScript):**
```ts
export interface Patient {
  id: string;
  name: string;
  status: 'active' | 'inactive';
  notes?: string;
}
```

Rules for this step:
- Every `@typedef` becomes an `export interface`
- `@property {string} [field]` (optional) becomes `field?: string`
- Union types like `'active' | 'inactive'` stay identical
- If a typedef is missing fields that the UI actually uses, add them now — don't carry incomplete types into TypeScript
- Delete the original `domain.js` after `domain.ts` is complete

---

## Step 5: Migrate `api.js` → `api.ts`

Add return types to every function using the interfaces from `domain.ts`.

**Before:**
```js
// @backend GET /api/patients
export async function getPatients() {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

**After:**
```ts
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[]; meta: { total: number } }
export async function getPatients(): Promise<ApiResponse<Patient[]>> {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

Add the `ApiResponse` generic type to `domain.ts`:
```ts
export interface ApiResponse<T> {
  data: T;
  meta: {
    total?: number;
    page?: number;
    [key: string]: unknown;
  };
}
```

---

## Step 6: Rename component files

Rename files in batches, not all at once:
- `Component.jsx` → `Component.tsx`
- `hook.js` → `hook.ts` (for files with no JSX)
- `utils.js` → `utils.ts`

Start with leaf components (no children) and work upward to avoid cascading errors.

**Do not rename everything at once.** TypeScript errors compound. Migrate one component, fix its errors, then move to the next.

---

## Step 7: Add prop interfaces to every component

**Before:**
```jsx
function PatientCard({ patient, onSelect }) {
  return <div onClick={() => onSelect(patient.id)}>{patient.name}</div>;
}
```

**After:**
```tsx
interface PatientCardProps {
  patient: Patient;
  onSelect: (id: string) => void;
}

function PatientCard({ patient, onSelect }: PatientCardProps) {
  return <div onClick={() => onSelect(patient.id)}>{patient.name}</div>;
}
```

Rules for this step:
- Every component gets a `ComponentNameProps` interface
- Event handlers typed as `() => void`, `(id: string) => void`, `(e: React.ChangeEvent<HTMLInputElement>) => void` etc.
- `children` typed as `React.ReactNode`
- Optional props marked with `?`
- **Never use `any` as a prop type.** If you don't know the type, use `unknown` and narrow it, or use a more specific interface.

---

## Step 8: Type React hooks

**useState:**
```ts
// Before
const [patients, setPatients] = useState([]);

// After
const [patients, setPatients] = useState<Patient[]>([]);
```

**useRef:**
```ts
const inputRef = useRef<HTMLInputElement>(null);
```

**useContext:**
```ts
const context = useContext(PatientContext);
// PatientContext should be typed: React.createContext<PatientContextType | null>(null)
```

**Event handlers:**
```ts
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};

const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  e.preventDefault();
};
```

---

## Step 9: Fix relative imports → absolute path aliases

As part of migration, replace all `../../` relative imports with `@/` aliases:

```ts
// Before
import { Patient } from '../../types/domain';
import { getPatients } from '../../../api/patients';

// After
import { Patient } from '@/domain';
import { getPatients } from '@/api';
```

---

## Step 10: Add Zod schemas

After `domain.ts` is stable, add runtime validation schemas in `src/schemas/`:

```ts
// src/schemas/patient.ts
import { z } from 'zod';

export const PatientSchema = z.object({
  id: z.string(),
  name: z.string(),
  status: z.enum(['active', 'inactive']),
  notes: z.string().optional(),
});

export type Patient = z.infer<typeof PatientSchema>;
```

Update `domain.ts` to re-export from schemas so there's one source of truth:
```ts
export type { Patient } from '@/schemas/patient';
```

---

## Step 11: Verify

```bash
npx tsc --noEmit
```

Fix every error. Do not suppress errors with `// @ts-ignore` or `as any`. Each error is a real bug caught by TypeScript — fixing it makes the codebase more reliable.

Common errors and fixes:

| Error | Fix |
| :--- | :--- |
| `Object is possibly undefined` | Add optional chaining `?.` or null check |
| `Type 'string' is not assignable to type 'never'` | Add explicit type to `useState` |
| `Property X does not exist on type Y` | Update the interface or fix the access pattern |
| `Parameter implicitly has 'any' type` | Add explicit parameter type |

---

## Step 12: Run the codebase

```bash
npm run dev
```

Verify the app renders identically to before migration. TypeScript migration must not change any visual output. If something looks different, the migration introduced a bug — find and fix it before proceeding to the 19-dimension audit.

---

## What not to do

- **No `any` types.** Ever. If you're tempted to use `any`, use `unknown` and add a type guard.
- **No `// @ts-ignore`.** This hides real bugs.
- **No mass renaming then fixing.** Rename one file, fix errors, move on.
- **No new features during migration.** Migration only — no refactoring beyond what TypeScript requires.
