# Conditional Navigation → React Router Migration

> **Read this when dimension 11 identifies a multi-page app using conditional rendering instead of a router.** This is a mechanical transformation with one known pitfall. Follow the steps in order — don't improvise the approach.

---

## Detect the pattern

The app switches views using state:

```tsx
const [currentView, setCurrentView] = useState<
  "landing" | "dashboard" | "workspace"
>("landing");

// In the render:
{
  currentView === "landing" && <LandingPage />;
}
{
  currentView === "dashboard" && <Dashboard />;
}
{
  currentView === "workspace" && <Workspace />;
}
```

The state variable names vary: `currentView`, `activePage`, `page`, `screen`, `view`. The pattern is the same.

---

## Step 1: Install react-router-dom

```bash
npm install react-router-dom
```

---

## Step 2: Add BrowserRouter to the entry point

Wrap the app in `BrowserRouter` at the entry level (`main.tsx` or `index.tsx`), NOT inside a component:

```tsx
// main.tsx
import { BrowserRouter } from "react-router-dom";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
```

---

## Step 3: Map views to route paths

Before editing any code, write down the mapping:

```
'landing'        → '/'
'loading'        → '/loading'
'workspace'      → '/workspace'
'dashboard'      → '/dashboard'
'ert-governance' → '/ert-governance'
```

Every value the state variable can hold becomes a route path. This is the reference for all replacements.

---

## Step 4: Add hooks BEFORE any early returns

**THIS IS THE KNOWN PITFALL.** React hooks cannot be called after a conditional return. If the component has an early return (loading screen, splash, auth check), the hooks MUST go before it.

```tsx
// WRONG — hooks after early return violates React rules
function AppContent() {
  if (showSplash) return <SplashScreen />;
  const navigate = useNavigate(); // ← React error
  const location = useLocation(); // ← React error
}

// RIGHT — hooks before any early return
function AppContent() {
  const navigate = useNavigate();
  const location = useLocation();

  if (showSplash) return <SplashScreen />;
  // ... rest of component
}
```

Add both `useNavigate` and `useLocation` at the top of the component, before any early returns or conditional logic.

---

## Step 5: Derive the current view from location (if needed)

If other components receive the current view as a prop (e.g., `<Header currentView={currentView} />`), derive it from the pathname instead of removing the prop:

```tsx
const location = useLocation();
const currentView = location.pathname.split("/")[1] || "landing";
```

This keeps the prop interface stable — child components don't need to change.

---

## Step 6: Replace all setCurrentView calls with navigate

This is a mechanical find-and-replace. Use the mapping from Step 3:

```
setCurrentView('landing')     → navigate('/')
setCurrentView('workspace')   → navigate('/workspace')
setCurrentView('dashboard')   → navigate('/dashboard')
```

Use `perl -i -pe` or `sed -i` for the replacement — it's the same pattern every time. Verify with `grep -n "setCurrentView" src/` afterward to confirm zero remaining.

---

## Step 7: Replace the conditional render block with Routes

Replace the `{currentView === 'x' && <Component />}` block with declarative routes:

```tsx
import { Routes, Route, Navigate } from "react-router-dom";

// BEFORE:
{
  currentView === "landing" && <LandingPage />;
}
{
  currentView === "dashboard" && <Dashboard />;
}
{
  currentView === "workspace" && <Workspace />;
}

// AFTER:
<Routes>
  <Route path="/" element={<LandingPage />} />
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/workspace" element={<Workspace />} />
  <Route path="*" element={<Navigate to="/" replace />} />
</Routes>;
```

Keep any wrapping layout (header, sidebar) OUTSIDE the `<Routes>` block — only the view-switching content goes inside.

---

## Step 8: Remove the old state variable

Delete the `useState` declaration for the view state (and its type if it was a union type). Also remove `setCurrentView` from any prop interfaces that passed it down.

Verify: `grep -n "currentView\|setCurrentView" src/` should return zero hits (except possibly derived-from-location references from Step 5).

---

## Step 9: Build and verify

```bash
npm run build
```

Then open the app in the browser and verify:

- Each route renders the correct view
- Browser back/forward works
- Any navigation buttons/links go to the right place
- The header/sidebar shows the correct active state at each route

This is a navigation-level change — runtime verification matters more than the build here.

---

## Common variations

**The app uses `useEffect` to trigger navigation on state change:**

```tsx
useEffect(() => {
  setCurrentView("workspace");
}, [studyLoaded]);
```

Replace with:

```tsx
useEffect(() => {
  navigate("/workspace");
}, [studyLoaded]);
```

**The app passes `setCurrentView` as a prop to children:**
Replace the prop with `navigate` from `useNavigate()` inside the child component, or pass `navigate` as the prop instead.

**The app has transition animations between views:**
Keep the animation wrapper around `<Routes>`. React Router supports `useTransition` and libraries like `framer-motion` with `AnimatePresence` around route switches.

**The app has nested conditional views (tabs inside a workspace):**
Only the top-level page switching becomes routes. Tab switching within a page stays as local state — don't over-route. Tabs are UI state, not navigation state.
