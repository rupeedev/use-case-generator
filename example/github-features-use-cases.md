# Use Cases: Project-Level GitHub/GitLab Connection

## Scenario Breakdown

```
======================================================================
Feature Area                                             Scenarios
======================================================================
  1. Project Settings Navigation                               11
  2. GitHub/GitLab Connect (per project)                       14
  3. GitHub/GitLab Disconnect (per project)                     8
  4. Repository Linking (per project)                           8
  5. Cross-Project Isolation                                    7
  6. Edge Cases & Error Handling                               10
  7. Security                                                   6
======================================================================
                                                  TOTAL        64
======================================================================

Priority Breakdown:
  P0 (must-have):      42
  P1 (should-have):    17
  P2 (nice-to-have):    5
```

---

## Summary
- **Core entity:** Project GitHub/GitLab connection link
- **Primary actors:** Workspace Member, Admin, Owner
- **Reference UI:** `Settings > Projects` — split-panel (left: project list grouped by team, right: General Settings + Repositories card with GitHub/GitLab rows)
- **Total scenarios:** 64

---

## 1. Project Settings Navigation

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 1 | No project selected — empty state | Member | Navigate to Settings > Projects, no projectId in URL | View page | Right panel shows "Select a Project" placeholder with folder icon | P0 |
| 2 | Select project from left panel | Member | Projects loaded | Click "voice" under IKA team | Right panel shows General Settings + Repositories card; URL updates to `?projectId=...` | P0 |
| 3 | Switch between projects | Member | "voice" selected | Click "backend" | Right panel refreshes with backend's settings and connection state | P0 |
| 4 | Team grouping in project list | Member | Projects in IKA and BLACK-SHIELD teams | View left panel | Projects grouped under collapsible team headers with icon + name + count | P0 |
| 5 | Collapse/expand team group | Member | Team groups visible | Click team header chevron | Group toggles, selected project stays highlighted | P1 |
| 6 | Deep-link to specific project | Member | Have projectId | Navigate to `/settings/projects?projectId=abc` | Left panel highlights that project, right panel loads its config | P1 |
| 7 | Projects loading state | Member | Slow network | Navigate to Projects settings | Left panel shows spinner, right panel shows skeleton | P1 |
| 8 | No projects exist | Member | Zero projects | View page | Left panel: "No projects found — Create a project to get started" | P1 |
| 9 | Repositories card layout | Member | Project selected, no repos | View right panel | GitHub row, GitLab row, "No repositories configured", "+ Add Repository" button — in that order | P0 |
| 10 | Project name edit + save | Member | Project selected | Change name, click Save | Name saved, success alert shown | P0 |
| 11 | Discard unsaved name change | Member | Name changed, not saved | Click Discard | Name reverts to original | P1 |

---

## 2. GitHub/GitLab Connect (per project)

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 12 | GitHub row shows "Connect" when not linked | Member | Project selected, no GitHub link | View Repositories card | GitHub row: icon + "GitHub" + "Connect" outline button | P0 |
| 13 | GitLab row shows "Connect" when not linked | Member | Same, no GitLab link | View Repositories card | GitLab row: icon + "GitLab" + "Connect" outline button | P0 |
| 14 | Connect GitHub — user has NO OAuth token (first time) | Member | Never connected GitHub anywhere | Click Connect on GitHub row | Pre-auth dialog > Continue > OAuth popup > authorize > token created + auto-linked to this project | P0 |
| 15 | Connect GitHub — user already has OAuth token (other project) | Member | Connected in another project, this one not linked | Click Connect on GitHub row | No popup — instant link via POST, row flips to Connected @username | P0 |
| 16 | Connect GitLab — first time | Member | No GitLab token | Click Connect on GitLab row | Pre-auth dialog > OAuth popup > token + project link created | P0 |
| 17 | Connect GitLab — already has token | Member | Connected elsewhere | Click Connect | Instant link, row flips to Connected @username | P0 |
| 18 | Connected state shows badge + username + unlink icon | Member | GitHub connected to this project | View Repositories card | Green "Connected" pill + "@rupeshpanwar" + unlink icon | P0 |
| 19 | Pre-auth dialog content | Member | No token, clicked Connect | Dialog opens | Explains: "iKanban will open GitHub in a popup", "Only your own repositories", "Scoped to you", "Disconnect anytime" | P0 |
| 20 | Pre-auth dialog — Cancel | Member | Dialog open | Click Cancel | Dialog closes, nothing changes | P1 |
| 21 | Pre-auth dialog — Continue | Member | Dialog open | Click Continue | OAuth popup opens, dialog closes | P0 |
| 22 | OAuth popup loading page | Member | Popup opened, /start in flight | View popup | Spinner + "Redirecting to GitHub..." + hint text | P1 |
| 23 | OAuth success — popup auto-closes | Member | Authorized on GitHub | Callback completes | Popup closes, parent window refetches, row shows Connected | P0 |
| 24 | Connecting spinner on button | Member | POST link in flight | View Connect button | Button shows spinner, disabled | P1 |
| 25 | Both providers connected | Member | GitHub and GitLab linked | View Repositories card | Both rows show green Connected badge with respective usernames | P0 |

---

## 3. GitHub/GitLab Disconnect (per project)

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 26 | Click unlink icon | Member | GitHub connected to this project | Click unlink icon on GitHub row | Confirmation dialog opens | P0 |
| 27 | Disconnect dialog message | Member | Dialog open | Read message | "This will disconnect GitHub from this project only. Your GitHub account stays connected — other projects are not affected." | P0 |
| 28 | Confirm disconnect | Member | Dialog open | Click "Disconnect" | Junction row deleted, row flips back to "Connect" button | P0 |
| 29 | Cancel disconnect | Member | Dialog open | Click "Cancel" | Dialog closes, nothing changes | P0 |
| 30 | Disconnect spinner | Member | Clicked Disconnect, API in flight | View button | "Disconnecting..." with spinner | P1 |
| 31 | Disconnect does NOT delete user's OAuth token | Member | Connected to this project + others | Disconnect from this project | User's `github_connections` row untouched, only junction row removed | P0 |
| 32 | Disconnect GitLab — same flow | Member | GitLab connected | Click unlink > Confirm | GitLab junction row removed, user token stays | P0 |
| 33 | Reconnect after disconnect | Member | Previously connected, then disconnected | Click Connect | If user still has token: instant re-link. If token was deleted: OAuth flow | P0 |

---

## 4. Repository Linking (per project)

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 34 | Add Repository — GitHub connected | Member | GitHub connected to this project | Click + Add Repository | RepoPickerDialog shows "From GitHub (@username)", "From GitLab" (if connected), "From Git Repository" | P0 |
| 35 | Add Repository — no provider connected | Member | Neither connected | Click + Add Repository | Only "From Git Repository" option shown | P0 |
| 36 | Select GitHub repo from picker | Member | GitHub stage open | Click a repo | Repo linked to project, appears in repo list | P0 |
| 37 | View linked repos | Member | 2 repos linked | View Repositories card | Repo list with display_name + path + delete icon per row | P0 |
| 38 | Delete repo from project | Member | Repo linked | Click trash icon | Repo removed from project list | P0 |
| 39 | Empty repos state | Member | No repos linked | View Repositories card | "No repositories configured" text | P0 |
| 40 | Duplicate repo prevention | Member | Repo X already linked | Try to add Repo X again | Skipped — already in list | P1 |
| 41 | Adding repo spinner | Member | Add in progress | View Add Repository button | Spinner shown, button disabled | P1 |

---

## 5. Cross-Project Isolation

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 42 | Connect in "voice", switch to "ai-agent" | Member | Just connected GitHub to "voice" | Click "ai-agent" | "ai-agent" shows GitHub as NOT connected | P0 |
| 43 | Connect in "voice" AND "backend" independently | Member | "voice" connected | Select "backend", click Connect | "backend" also shows Connected (same @username, separate junction row) | P0 |
| 44 | Disconnect from "voice", "backend" unaffected | Member | Both connected | Disconnect from "voice", switch to "backend" | "backend" still shows Connected | P0 |
| 45 | Rupesh connected in Project A, Sebastian sees nothing | Member | Rupesh linked GitHub to Project A | Sebastian views same project | Sebastian sees "Connect" button — Rupesh's connection not visible | P0 |
| 46 | Sebastian connects his own GitHub to same project | Member | Rupesh already linked | Sebastian clicks Connect, authorizes his GitHub | Sebastian sees his @username, Rupesh's link overwritten (UNIQUE project_id) | P1 |
| 47 | User deletes global GitHub connection — all project links cascade | Member | Linked to 3 projects | DELETE /settings/github (user-level disconnect) | github_connections row gone, CASCADE deletes all project_github_connections rows | P0 |
| 48 | Delete project — junction row cleaned up | Admin | Project has GitHub link | Delete project | project_github_connections row CASCADE deleted | P0 |

---

## 6. Edge Cases & Error Handling

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 49 | OAuth popup blocked by browser | Member | Popup blocker enabled | Click Connect (first time) | window.open returns null, no crash | P0 |
| 50 | User denies OAuth on GitHub consent screen | Member | Popup opened | Click Deny on GitHub | Popup closes, callback sends error message, row stays "Connect" | P0 |
| 51 | User closes popup before completing OAuth | Member | Popup open | Manually close popup | Parent detects close, refreshes (still not connected) | P1 |
| 52 | Double-click Connect button | Member | Not connected | Click Connect twice fast | Only one popup opens (same window name) | P1 |
| 53 | Network failure during link API call | Member | Has token, clicks Connect | Network drops | Error state, button re-enabled, user can retry | P1 |
| 54 | GitHub token expired/revoked externally | Member | Connected, token stale | Fetch available repos via Add Repository | 502 from GitHub API, error shown in picker | P1 |
| 55 | Connect to project that already has a link (UPSERT) | Member | Already linked | POST link again | ON CONFLICT updates connection_id + linked_at, no error | P1 |
| 56 | Disconnect from project with no link | Member | Not connected | Call DELETE API directly | 404 "no connection linked" | P2 |
| 57 | API returns 500 on connect | Member | Backend error | Click Connect (has token) | Error toast or inline error, row stays "Connect" | P2 |
| 58 | Very slow API response | Member | High latency | Click Connect | Spinner shown for duration, no timeout crash | P2 |

---

## 7. Security

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 59 | Unauthenticated user calls project connection API | Guest | Not logged in | GET /settings/projects/{id}/github | 401 Unauthorized | P0 |
| 60 | Sebastian tries to read Rupesh's connection via project endpoint | Member | Rupesh linked | Sebastian calls GET | Returns null (user_id filter in backend) | P0 |
| 61 | Sebastian tries to disconnect Rupesh's link | Member | Rupesh linked | Sebastian calls DELETE | 403 "not your connection" | P0 |
| 62 | OAuth state JWT prevents CSRF | Attacker | Crafts fake callback | Fake callback URL | State JWT verification fails, connection not created | P0 |
| 63 | Access token never exposed to frontend | Member | Connected | View API response | Response includes username but access_token is omitted or irrelevant to frontend | P0 |
| 64 | CASCADE delete doesn't leak tokens | System | User deletes account | Account deleted | github_connections row deleted, all project links cascade, no orphaned tokens | P0 |

---

## Implementation Checklist

- [ ] All P0 scenarios implemented
- [ ] All P0 scenarios tested (manual or automated)
- [ ] All P1 scenarios implemented
- [ ] Edge cases have proper error messages
- [ ] Disconnect dialog clearly states "this project only"
- [ ] Cross-project isolation verified with 2+ projects
- [ ] Multi-user isolation verified (Rupesh vs Sebastian)
- [ ] CASCADE deletes verified for project deletion and user connection deletion
