# Dimensions: Data & API
# Covers: 3 (Domain Types), 4 (API Stubs), 7 (Backend Markers)

---

## Dimension 3: Canonical Domain Types

A single file describing every data entity the UI renders. Apply the correct format based on detected stack (see `stacks/react.md`).

**Quality checks (both TS and JS):**
- No duplicated or conflicting type definitions across the codebase — one definition per entity, nowhere else
- Every type must be complete — all fields the UI actually uses must be documented, not just a subset
- In audit mode, verify existing types match actual runtime usage. If a component accesses `patient.address` but `Patient` has no `address` field, that's a violation
- Cite specific line numbers when flagging duplicate definitions — "domain.js, domain.js, domain.js" is not acceptable evidence

**Runtime validation:**
Follow the library detection logic in `stacks/react.md`. Every API response shape should have a corresponding schema so bad data from a real API is caught at the boundary, not deep inside a component.

---

## Dimension 4: API Contract Stubs

**Hard rule (all-or-nothing):** No component imports mock data directly or calls `fetch()` internally. One violation = this dimension fails entirely. Do not partial-pass.

Scope of the rule — covers every component type:
- Maps fetching GeoJSON directly — violation
- Charts loading CSV internally — violation
- Tables importing `MOCK_` arrays — violation
- Third-party widgets making their own network requests — violation

**Data flow pattern:**
```
src/data/          ← mock seed data lives here
    ↓
src/api/index.ts   ← async stubs consume data, expose functions
    ↓
src/api/hooks/     ← TanStack Query / SWR hooks wrap the stubs
    ↓
Components         ← consume hooks only, never raw data or fetch
```

**Latency simulation:** every stub must use `await delay(300)` to force loading states during development. Without this, loading skeletons never appear and developers assume loading states aren't needed.

**Response envelope:** mock responses use realistic envelopes, not flat arrays:
```ts
// Correct
return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };

// Wrong
return MOCK_PATIENTS;
```

---

## Dimension 7: Backend Integration Markers

Every function in `src/api/index.ts` or `src/api/index.js` must carry a `// @backend` annotation. These annotations are the developer's integration guide — they live in the code, not in a separate document.

**Required annotation format:**
```ts
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[]; meta: { total: number } }
export async function getPatients(): Promise<ApiResponse<Patient[]>> {
```

Minimum required fields per annotation:
- HTTP method + path
- Auth requirement
- Response shape

**Annotation-to-type sync:** what the annotation describes must match what `domain.ts`/`domain.js` defines. If the annotation says `Response: { data: Patient[] }` but `Patient` in `domain.ts` has different fields than what the stub actually returns, that's a contract drift violation.

**In audit mode:**
```bash
# Find API exports missing @backend annotations
grep -nE 'export\s+(async\s+)?function' src/api/index.ts src/api/index.js src/services/api.ts src/services/api.js 2>/dev/null | grep -v '@backend'
```

Flag every missing annotation as High severity — a developer cannot know what endpoint to build without it.
