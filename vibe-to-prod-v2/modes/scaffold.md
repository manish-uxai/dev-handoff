# Mode: Scaffold

> Load this file when the user runs `/vibe-to-prod-v2 scaffold`. They are starting a new project from scratch.

---

## Step 0: Ask before scaffolding

Ask two questions:
1. "TypeScript or JavaScript?" — default to TypeScript
2. "Vite, Next.js, or something else?" — default to Vite + React

Then follow the matching path.

---

## Path A: Vite + React + TypeScript (default)

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm install @tanstack/react-query zod react-router-dom zustand lucide-react
npm install --save-dev @types/node
npx shadcn@latest init
```

**`tsconfig.json`** — add path aliases:
```json
{ "compilerOptions": { "baseUrl": ".", "paths": { "@/*": ["./src/*"] } } }
```

**`vite.config.ts`** — add alias:
```ts
import path from 'path';
export default defineConfig({
  plugins: [react()],
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
});
```

**Scaffold folders:**
```bash
mkdir -p src/api/hooks src/components/ui src/components/icons \
  src/contexts src/data src/hooks src/pages src/routes \
  src/schemas src/store src/utils
touch src/domain.ts src/api/index.ts
```

**`src/domain.ts`:**
```ts
export interface ApiResponse<T> {
  data: T;
  meta: { total?: number; page?: number; [key: string]: unknown };
}
```

**`src/api/index.ts`:**
```ts
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

// @backend GET /api/health
export async function checkHealth() {
  await delay(300);
  return { data: { status: 'ok' }, meta: {} };
}
```

**`src/api/hooks/index.ts`:**
```ts
import { QueryClient } from '@tanstack/react-query';
export const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 1000 * 60 * 5, retry: 1 } },
});
```

**`src/main.tsx`:**
```tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/api/hooks';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);
```

**`.env.example`:**
```
VITE_API_BASE_URL=https://api.example.com/v1
VITE_AUTH_DOMAIN=auth.example.com
```

---

## Path B: Vite + React + JavaScript

Same as Path A with:
- `--template react` instead of `react-ts`
- `jsconfig.json` instead of `tsconfig.json`
- `domain.js` with JSDoc, `api/index.js`
- PropTypes instead of TypeScript interfaces
- Skip Zod initially, add PropTypes: `npm install prop-types`

---

## Path C: Next.js

```bash
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd my-app
npm install @tanstack/react-query zod zustand lucide-react
npx shadcn@latest init
```

**`src/components/Providers.tsx`:**
```tsx
'use client';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/api/hooks';
export function Providers({ children }: { children: React.ReactNode }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

**`src/app/layout.tsx`** — wrap with Providers:
```tsx
import { Providers } from '@/components/Providers';
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html><body><Providers>{children}</Providers></body></html>;
}
```

Create `src/app/error.tsx`, `src/app/not-found.tsx`, `src/app/loading.tsx` as empty shells.

---

## After scaffolding

Tell the user:

> "Your project is scaffolded and ready.
>
> - Components go in `src/components/`
> - Pages go in `src/pages/` (or `src/app/` for Next.js)
> - Data shapes go in `src/domain.ts`
> - API connections go in `src/api/index.ts`
>
> When you're ready to hand off to a developer, run `/vibe-to-prod-v2 audit` and I'll check everything."
