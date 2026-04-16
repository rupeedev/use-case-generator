---
description: Generate actionable feature scenarios grounded in the actual codebase. Reads UI components, hooks, and API routes, then produces a concise spec with acceptance criteria checkboxes that drive implementation. Invoke with /use-case-generator:generate-use-cases <feature description>.
---

# Use Case Scenario Generator

Given a feature description, **read the relevant code**, then produce a concise, actionable spec that covers all scenarios in one pass — so the feature ships complete without fix-up rounds.

## Input

`$ARGUMENTS` — a feature description, optionally with scope or file hints.

Examples:
- `GitHub OAuth connection for user settings`
- `Document sharing — affects DocShareDialog.tsx and documents.rs`
- `IKA-700 Team invitation system`

## Step 1: Read the Code (MANDATORY)

Before generating anything, find and read the relevant source files:

1. **Search** — Glob/Grep for components, hooks, routes, stores matching the feature keywords
2. **Read** — For each file, note: UI controls (buttons, forms, toggles), API calls (endpoint + method), state (loading/error/empty), guards (auth/role checks), feedback (toasts, dialogs)
3. **Backend** — Check route handlers for validation, DB queries, authorization

Keep this fast — read only files directly related to the feature, not the whole codebase.

## Step 2: Generate the Spec

Output this format — **keep it tight, no filler**:

```markdown
# Feature Spec: [Name]

## Code Audit

**Files:** (list each file read + one-line purpose)
**UI actions:** (every clickable/submittable thing found)
**API endpoints:** (method + path + what it does)

---

## SC-01: [Scenario name] — P0
- **Entry:** [where the user starts — page, button, menu item]
- **Source:** `file.tsx:line`
- **Flow:**
  1. User does X
  2. System calls `POST /api/...`
  3. UI updates to show Y
- **API:** `METHOD /path` → `200 { field: value }` | `422 { error: "..." }`
- **Accept:**
  - [ ] [criterion — what must be true when this works]
  - [ ] [criterion]
- **Errors:**
  - [ ] [what happens on failure — specific error + UI response]

## SC-02: [Scenario name] — P0
...

## SC-N: [GAP] [Missing thing] — P1
- **Source:** `file.tsx:line` — [what exists that implies this should exist]
- **Fix:** [one-line description of what to add]
- **Accept:**
  - [ ] [criterion]

---

## Build Order

1. SC-01, SC-02 (core path — build these first)
2. SC-03, SC-04 (variations — build after core works)
3. SC-N gaps (harden — address before shipping)
```

## Rules

1. **Code-first** — Only generate scenarios for actions the code supports or clearly should support. No hypotheticals.
2. **Concise** — Each scenario should be 5-10 lines max. No tables. No redundant columns.
3. **Acceptance criteria = checkboxes** — Every criterion is something you can verify with one action (click, curl, or look at DB).
4. **Errors are part of the scenario, not a separate category** — Each scenario lists its own failure modes inline.
5. **Gaps are scenarios too** — If code implies something should exist but doesn't (button with no handler, API with no error state, list with no empty state), write it as `[GAP] SC-N`.
6. **Build order at the end** — Tell the developer what to build first so dependencies flow correctly.
7. **Limit to ~10 scenarios max** — Focus on what matters. If the feature genuinely has 20 scenarios, split into two specs.
8. **Cite source files** — Every scenario references the file:line it came from.
9. **Real names** — "Sebastian clicks Disconnect" not "User A performs action."

## After Generation

Ask the user:
1. Any scenarios missing from your mental model?
2. Save to `docs/use-cases/IKA-XXX-feature-name.md`?
