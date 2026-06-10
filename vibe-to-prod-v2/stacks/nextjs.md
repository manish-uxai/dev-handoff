# Stack: Next.js

> Load this file when `next.config.*` is present or `next` appears in `package.json`. Load `stacks/react.md` as well — this file contains overrides only for what's genuinely different in Next.js. Everything in `stacks/react.md` applies unless explicitly overridden here.

---

## What's different in Next.js

### Routing (overrides dimension 11)
`react-router-dom` is irrelevant. Next.js uses file-based routing. Check:
- Pages live in `/app` (App Router) or `/pages` (Pages Router) — not conditionally rendered in a root component
- Dynamic routes use `[param]` and `[[...slug]]` conventions
- Route guards implemented via `middleware.ts` at the project root, not `<ProtectedRoute />` wrappers
- If conditional rendering (`if (page === 'home')`) is still used instead of file-based routes, flag it as High severity

### Lazy loading (overrides React.lazy in dimension 11)
`React.lazy` is not needed — Next.js handles code splitting per page automatically. Instead check:
- Heavy components (maps, charts, rich editors) use `dynamic()` from `next/dynamic` with `{ ssr: false }` where appropriate
- Flag any `React.lazy` usage as unnecessary

### Error boundaries (overrides dimension 15 route-level)
Instead of manual React Error Boundary components, check for Next.js built-in error files:
- `error.tsx` at appropriate layout levels
- `not-found.tsx` for 404 handling
- `loading.tsx` for async suspense states

Component-level resilience checks (loading/error/empty states per component) still apply unchanged.

### State hooks in Server Components (extends dimension 5)
Flag any `useState`, `useContext`, or `useReducer` inside a Server Component. State hooks only work in Client Components. Files using them must have `"use client"` at the top.

Plain language: "This file manages interactive state but isn't marked as a client component — the developer will need to fix this before it works."

### API stubs in Next.js (extends dimension 4 & 7)
The `api/index.ts` stub pattern still applies for frontend data consumption. `// @backend` annotations should note whether endpoints are expected as:
- Next.js API routes: `/app/api/resource/route.ts`
- External backend service: `https://api.example.com/resource`

### QueryClientProvider in Next.js
TanStack Query requires a client-side provider. Wrap in a dedicated client component:

```tsx
// src/components/Providers.tsx
'use client';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/api/hooks';

export function Providers({ children }: { children: React.ReactNode }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

```tsx
// src/app/layout.tsx
import { Providers } from '@/components/Providers';
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html><body><Providers>{children}</Providers></body></html>;
}
```

---

## Target file layout for Next.js

```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── error.tsx
│   ├── not-found.tsx
│   └── loading.tsx
├── api/
│   ├── index.ts
│   └── hooks/
├── components/
│   ├── ui/
│   ├── icons/
│   ├── Providers.tsx
│   └── ConfirmDialog.tsx
├── data/
├── domain.ts
├── schemas/
├── store/
└── utils/
middleware.ts
.env.example
```
