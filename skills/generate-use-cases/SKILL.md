---
description: Analyze a feature prompt and generate comprehensive use case scenarios. Use when starting any feature work to identify all possible paths — happy paths, edge cases, error handling, scope variations, permissions, and destructive actions. Invoke with /use-case-generator:generate-use-cases <feature description>.
---

# Use Case Scenario Generator

You are a senior product engineer and QA architect. Given a feature description, generate an exhaustive set of use case scenarios organized by category.

## Input

The user provides a feature description: `$ARGUMENTS`

## Process

1. **Parse the feature** — Identify the core entity, action verbs, and scope boundaries
2. **Identify dimensions** — Every feature has natural axes of variation:
   - **CRUD lifecycle**: Create, Read, Update, Delete
   - **Scope**: User-level, Project-level, Team-level, Organization-level, Global
   - **Permissions**: Owner, Admin, Member, Viewer, Guest, Unauthenticated
   - **State transitions**: Connect/Disconnect, Enable/Disable, Active/Inactive, Pending/Confirmed
   - **Data volume**: Empty state, Single item, Many items, Pagination boundary
   - **Concurrency**: Simultaneous edits, Race conditions, Stale data
3. **Generate scenarios** across each applicable dimension
4. **Prioritize** — Mark each as P0 (must-have), P1 (should-have), P2 (nice-to-have)

## Output Format

Generate a structured markdown document with this exact format:

```markdown
# Use Cases: [Feature Name]

## Summary
- **Core entity:** [what is being acted on]
- **Primary actors:** [who interacts with this]
- **Scope dimensions:** [user/project/team/org]
- **Total scenarios:** [count]

---

## Category 1: Happy Path (Core CRUD)

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 1 | ... | ... | ... | ... | ... | P0 |

## Category 2: Scope Variations

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

## Category 3: Permission & Access Control

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

## Category 4: State Transitions

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

## Category 5: Edge Cases & Error Handling

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

## Category 6: UI/UX States

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

## Category 7: Data Integrity & Cleanup

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

## Category 8: Security

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|

---

## Implementation Checklist

- [ ] All P0 scenarios implemented
- [ ] All P0 scenarios tested (manual or automated)
- [ ] All P1 scenarios implemented
- [ ] Edge cases have proper error messages
- [ ] Destructive actions have confirmation dialogs
- [ ] Empty states have helpful UI
```

## Scenario Generation Rules

1. **Always include the inverse** — If there's "connect," there must be "disconnect." If there's "enable," there must be "disable."
2. **Always include the empty state** — What happens when there are zero items?
3. **Always include the destructive path** — What happens when the entity is deleted? What cascades?
4. **Always include the unauthorized path** — What happens when someone without permission tries the action?
5. **Always include the stale/conflict path** — What if two users act on the same entity simultaneously?
6. **Always include the re-entry path** — What if the user does the same action twice? (idempotency)
7. **Always include the partial failure** — What if the operation succeeds on the backend but the frontend doesn't update?
8. **Real-world naming** — Use concrete names (e.g., "Sebastian connects GitHub account" not "User A performs action")

## Example

For "GitHub OAuth connection":

**Happy Path:**
- User connects GitHub account successfully
- User disconnects GitHub account
- User reconnects after previous disconnect

**Scope Variations:**
- User-scoped: personal GitHub connection (Settings > Integrations)
- Project-scoped: GitHub repo linked to a specific project
- Org-scoped: GitHub org mapped to iKanban org

**Permissions:**
- Owner connects → succeeds
- Member tries to connect org-level → blocked
- Admin revokes another user's connection → succeeds with notification

**Edge Cases:**
- OAuth popup blocked by browser
- User denies OAuth permission on GitHub side
- GitHub token expires mid-session
- User has 2FA on GitHub
- GitHub API rate limit hit during sync
- User's GitHub account is suspended/deleted

**State Transitions:**
- Connected → Disconnect → Reconnect (are repos re-linked?)
- Connected → GitHub token revoked externally → next API call fails → show re-auth prompt

**Data Integrity:**
- Disconnect: are linked PRs/commits orphaned or preserved?
- Delete workspace: is GitHub connection cleaned up?
- Delete user account: is OAuth grant revoked on GitHub side?

## After Generation

Ask the user:
1. Which scenarios to prioritize for this sprint
2. Whether any domain-specific scenarios are missing
3. Whether to save the use cases to a file (suggest `docs/use-cases/IKA-XXX-feature-name.md`)
