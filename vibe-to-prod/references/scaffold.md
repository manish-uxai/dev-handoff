# Scaffold Mode: Greenfield Project Setup

> Read this file when the user runs `/vibe-to-prod scaffold` or `/vibe-to-prod start`, or when **converting a plain HTML project to React** (Step 0 HTML detection). In all cases, the goal is the same: set up the full Vite + React + TypeScript folder structure, config files, and base architecture.
>
> For greenfield: every component the designer builds lands in the right place from day one, with zero refactoring needed later.
> For HTML conversion: the scaffold creates the React project structure, then the agent converts the HTML files into React components inside it.

---

## Pre-flight: Node.js check

Before scaffolding, run `node --version`. If Node.js is not installed, stop and direct the designer to install it (see SKILL.md pre-flight section for the exact message). Scaffolding requires Node for `npm create vite`, dependency installation, and running the dev server.

---

## Step 0: Default stack — no questions needed

vibe-to-prod always scaffolds **Vite + React + TypeScript**. Don't ask about JS vs TS, and don't ask Vite vs Next.js unless the user explicitly mentions Next.js. A designer starting fresh gets the default stack — decision already made for them, zero friction.

Only deviate to Next.js (Path B) if the user explicitly says they need it (SSR, server components, file-based routing). Otherwise: Path A.

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
# UI primitives
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu @radix-ui/react-tooltip
# Or install shadcn/ui (recommended — includes all Radix primitives pre-styled):
npx shadcn@latest init

# Icons
npm install lucide-react

# Routing
npm install react-router-dom
```

Deliberately NOT installed: TanStack Query, SWR, Zod. Data is served by plain async stub functions (setTimeout-simulated) shared through React Context API. The developer adds a real data-fetching library their own way at backend integration — scaffolding it now imposes a pattern they may not want and a provider that can crash if unmounted.

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
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### Step 5: Scaffold the folder structure

Create this exact structure inside `src/`:

```
src/
├── api/
│   └── index.ts          # Self-documenting async stub functions (setTimeout-simulated)
├── components/
│   ├── ui/               # shadcn/Radix primitives
│   ├── icons/            # Custom SVG icons (only if no lucide-react match)
│   └── ConfirmDialog.tsx
├── contexts/             # One file per domain context (shared data + state via Context API)
├── data/                 # Mock seed data (consumed only by api/index.ts)
├── domain.ts             # TypeScript interfaces — one source of truth for all shared types
├── hooks/                # Shared hooks (e.g. useReducer-based screen state)
├── pages/                # Route-level page components
├── routes/               # react-router setup + guards
└── utils/                # Helpers, formatters
```

Create placeholder files to establish the structure:

```bash
mkdir -p src/components/ui src/components/icons src/contexts src/data src/hooks src/pages src/routes src/utils
touch src/domain.ts src/api/index.ts
```

### Step 6: Create base files

**`src/domain.ts`** — empty, ready for interfaces:

```ts
// One source of truth for every data shape the UI renders.
// Components import their types from here — they never define their own.
// Example:
// export interface HealthStatus {
//   status: 'ok' | 'degraded' | 'down';
// }
```

**`src/api/index.ts`** — delay utility + first stub:

```ts
import type { HealthStatus } from "@/domain";

const delay = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));

// Returns service health. Swap the body for a real API call at integration.
export async function checkHealth(): Promise<HealthStatus> {
  await delay(300);
  return { status: "ok" };
}
```

The descriptive name + typed return is the integration map — no annotation system.

**`src/contexts/AppDataContext.tsx`** — share data across screens via Context API (no TanStack):

```tsx
import { createContext, useContext, useEffect, useState } from "react";
import { checkHealth } from "@/api";
import type { HealthStatus } from "@/domain";

interface AppData {
  health: HealthStatus | null;
  loading: boolean;
  error: string | null;
}

const AppDataContext = createContext<AppData | null>(null);

export function AppDataProvider({ children }: { children: React.ReactNode }) {
  const [health, setHealth] = useState<HealthStatus | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    checkHealth()
      .then(setHealth)
      .catch(() => setError("Failed to load."))
      .finally(() => setLoading(false));
  }, []);

  return (
    <AppDataContext.Provider value={{ health, loading, error }}>
      {children}
    </AppDataContext.Provider>
  );
}

export function useAppData() {
  const ctx = useContext(AppDataContext);
  if (!ctx) throw new Error("useAppData must be used within AppDataProvider");
  return ctx;
}
```

**`src/main.tsx`** — wrap app with the data provider (mount it, or the hook throws at runtime):

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import { AppDataProvider } from "@/contexts/AppDataContext";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <BrowserRouter>
      <AppDataProvider>
        <App />
      </AppDataProvider>
    </BrowserRouter>
  </React.StrictMode>,
);
```

**`src/components/ConfirmDialog.tsx`** — reusable destructive action guard:

```tsx
import * as Dialog from "@radix-ui/react-dialog";

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
  confirmLabel = "Confirm",
}: ConfirmDialogProps) {
  return (
    <Dialog.Root open={open} onOpenChange={onCancel}>
      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/40" />
        <Dialog.Content className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 bg-white rounded-lg p-6 shadow-xl w-[400px]">
          <Dialog.Title className="text-lg font-semibold">{title}</Dialog.Title>
          <Dialog.Description className="mt-2 text-sm text-gray-600">
            {description}
          </Dialog.Description>
          <div className="mt-4 flex justify-end gap-3">
            <button
              onClick={onCancel}
              className="px-4 py-2 text-sm rounded border"
            >
              Cancel
            </button>
            <button
              onClick={onConfirm}
              className="px-4 py-2 text-sm rounded bg-red-600 text-white"
            >
              {confirmLabel}
            </button>
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
> When you're ready to prepare the project for developer handoff, run `/vibe-to-prod audit` and I'll check everything against the 18-dimension framework."

---

## Path B: Next.js

```bash
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd my-app
```

This command sets up TypeScript, Tailwind, ESLint, App Router, `src/` directory, and `@/*` path aliases in one step.

Then install additional dependencies:

```bash
npm install lucide-react
npx shadcn@latest init
```

(No TanStack Query, Zod, or Zustand — same reasoning as the Vite path: data through async stubs + Context API, validation left to the backend dev.)

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
│   └── index.ts          # Self-documenting async stub functions
├── components/
│   ├── ui/               # shadcn/Radix primitives
│   └── ConfirmDialog.tsx
├── contexts/             # Domain data + state via Context API
├── data/                 # Mock seed data
├── domain.ts             # TypeScript interfaces
└── utils/                # Helpers
```

Wrap the root layout with the data provider in a client component:

```tsx
// src/components/Providers.tsx
"use client";
import { AppDataProvider } from "@/contexts/AppDataContext";

export function Providers({ children }: { children: React.ReactNode }) {
  return <AppDataProvider>{children}</AppDataProvider>;
}
```

```tsx
// src/app/layout.tsx
import { Providers } from "@/components/Providers";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
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
4. **Mount any Context provider you create in the tree.** An unmounted provider makes its hook throw at runtime — verify it wraps the app.
5. **Always create `ConfirmDialog.tsx`.** Destructive actions need this from the first feature built.
6. **Tell the user what to do next.** End scaffold mode with a clear summary of where things go.
