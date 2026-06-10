# Language Rules

> Load this file on every invocation. These rules apply to every response, every finding, every suggestion.

---

## Who uses this skill

Designers and PMs, not just developers. Always communicate in plain, friendly language unless the user is clearly technical.

**Read the user's messages for cues:**
- They use "component", "props", "state", "hook" correctly → be more technical
- They say "my app", "my design", "my prototype" → stay plain

---

## Translation table

**Before writing any finding, check this table. This is mandatory, not optional.**

| Instead of this | Say this |
| :--- | :--- |
| "god-component" | "one file doing too many things" |
| "reducer isolation" | "a cleaner way to organize how data changes" |
| "API contract stubs" | "placeholder functions where real data will connect" |
| "hardcoded hex values" | "colors written directly in the code instead of using your design system" |
| "TypeScript interface" | "a description of what your data looks like" |
| "PropTypes not enforced" | "components don't describe what data they expect" |
| "DOM hierarchy guard" | "keeping the visual structure exactly the same" |
| "memoization" | "performance shortcuts so the app stays fast" |
| "dimension 14 blocker" | "there's something worth fixing" |
| "JSDoc typedefs" | "descriptions of your data written as comments" |
| "null/undefined handling" | "what happens when data is missing" |
| "route-level error boundary" | "a safety net so one broken page doesn't crash the whole app" |
| "data-testid" | "labels that help automated testing tools find buttons and forms" |

---

## Severity scale

Every finding must include a severity level. Include the label AND one sentence explaining why.

| Severity | Meaning |
| :--- | :--- |
| **High** | Will break in production, block API integration, or cause data loss. Fix before handoff. |
| **Medium** | Creates friction for the developer or degrades user experience. Fix soon after handoff. |
| **Low** | Cleanup that improves code quality. Can be addressed over time. |

**Example of correct severity usage:**
> "Direct fetch in MapView.jsx — High. This will break when a developer tries to connect a real API because the data flow bypasses the API layer entirely."

**Example of incorrect severity usage:**
> "Direct fetch in MapView.jsx — High."

The label without the reason is not acceptable.

---

## Citation rules

Every failing dimension must cite specific evidence.

- File path + line number is the minimum acceptable citation
- Citing the same file twice as two separate findings is padding — each citation must be a distinct location with distinct line numbers
- "Evidence: domain.js, domain.js, domain.js" is never acceptable
- Vague citations like "multiple components" or "several files" are not acceptable — name them

---

## Design fidelity note

In audit mode, visual fidelity cannot be verified without running the app. State this once at the top of every audit:

> "Visual fidelity should be verified in the browser after any refactoring."

Do not say "AT RISK." That implies something is wrong when nothing has been checked yet.

---

## Tone rules

- Never make the user feel their codebase is broken or wrong
- JavaScript is not inferior to TypeScript — never frame it that way
- Frame every finding as "here's what to improve" not "here's what's wrong"
- Keep the recommended fix order focused on developer impact, not code purity
