# Stack: React

> Load this file when the detected stack is React (TypeScript or JavaScript). Apply the correct path based on file extensions found.

---

## Detect the path

| Signal | Path |
| :--- | :--- |
| `.tsx` / `.ts` files in `src/` | TypeScript path |
| `.jsx` / `.js` files, no `.ts` files | JavaScript path |
| Mix of both | TypeScript path — treat JS files as migration candidates |

---

## TypeScript path

**Domain types:** strict interfaces in `domain.ts`
```ts
export interface Patient {
  id: string;
  name: string;
  status: 'active' | 'inactive';
  notes?: string;
}

export interface ApiResponse<T> {
  data: T;
  meta: { total?: number; page?: number };
}
```

**Runtime validation:** Zod schemas in `src/schemas/`
- If Zod already in `package.json` — use it
- If Yup present — use `yup.object()` + `yup.InferType<>`
- If Valibot/Joi present — use that library's equivalent
- If nothing present — `npm install zod`

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

**API stubs:** typed async functions in `src/api/index.ts`
```ts
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[]; meta: { total: number } }
export async function getPatients(): Promise<ApiResponse<Patient[]>> {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

**Data fetching hooks:** TanStack Query in `src/api/hooks/`
- If TanStack Query already in `package.json` — use it
- If SWR present — wrap stubs in `useSWR` pattern
- If Axios with custom hooks present — use that pattern
- If nothing present — `npm install @tanstack/react-query`

```ts
// src/api/hooks/usePatients.ts
import { useQuery } from '@tanstack/react-query';
import { getPatients } from '@/api';

export function usePatients() {
  return useQuery({ queryKey: ['patients'], queryFn: getPatients });
}
```

**Code quality:** no `any` types, strict interfaces, organized imports, ESLint/Prettier

**Path aliases:** `@/*` mapping in `tsconfig.json`, no `../../` relative imports

**TypeScript migration offer:** If the codebase is JavaScript and the developer has indicated they prefer TypeScript, offer migration:
> "Your project is in JavaScript and that works fine. Your developer mentioned they prefer TypeScript — want me to migrate the codebase first? I'll do it properly: full interfaces, typed API stubs, no shortcuts."
If confirmed, load `modes/migrate.md` before proceeding.

---

## JavaScript path

**Domain types:** JSDoc typedefs in `domain.js`
```js
/**
 * @typedef {Object} Patient
 * @property {string} id
 * @property {string} name
 * @property {'active' | 'inactive'} status
 * @property {string} [notes]
 */
```

**Runtime validation:** same library detection logic as TypeScript path. Default to Zod if nothing present — it works in JavaScript too.

**API stubs:** async functions in `src/api/index.js`
```js
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[], meta: { total: number } }
export async function getPatients() {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

**Data fetching hooks:** same library detection logic as TypeScript path.

**Code quality:** PropTypes on all components, JSDoc on all functions in `api/index.js` and `domain.js`, organized imports, ESLint/Prettier

**Path aliases:** `@/*` mapping in `jsconfig.json`, no `../../` relative imports

JavaScript is fully valid for production handoff. Never frame it as inferior or suggest migration unless the developer has explicitly asked for it.

---

## Routing

- Replace conditional page rendering (`if (page === 'home')`) with `react-router-dom` paths
- Lazy loading with `React.lazy` for heavy route containers
- Route guards via `<ProtectedRoute />` for authenticated views

## State management

- Avoid chains of 5+ independent `useState` hooks
- Consolidate into reducer-based state or Zustand store
- If Zustand already present — use it
- If Redux present — use it
- If nothing present — recommend Zustand for new global state: `npm install zustand`
