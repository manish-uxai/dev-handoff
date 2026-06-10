# Scaffold Mode: Greenfield Project Setup

> Read this file when the user runs `/vibe-to-prod scaffold`. They are starting a new project from scratch and want the production-ready architecture set up before building any UI.
>
> Your job is to set up the full folder structure, config files, and base architecture — so every component the designer builds lands in the right place from day one, with zero refactoring needed later.

---

## Step 0: Ask two questions before scaffolding

Before writing any files, ask:

1. "Are you using TypeScript or JavaScript?" — Default to TypeScript unless they say otherwise.
2. "Are you using Vite, Next.js, or something else?" — Default to Vite + React unless they say otherwise.

Then follow the matching setup path below.

---

## Path A: Vite + React + TypeScript (default)

### Step 1: Create the Vite project

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
```

### Step 2: Install core dependencies

```bash
# Data fetching
npm install @tanstack/react-query

# Runtime validation
npm install zod

# UI primitives
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu @radix-ui/react-tooltip
# Or install shadcn/ui (recommended — includes all Radix primitives pre-styled):
npx shadcn@latest init

# Icons
npm install lucide-react

# Routing
npm install react-router-dom

# State management (global UI state)
npm install zustand
```

### Step 3: Update `tsconfig.json`

Add path aliases:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Step 4: Update `vite.config.ts`

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

### Step 5: Scaffold the folder structure

Create this exact structure inside `src/`:

```
src/
├── api/
│   ├── index.ts          # API stub functions with @backend annotations
│   └── hooks/            # TanStack Query hooks
├── components/
│   ├── ui/               # shadcn/Radix primitives
│   ├── icons/            # Custom SVG icons (only if no lucide-react match)
│   └── ConfirmDialog.tsx
├── contexts/             # One file per domain context
├── data/                 # Mock seed data (consumed only by api/index.ts)
├── domain.ts             # TypeScript interfaces
├── hooks/                # Shared non-data-fetching hooks
├── pages/                # Route-level page components
├── routes/               # react-router setup + guards
├── schemas/              # Zod validation schemas
├── store/                # Zustand global UI state stores
└── utils/                # Helpers, formatters
```

Create placeholder files to establish the structure:

```bash
mkdir -p src/api/hooks src/components/ui src/components/icons src/contexts src/data src/hooks src/pages src/routes src/schemas src/store src/utils
touch src/domain.ts src/api/index.ts
```

### Step 6: Create base files

**`src/domain.ts`** — empty, ready for interfaces:
```ts
// Add your data interfaces here
// Example:
// export interface User {
//   id: string;
//   name: string;
// }

export interface ApiResponse<T> {
  data: T;
  meta: {
    total?: number;
    page?: number;
    [key: string]: unknown;
  };
}
```

**`src/api/index.ts`** — delay utility + first stub:
```ts
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

// @backend GET /api/health
// Auth: None
// Response: { data: { status: string } }
export async function checkHealth(): Promise<ApiResponse<{ status: string }>> {
  await delay(300);
  return { data: { status: 'ok' }, meta: {} };
}
```

**`src/api/hooks/index.ts`** — base query client setup:
```ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      retry: 1,
    },
  },
});
```

**`src/main.tsx`** — wrap app with QueryClientProvider:
```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/api/hooks';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);
```

**`src/components/ConfirmDialog.tsx`** — reusable destructive action guard:
```tsx
import * as Dialog from '@radix-ui/react-dialog';

interface ConfirmDialogProps {
  open: boolean;
  onConfirm: () => void;
  onCancel: () => void;
  title: string;
  description: string;
  confirmLabel?: string;
}

export function ConfirmDialog({
  open,
  onConfirm,
  onCancel,
  title,
  description,
  confirmLabel = 'Confirm',
}: ConfirmDialogProps) {
  return (
    <Dialog.Root open={open} onOpenChange={onCancel}>
      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/40" />
        <Dialog.Content className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 bg-white rounded-lg p-6 shadow-xl w-[400px]">
          <Dialog.Title className="text-lg font-semibold">{title}</Dialog.Title>
          <Dialog.Description className="mt-2 text-sm text-gray-600">{description}</Dialog.Description>
          <div className="mt-4 flex justify-end gap-3">
            <button onClick={onCancel} className="px-4 py-2 text-sm rounded border">Cancel</button>
            <button onClick={onConfirm} className="px-4 py-2 text-sm rounded bg-red-600 text-white">{confirmLabel}</button>
          </div>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

### Step 7: Create `.env.example`

```bash
VITE_API_BASE_URL=https://api.example.com/v1
VITE_AUTH_DOMAIN=auth.example.com
```

### Step 8: Confirm to the user

Tell the user:

> "Your project is scaffolded and ready. Every component you build should go in `src/components/`. Pages go in `src/pages/`. Data shapes go in `src/domain.ts`. API connections go in `src/api/index.ts`.
>
> When you're ready to prepare the project for developer handoff, run `/vibe-to-prod audit` and I'll check everything against the 19-dimension framework."

---

## Path B: Vite + React + JavaScript

Same as Path A with these differences:

- Skip `tsconfig.json` TypeScript strict settings — create `jsconfig.json` instead:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },
    "checkJs": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
```
- Use `domain.js` with JSDoc typedefs instead of `domain.ts`
- Use `api/index.js` instead of `api/index.ts`
- All component files use `.jsx` instead of `.tsx`
- Skip Zod (can add later) — use PropTypes for component contracts instead

---

## Path C: Next.js

```bash
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd my-app
```

This command sets up TypeScript, Tailwind, ESLint, App Router, `src/` directory, and `@/*` path aliases in one step.

Then install additional dependencies:
```bash
npm install @tanstack/react-query zod lucide-react zustand
npx shadcn@latest init
```

Folder structure for Next.js:
```
src/
├── app/                  # App Router pages and layouts
│   ├── layout.tsx
│   ├── page.tsx
│   ├── error.tsx         # Route-level error boundary
│   ├── not-found.tsx     # 404 page
│   └── loading.tsx       # Loading state
├── api/
│   ├── index.ts          # API stub functions
│   └── hooks/            # TanStack Query hooks
├── components/
│   ├── ui/               # shadcn/Radix primitives
│   └── ConfirmDialog.tsx
├── data/                 # Mock seed data
├── domain.ts             # TypeScript interfaces
├── schemas/              # Zod schemas
├── store/                # Zustand stores
└── utils/                # Helpers
```

Wrap the root layout with QueryClientProvider in a client component:
```tsx
// src/components/Providers.tsx
'use client';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/api/hooks';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

```tsx
// src/app/layout.tsx
import { Providers } from '@/components/Providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## Rules for scaffold mode

1. **Never create placeholder components with hardcoded data.** The scaffold sets up structure only — no UI yet.
2. **Always create `.env.example`.** Even if empty, it establishes the pattern.
3. **Always set up path aliases.** `@/*` from day one prevents a painful migration later.
4. **Always wrap the app with QueryClientProvider.** Forgetting this causes confusing errors when the first hook is added.
5. **Always create `ConfirmDialog.tsx`.** Destructive actions need this from the first feature built.
6. **Tell the user what to do next.** End scaffold mode with a clear summary of where things go.
