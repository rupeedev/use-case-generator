# Use Cases: Project-Level GitHub/GitLab Connection Scoping

## Context

**Problem:** GitHub/GitLab connections were user-scoped (IKA-670 security fix). Connecting GitHub in any project showed "Connected" on ALL projects because there's one token per user. No way to connect Project A without bleeding into Project B.

**Solution (IKA-690):** User authenticates once (user-level token stays for security). Each project independently links/unlinks to that token via `project_github_connections` / `project_gitlab_connections` junction tables. Disconnecting from Project A does NOT affect Project B.

**Scoping history:**
1. ~~Workspace-level~~ — leaked tokens across users (security bug)
2. ~~User-level (IKA-670)~~ — fixed leak, but bled "Connected" badge across all projects
3. **Project-level (IKA-690)** — user token + per-project junction table

## Scenario Breakdown

```
======================================================================
Feature Area                                             Scenarios
======================================================================
  1. Scoping: User-Level Token (retained)                       8
  2. Scoping: Project-Level Link/Unlink                        16
  3. Scoping: Cross-Project Isolation                          10
  4. Scoping: Cross-User Isolation                              8
  5. Mapping: Project to Repository                            10
  6. UI: Project Settings Navigation                            9
  7. UI: Connection Rows in Repositories Card                  12
  8. Edge Cases & Error Handling                               10
======================================================================
                                                  TOTAL        83
======================================================================

Priority:  P0: 57    P1: 21    P2: 5
```

---

## 1. Scoping: User-Level Token (retained from IKA-670)

> User authenticates with GitHub/GitLab once. Token stored in `github_connections` / `gitlab_connections` scoped by `user_id`. This layer is NOT removed — it's the foundation that project-level links reference.

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 1 | Rupesh connects GitHub for the first time | Member | No GitHub token exists | OAuth flow completes | Row created in `github_connections` with Rupesh's `user_id` | P0 |
| 2 | Sebastian connects his own GitHub | Member | Rupesh already connected | Sebastian does OAuth | Separate row in `github_connections` with Sebastian's `user_id` | P0 |
| 3 | Rupesh's token is invisible to Sebastian | Member | Both connected | Sebastian queries `/settings/github` | Only sees his own connection (unique index on `user_id`) | P0 |
| 4 | Token persists across projects | Member | Rupesh connected | Rupesh links to Project A, then B | Same `github_connections` row referenced by both project links | P0 |
| 5 | Deleting user token cascades to all project links | Member | Linked to 3 projects | Rupesh deletes his GitHub connection (user-level) | `github_connections` row deleted, CASCADE removes all `project_github_connections` rows | P0 |
| 6 | GitLab token — same isolation | Member | Rupesh connects GitLab | Sebastian queries | Only Rupesh's token visible to Rupesh | P0 |
| 7 | Token refresh/update | Member | Token expired | Re-authorize | `github_connections` row updated, all project links still valid (they reference `connection_id`) | P1 |
| 8 | OAuth grant revoked on disconnect (IKA-687) | Member | Connected | Delete user-level connection | GitHub grant revoked via API, consent screen forced on reconnect | P1 |

---

## 2. Scoping: Project-Level Link/Unlink

> Each project independently enables the user's token via `project_github_connections` junction table: `project_id` + `connection_id`. One row per project (UNIQUE on `project_id`).

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 9 | Connect GitHub to project — user has token | Member | Token exists, project not linked | Click Connect in project settings | POST creates junction row, project shows "Connected @username" | P0 |
| 10 | Connect GitHub to project — user has NO token | Member | Never connected | Click Connect | Pre-auth dialog > OAuth popup > token created > auto-link to this project | P0 |
| 11 | Connect GitLab to project — user has token | Member | GitLab token exists | Click Connect | Junction row created for GitLab | P0 |
| 12 | Connect GitLab to project — no token | Member | Never connected GitLab | Click Connect | OAuth flow > token + project link | P0 |
| 13 | Disconnect GitHub from project | Member | Project linked | Click Unlink > Confirm | Junction row deleted, token stays | P0 |
| 14 | Disconnect GitLab from project | Member | Project linked | Click Unlink > Confirm | Junction row deleted, token stays | P0 |
| 15 | Reconnect after disconnect | Member | Previously linked, then unlinked | Click Connect | If token still exists: instant re-link. If token was deleted: OAuth flow first | P0 |
| 16 | UPSERT: re-link same project | Member | Already linked | POST link again | ON CONFLICT updates `connection_id` + `linked_at`, no error, no duplicate | P1 |
| 17 | Delete project — junction row cascades | Admin | Project has GitHub link | Delete project | `project_github_connections` row CASCADE deleted | P0 |
| 18 | Both providers connected to same project | Member | GitHub + GitLab both linked | View project | Both rows show green Connected badge | P0 |
| 19 | Connect to multiple projects | Member | Token exists | Link to Project A, B, C | Each project has its own junction row, all show Connected | P0 |
| 20 | UNIQUE constraint: one connection per project | System | Rupesh linked to Project A | Sebastian links his connection to same Project A | Rupesh's link overwritten (UPSERT on `project_id`) — latest user wins | P1 |
| 21 | Disconnect dialog message | Member | Connected | Click Unlink | "This will disconnect GitHub from this project only. Your GitHub account stays connected — other projects are not affected." | P0 |
| 22 | Cancel disconnect | Member | Dialog open | Click Cancel | Nothing changes | P0 |
| 23 | Connecting spinner | Member | POST in flight | View button | Spinner + disabled state | P1 |
| 24 | Disconnecting spinner | Member | DELETE in flight | View button | "Disconnecting..." + spinner | P1 |

---

## 3. Scoping: Cross-Project Isolation

> The core requirement: connecting in Project A does NOT affect Project B.

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 25 | Connect in "voice", check "ai-agent" | Member | Just connected to "voice" | Switch to "ai-agent" | "ai-agent" shows "Connect" button — NOT connected | P0 |
| 26 | Connect in "voice", connect in "backend" too | Member | "voice" connected | Select "backend", click Connect | "backend" shows Connected independently (separate junction row) | P0 |
| 27 | Disconnect from "voice", check "backend" | Member | Both connected | Disconnect "voice", switch to "backend" | "backend" still Connected, unaffected | P0 |
| 28 | Five projects, only 2 connected | Member | Projects: voice, ai-agent, backend, frontend, docs | Connect voice + backend only | voice + backend show Connected; ai-agent, frontend, docs show Connect button | P0 |
| 29 | Different teams, same behavior | Member | IKA/voice + BLACK-SHIELD/api both exist | Connect in IKA/voice | BLACK-SHIELD/api shows "Connect" — team boundary doesn't matter, it's project-scoped | P0 |
| 30 | Switch projects rapidly | Member | Multiple projects | Click through voice → backend → frontend fast | Each shows correct independent state without stale data | P1 |
| 31 | Query key isolation | System | TanStack Query | Switch projects | Query key `['project', projectId, 'github']` ensures each project fetches its own status | P0 |
| 32 | Invalidation on connect/disconnect | System | Connected to "voice" | Disconnect | Only `['project', voiceId, 'github']` invalidated, other projects' queries untouched | P0 |
| 33 | URL deep-link preserves correct project state | Member | Connected to "voice" only | Navigate to `?projectId=backend` directly | "backend" shows correct not-connected state | P1 |
| 34 | Page refresh after connect | Member | Just connected to "voice" | Refresh browser | "voice" still shows Connected (server state, not client cache) | P0 |

---

## 4. Scoping: Cross-User Isolation

> Rupesh's connection is invisible and untouchable by Sebastian. Retained from IKA-670 but now applied at project level too.

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 35 | Rupesh connected to Project A, Sebastian views same project | Sebastian | Rupesh linked | GET /settings/projects/{A}/github | Returns null — `user_id` filter in backend | P0 |
| 36 | Sebastian sees "Connect" even though Rupesh is connected | Sebastian | Rupesh linked to Project A | Sebastian views Project A | Shows "Connect" button (Rupesh's link not visible) | P0 |
| 37 | Sebastian connects his own GitHub to same project | Sebastian | Rupesh already linked | Sebastian clicks Connect + OAuth | Sebastian's connection linked, Rupesh's overwritten (UNIQUE `project_id`) | P1 |
| 38 | Sebastian tries to disconnect Rupesh's link via API | Sebastian | Rupesh linked | DELETE /settings/projects/{A}/github | 403 "not your connection" | P0 |
| 39 | Unauthenticated request | Guest | Not logged in | Any project connection endpoint | 401 Unauthorized | P0 |
| 40 | Each user sees their own @username | Both | Both connected to different projects | Rupesh sees @rupeshpanwar, Sebastian sees @sebastianhinz | Correct username per user | P0 |
| 41 | User's access_token never exposed in API response | Member | Connected | Inspect API response | Response includes `github_username` but not raw `access_token` | P0 |
| 42 | OAuth state JWT prevents CSRF attack | Attacker | Crafts fake callback | Fake OAuth redirect | State JWT verification fails, no connection created | P0 |

---

## 5. Mapping: Project to Repository

> After connecting GitHub/GitLab to a project, user links specific repos to that project. Repos are per-project via `project_repos` table.

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 43 | Add Repository — GitHub connected to project | Member | GitHub linked | Click + Add Repository | RepoPickerDialog shows "From GitHub (@username)" option | P0 |
| 44 | Add Repository — GitHub NOT connected to project | Member | Not linked | Click + Add Repository | "From GitHub" option NOT shown, only "From Git Repository" | P0 |
| 45 | Add Repository — both providers connected | Member | GitHub + GitLab linked | Click + Add Repository | Shows "From GitHub", "From GitLab", "From Git Repository" | P0 |
| 46 | Select GitHub repo from picker | Member | Picker open, GitHub stage | Click repo | Repo added to project repo list | P0 |
| 47 | View linked repos | Member | 2 repos linked | View card | Repo rows with display_name + path + delete icon | P0 |
| 48 | Delete repo from project | Member | Repo linked | Click trash | Repo removed | P0 |
| 49 | Empty repos state | Member | No repos | View card | "No repositories configured" | P0 |
| 50 | Same repo linked to multiple projects | Member | Repo X in Project A | Link Repo X to Project B | Both projects show Repo X independently | P1 |
| 51 | Duplicate repo prevention | Member | Repo X already in project | Try to add again | Silently skipped — already in list | P1 |
| 52 | Disconnect GitHub — repos stay | Member | GitHub connected, 2 repos linked | Disconnect GitHub from project | GitHub junction removed, but project_repos rows stay (repos are independent of provider connection) | P1 |

---

## 6. UI: Project Settings Navigation

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 53 | Empty state — no project selected | Member | No projectId | View page | "Select a Project" placeholder | P0 |
| 54 | Select project | Member | Projects loaded | Click project | Right panel loads General Settings + Repositories | P0 |
| 55 | Team grouping | Member | Multiple teams | View left panel | Projects grouped by team with icons + count | P0 |
| 56 | Collapse/expand team | Member | Groups visible | Click chevron | Toggle group | P1 |
| 57 | Loading state | Member | Slow network | Navigate | Spinner in left panel, skeleton in right | P1 |
| 58 | No projects exist | Member | Zero projects | View | "No projects found" empty state | P1 |
| 59 | Deep-link | Member | Have projectId | URL with `?projectId=` | Correct project selected + loaded | P1 |
| 60 | Project name save | Member | Changed name | Click Save | Saved, success alert | P0 |
| 61 | Discard changes | Member | Unsaved name | Click Discard | Reverts | P1 |

---

## 7. UI: Connection Rows in Repositories Card

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 62 | Not connected — Connect button | Member | No link | View | Outline "Connect" button | P0 |
| 63 | Connected — badge + username + unlink | Member | Linked | View | Green "Connected" pill + "@username" + unlink icon | P0 |
| 64 | Pre-auth dialog (first connect) | Member | No user token | Click Connect | Dialog: what GitHub sees, scoped to you, disconnect anytime | P0 |
| 65 | Pre-auth Cancel | Member | Dialog open | Cancel | Closes, no action | P1 |
| 66 | Pre-auth Continue | Member | Dialog open | Continue | OAuth popup opens | P0 |
| 67 | OAuth popup loading | Member | Popup open, /start in flight | View popup | Spinner + "Redirecting to GitHub..." | P1 |
| 68 | OAuth success — popup closes | Member | Authorized | Callback | Popup closes, row refreshes to Connected | P0 |

---

## 8. Edge Cases & Error Handling

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 69 | Popup blocked by browser | Member | Blocker enabled | Click Connect (first time) | window.open returns null, no crash | P0 |
| 70 | User denies OAuth on GitHub | Member | Popup open | Deny on GitHub | Popup closes, error message, row stays "Connect" | P0 |
| 71 | User closes popup manually | Member | Popup open | Close window | Parent detects close, refreshes (still not connected) | P1 |
| 72 | Double-click Connect | Member | Not connected | Click twice fast | Only one popup (same window name) | P1 |
| 73 | Network failure during link | Member | Has token | Network drops on POST | Error state, button re-enabled | P1 |
| 74 | Stale token — GitHub revoked externally | Member | Connected | Fetch repos via Add Repository | 502 from GitHub, error in picker | P1 |
| 75 | API 500 on connect | Member | Backend error | Click Connect | Error shown, row stays "Connect" | P2 |
| 76 | API 404 on disconnect (no link exists) | Member | Not actually connected | Direct DELETE call | 404 "no connection linked" | P2 |
| 77 | Slow API response | Member | High latency | Connect | Spinner for duration, no timeout crash | P2 |
| 78 | Refresh page mid-connect | Member | OAuth popup open | Refresh parent | Popup stays open, user can complete; parent reloads fresh state | P1 |

---

## Data Model Reference

```
github_connections (user-level token — IKA-670)
├── id (PK)
├── user_id (UNIQUE — one per user)
├── access_token
├── github_username
└── connected_at / updated_at

project_github_connections (project-level link — IKA-690)
├── id (PK)
├── project_id (UNIQUE — one connection per project)
├── connection_id (FK → github_connections.id, CASCADE)
└── linked_at

Same structure for gitlab_connections / project_gitlab_connections.
```

**CASCADE behavior:**
- Delete `github_connections` row → all `project_github_connections` rows referencing it are deleted
- Delete `projects` row → `project_github_connections` row for that project is deleted
- Delete `project_github_connections` row → `github_connections` row is NOT deleted (token stays)

---

## Implementation Checklist

- [ ] All P0 scenarios implemented and tested
- [ ] Cross-project isolation verified: connect in A, switch to B → shows "Connect"
- [ ] Cross-user isolation verified: Rupesh connects, Sebastian sees nothing
- [ ] Disconnect dialog clearly says "this project only"
- [ ] CASCADE deletes verified (delete project, delete user connection)
- [ ] OAuth flow works for first-time + returning users
- [ ] RepoPickerDialog shows/hides "From GitHub" based on project-level connection (not user-level)
