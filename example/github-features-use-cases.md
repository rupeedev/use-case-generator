# Use Cases: GitHub Features in iKanban

## Summary
- **Core entities:** GitHub Connection, GitHub Repository, GitHub App, PR, Document Sync, Agent Assignment
- **Primary actors:** Workspace Owner, Admin, Member, AI Agent (Claude/Copilot/Gemini)
- **Scope dimensions:** User-level (OAuth token), Project-level (connection link), Team-level (repos/sync), Organization-level (GitHub App)
- **Total scenarios:** 170 (28 UI-specific + 142 feature-level)
- **Reference UI:** Settings > Projects (split-panel: left = project list grouped by team, right = General Settings + Repositories card with GitHub/GitLab connection rows)

---

## Feature Area 0: Project Settings Page — UI Navigation & State

> Based on: `dev-i-kanban.com/settings/projects?projectId=...`
> Left panel shows projects grouped by team (BLACK-SHIELD, IKA). Right panel shows General Settings + Repositories card for the selected project.

### Category 6: UI/UX States

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| U1 | No project selected — empty state | Member | Navigate to Settings > Projects, no projectId in URL | View page | Right panel shows "Select a Project" placeholder with folder icon | P0 |
| U2 | Select project from left panel | Member | Projects loaded | Click "voice" under IKA team | Right panel shows voice's General Settings + Repositories; URL updates to `?projectId=...` | P0 |
| U3 | Switch from "voice" to "backend" | Member | "voice" selected, GitHub connected there | Click "backend" | Right panel refreshes — "backend" may show different connection state (not connected) | P0 |
| U4 | Connection state differs per project | Member | GitHub connected to "voice", not "backend" | Switch between them | "voice" shows green Connected badge, "backend" shows Connect button | P0 |
| U5 | Team grouping in project list | Member | Projects exist in IKA and BLACK-SHIELD teams | View left panel | Projects grouped under collapsible team headers with icons + names | P0 |
| U6 | Collapse/expand team group | Member | Team groups visible | Click team header chevron | Group collapses/expands, selected project stays highlighted | P1 |
| U7 | Project count shown per team | Member | IKA has 4 projects | View left panel | Team header shows "4" count | P1 |
| U8 | Ripple effect on project click | Member | Projects visible | Click a project | Material-style ripple animation plays on the clicked row | P2 |
| U9 | Deep-link to specific project | Member | Have projectId | Navigate to `/settings/projects?projectId=abc` | Left panel highlights that project, right panel loads its config | P1 |
| U10 | Projects loading skeleton | Member | Slow network | Navigate to Projects settings | Left panel shows spinner, right panel shows skeleton | P1 |
| U11 | No projects exist | Member | No projects created | View page | Left panel shows empty state: "No projects found — Create a project to get started" | P1 |

### Category 1: Happy Path — GitHub/GitLab Rows in Repositories Card

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| U12 | GitHub row shows "Connect" button when not linked | Member | "voice" selected, no project-level GitHub link | View Repositories card | GitHub row: icon + "GitHub" label + "Connect" outline button on right | P0 |
| U13 | GitLab row shows "Connect" button when not linked | Member | Same, no GitLab link | View Repositories card | GitLab row: icon + "GitLab" label + "Connect" outline button on right | P0 |
| U14 | GitHub row shows Connected badge after linking | Member | Clicked Connect, OAuth completed or already had token | View Repositories card | GitHub row: icon + "GitHub" + green "Connected" pill + "@username" + unlink icon | P0 |
| U15 | Click Connect when user already has OAuth token | Member | User connected GitHub before (other project), this project not linked | Click Connect on GitHub row | No OAuth popup — instant link, row flips to Connected | P0 |
| U16 | Click Connect when user has NO OAuth token | Member | Never connected GitHub | Click Connect | Pre-auth dialog appears explaining what happens > Continue > OAuth popup > connect + auto-link | P0 |
| U17 | Click Unlink icon on Connected row | Member | GitHub connected to this project | Click unlink icon | Confirmation dialog: "This will disconnect GitHub from this project only. Your GitHub account stays connected — other projects are not affected." | P0 |
| U18 | Confirm disconnect | Member | Disconnect dialog open | Click "Disconnect" | Junction row deleted, row flips back to "Connect" button, other projects unaffected | P0 |
| U19 | Cancel disconnect | Member | Disconnect dialog open | Click "Cancel" | Dialog closes, nothing changes | P0 |
| U20 | Repos list below connection rows | Member | 2 repos linked to project | View Repositories card | GitHub/GitLab rows at top, then repo list with display_name + path + delete icon, then "+ Add Repository" button | P0 |
| U21 | Empty repos state | Member | No repos linked | View Repositories card | "No repositories configured" text between connection rows and Add Repository button | P0 |
| U22 | Add Repository when GitHub connected | Member | GitHub connected to this project | Click + Add Repository | RepoPickerDialog shows: "From GitHub" (with @username), "From GitLab" (if connected), "From Git Repository" | P0 |
| U23 | Add Repository when neither provider connected | Member | Neither GitHub nor GitLab connected | Click + Add Repository | RepoPickerDialog only shows "From Git Repository" option | P0 |

### Category 4: State Transitions — Cross-Project Switching

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| U24 | Connect GitHub in "voice", switch to "ai-agent" | Member | Just connected GitHub to "voice" | Click "ai-agent" in left panel | "ai-agent" shows GitHub as NOT connected (independent project scope) | P0 |
| U25 | Connect GitHub in "voice", connect in "backend" too | Member | "voice" connected | Select "backend", click Connect | "backend" also shows Connected (same @username, separate junction row) | P0 |
| U26 | Disconnect from "voice", check "backend" | Member | Both "voice" and "backend" connected | Disconnect from "voice", switch to "backend" | "backend" still shows Connected, unaffected | P0 |
| U27 | Save project name, then switch project | Member | Changed "voice" name, unsaved | Click "backend" | Unsaved changes warning? Currently no guard — name resets to "backend"'s name | P1 |
| U28 | General Settings + Repositories both visible | Member | Project selected | Scroll right panel | General Settings card (name, save) above Repositories card (connections + repos) | P0 |

---

## Feature Area 1: GitHub OAuth Connection (User-Level)

### Category 1: Happy Path (Core CRUD)

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 1 | Rupesh connects GitHub account for the first time | Member | No GitHub connection exists | Click Connect > Authorize in popup | Connection created, `@rupeshpanwar` badge shown | P0 |
| 2 | Rupesh views existing GitHub connection | Member | GitHub connected | Navigate to project settings | Shows "Connected @rupeshpanwar" with Disconnect button | P0 |
| 3 | Rupesh disconnects GitHub account | Member | GitHub connected | Click Disconnect > Confirm | Connection deleted, OAuth grant revoked on GitHub side, badge gone | P0 |
| 4 | Rupesh reconnects after previous disconnect | Member | Previously connected then disconnected | Click Connect > Authorize | New connection created, fresh token stored | P0 |
| 5 | Rupesh updates GitHub connection (re-authorize) | Member | GitHub connected, token expired | Click Connect again | Token refreshed, connection updated | P1 |

### Category 2: Scope Variations

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 6 | User-scoped: Rupesh's connection is invisible to Sebastian | Member | Rupesh connected to GitHub | Sebastian navigates to same project settings | Sebastian sees "Not Connected" — Rupesh's token not leaked | P0 |
| 7 | Project-scoped: Rupesh connects GitHub to Project A only | Member | Rupesh has GitHub OAuth token | Connect GitHub in Project A settings | Project A shows Connected, Project B shows Not Connected | P0 |
| 8 | Project-scoped: Rupesh disconnects from Project A | Member | GitHub linked to Project A and B | Disconnect from Project A | Project A shows Not Connected, Project B still Connected, user token preserved | P0 |
| 9 | Cross-team: Rupesh connects GitHub to projects in different teams | Member | Member of Team IKA and Team SCH | Connect in IKA/voice, then SCH/backend | Both projects independently show Connected | P1 |
| 10 | Workspace-scoped: Rupesh's connection persists across workspace switch | Member | Connected in Workspace 1 | Switch to Workspace 2 | Connection data persists (user-level), but projects in WS2 need separate linking | P1 |

### Category 3: Permission & Access Control

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 11 | Owner connects GitHub | Owner | No connection | Connect GitHub | Succeeds, full access | P0 |
| 12 | Admin connects GitHub | Admin | No connection | Connect GitHub | Succeeds, scoped to their user | P0 |
| 13 | Member connects GitHub | Member | No connection | Connect GitHub | Succeeds, scoped to their user | P0 |
| 14 | Unauthenticated user attempts GitHub connection | Guest | Not logged in | Direct API call to /settings/github | 401 Unauthorized | P0 |
| 15 | Sebastian tries to disconnect Rupesh's project connection | Member | Rupesh connected GitHub to Project A | Sebastian calls DELETE /settings/projects/{id}/github | 403 Forbidden — "not your connection" | P0 |
| 16 | Sebastian tries to read Rupesh's GitHub token via project endpoint | Member | Rupesh connected | Sebastian calls GET /settings/projects/{id}/github | Returns null (filtered by user_id check) | P0 |

### Category 4: State Transitions

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 17 | Connected > Disconnect > Reconnect: are project links preserved? | Member | GitHub connected to 3 projects, then disconnected | Reconnect GitHub OAuth | New connection created, but project links are gone (CASCADE delete on github_connections) | P0 |
| 18 | Connected > GitHub token revoked externally | Member | Connected, then user revokes app on github.com/settings | Next API call (e.g., fetch repos) | 502 Bad Gateway from GitHub API, UI shows error | P1 |
| 19 | Connected > GitHub account deleted/suspended | Member | Connected | GitHub API returns 401 | Connection becomes stale, user sees error on repo fetch | P2 |
| 20 | Project linked > User deletes their connection | Member | Project A linked to GitHub | User calls DELETE /settings/github (user-level) | Connection deleted, CASCADE removes project link row too | P0 |
| 21 | Project linked > Project deleted | Admin | Project A has GitHub link | Delete Project A | project_github_connections row CASCADE deleted | P0 |

### Category 5: Edge Cases & Error Handling

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 22 | OAuth popup blocked by browser | Member | Popup blocker enabled | Click Connect | window.open returns null, no crash, user should see error or try again | P0 |
| 23 | User denies OAuth permission on GitHub consent screen | Member | Popup opened | Click "Deny" on GitHub | Popup closes, callback receives error, UI stays as "Not Connected" | P0 |
| 24 | OAuth popup closed before completing flow | Member | Popup opened | User manually closes popup | checkOAuthWindow interval detects close, refreshes connection (still null) | P1 |
| 25 | Double-click Connect button | Member | No connection | Click Connect twice rapidly | Only one popup opens (same window name "github-oauth") | P1 |
| 26 | Network failure during OAuth token exchange | Member | Popup reached GitHub | Network drops during callback | Backend logs error, popup shows error message | P1 |
| 27 | User tries to connect but already has a connection (409 Conflict) | Member | Already connected | POST /settings/github again | 409 "GitHub connection already exists. Use PUT to update." | P1 |
| 28 | Connect to project when user has no OAuth token yet | Member | No GitHub connection | Click Connect in project settings | Pre-auth dialog > OAuth flow > auto-link to project after success | P0 |
| 29 | GitHub API rate limit during available repos fetch | Member | Connected | Fetch available repos (GET /user/repos) | 502 from backend, user sees error "Failed to load GitHub repositories" | P2 |
| 30 | User has 200+ GitHub repos | Member | Connected, many repos | Fetch available repos | Only first 100 returned (per_page=100), user can search/filter | P2 |

### Category 6: UI/UX States

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 31 | Loading state while fetching connection status | Member | Page loading | Navigate to project settings | Skeleton/spinner shown while query resolves | P0 |
| 32 | Connected state with username badge | Member | GitHub connected to this project | View project settings | Green "Connected" badge + "@username" + Unlink icon | P0 |
| 33 | Not connected state with Connect button | Member | No project-level link | View project settings | "Connect" outline button shown | P0 |
| 34 | Disconnect confirmation dialog | Member | Connected | Click Unlink icon | Dialog: "This will disconnect GitHub from this project only. Your GitHub account stays connected — other projects are not affected." | P0 |
| 35 | Pre-auth explanation dialog (first-time connect) | Member | No user-level connection | Click Connect | Dialog explaining: what GitHub will see, scoped to you, disconnect anytime | P0 |
| 36 | Connecting spinner state | Member | Clicked Connect for project | API call in flight | Button shows spinner, disabled | P1 |
| 37 | Disconnecting spinner state | Member | Clicked Disconnect confirm | API call in flight | Button shows "Disconnecting..." with spinner | P1 |
| 38 | OAuth popup loading page | Member | Popup opened, /start API in flight | Popup shows loading | Spinner + "Redirecting to GitHub..." + hint text | P1 |

---

## Feature Area 2: Project Repository Linking

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 39 | Add GitHub repo to project | Member | GitHub connected to project | Click Add Repository > From GitHub > Select repo | Repo appears in project's repository list | P0 |
| 40 | Add local git repo to project | Member | Local repo exists | Click Add Repository > From Git Repository > Browse | Repo registered and linked to project | P0 |
| 41 | Remove repo from project | Member | Repo linked to project | Click trash icon on repo row | Repo removed from project list | P0 |
| 42 | View project repositories | Member | 3 repos linked | Navigate to project settings | All 3 repos listed with display name + path | P0 |

### Category 2: Scope Variations

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 43 | Same GitHub repo linked to multiple projects | Member | Repo X linked to Project A | Link Repo X to Project B too | Both projects show Repo X independently | P1 |
| 44 | Project has repos from both GitHub and local filesystem | Member | GitHub connected | Add one GitHub repo + one local repo | Both appear in project repo list, different path formats (URL vs local path) | P1 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 45 | Add repository that's already linked | Member | Repo X already in project | Try to add Repo X again | Duplicate check prevents re-add (already linked) | P1 |
| 46 | RepoPickerDialog with no GitHub connection | Member | Not connected to GitHub | Open Add Repository | "From GitHub" option not shown, only "From Git Repository" | P0 |
| 47 | RepoPickerDialog with GitHub + GitLab both connected | Member | Both connected | Open Add Repository | Shows "From GitHub", "From GitLab", and "From Git Repository" options | P1 |
| 48 | Delete project with linked repos | Admin | Project has 3 repos | Delete project | project_repos rows CASCADE deleted, repos table unchanged | P1 |

---

## Feature Area 3: GitHub App (Organization-Level)

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 49 | Install GitHub App for organization | Org Admin | Organization exists, no app installed | Settings > Install GitHub App | Redirects to GitHub App install page, callback saves installation | P0 |
| 50 | View GitHub App installation status | Org Admin | App installed | Navigate to org settings | Shows installed status, repository count, account login | P0 |
| 51 | Uninstall GitHub App | Org Admin | App installed | Click Uninstall > Confirm | Installation removed from DB | P0 |
| 52 | List accessible repositories from GitHub App | Org Admin | App installed | View repository list | Shows all repos the app can access | P1 |
| 53 | Enable PR review for a repository | Org Admin | App installed, repo listed | Toggle review_enabled | PR events for that repo trigger automated review | P1 |
| 54 | Disable PR review for a repository | Org Admin | Review enabled | Toggle review_enabled off | PR events no longer trigger review | P1 |
| 55 | Bulk enable/disable PR review | Org Admin | Multiple repos | PATCH bulk endpoint | All selected repos updated | P2 |

### Category 3: Permissions

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 56 | Non-admin tries to install GitHub App | Member | Not org admin | Call install endpoint | Forbidden or installation fails on GitHub side (no org admin perms) | P0 |
| 57 | Personal org (not allowed) | Owner | Personal GitHub account, not org | Try to install app | Error: GitHub Apps can only be installed on organizations | P1 |

### Category 4: State Transitions

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 58 | App installed > Suspended on GitHub side | External | GitHub admin suspends app | Webhook: installation.suspended | suspended_at field updated, features disabled | P1 |
| 59 | App suspended > Unsuspended | External | App was suspended | Webhook: installation.unsuspend | suspended_at cleared, features re-enabled | P2 |
| 60 | App installed > Repository added to installation | External | App installed with specific repos | User adds repo on GitHub settings | Webhook updates repo list | P2 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 61 | OAuth state token expired during app install | User | Started install flow, waited > 15 min | Complete callback | State token expired, error shown, user must retry | P1 |
| 62 | Webhook with invalid signature | Attacker | No valid HMAC | POST /github/webhook with bad signature | 401 Unauthorized, constant-time comparison prevents timing attacks | P0 |
| 63 | Duplicate webhook delivery | GitHub | Network retry | Same webhook delivered twice | Idempotent handling, no duplicate installations | P1 |

---

## Feature Area 4: Document Sync with GitHub

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 64 | Configure sync path for a repository | Member | Repo linked to team | Configure sync: folder_id + sync_path | Sync config saved to github_repositories | P0 |
| 65 | Push documents to GitHub | Member | Sync configured | Click Push | Documents committed to repo at configured path | P0 |
| 66 | Pull documents from GitHub | Member | Sync configured, remote has changes | Click Pull | Remote documents imported/updated in workspace | P0 |
| 67 | Bidirectional sync (pull then push) | Member | Sync configured | Click Sync | Pull first (get remote changes), then push (send local changes) | P0 |
| 68 | Clear sync configuration | Member | Sync configured | Clear sync | sync_path and sync_folder_id set to null | P1 |
| 69 | Configure multi-folder sync | Member | Repo linked | Set up multiple folder mappings | Multiple folder-to-path mappings saved | P1 |
| 70 | Clear multi-folder sync | Member | Multi-folder configured | Clear all folder syncs | All sync configs removed for this repo | P1 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 71 | Push with no documents in folder | Member | Sync configured, folder empty | Push | Empty commit or no-op, no error | P1 |
| 72 | Pull when remote branch doesn't exist | Member | Configured to non-existent branch | Pull | Error: branch not found | P1 |
| 73 | Push with merge conflict on remote | Member | Both local and remote changed same file | Push | Conflict error, user must resolve | P2 |
| 74 | Sync with revoked GitHub token | Member | Token expired/revoked | Push or Pull | 502 error, prompt to reconnect | P1 |
| 75 | Sync with very large files (>100MB) | Member | Large binary in docs | Push | GitHub rejects (file size limit), error shown | P2 |

### Category 7: Data Integrity

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 76 | Disconnect GitHub: are sync configs preserved? | Member | Sync configured, disconnect GitHub | Disconnect project | project_github_connections row deleted, sync config on github_repositories stays (orphaned but harmless) | P1 |
| 77 | Delete repo link: is sync config cleaned up? | Member | Sync configured | Unlink repository | github_repositories row deleted (CASCADE), sync config gone | P0 |

---

## Feature Area 5: Agent Assignments (@claude, @copilot, @gemini)

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 78 | Assign task to @claude via comment | Member | Task exists, Claude agent configured | Type "@claude fix the login bug" in comment | Claude assignment created, GitHub issue opened, Claude Code Action triggered | P0 |
| 79 | Assign task to @copilot | Member | Copilot configured | "@copilot implement dark mode" | Copilot assignment created, GitHub issue opened | P0 |
| 80 | Assign task to @gemini | Member | Gemini configured | "@gemini review this code" | Gemini assignment created, GitHub issue opened | P0 |
| 81 | View Claude assignments for a task | Member | Assignments exist | GET /tasks/{id}/claude-assignments | List of all Claude assignments returned | P1 |
| 82 | View latest Claude assignment | Member | Multiple assignments | Check latest | Most recent assignment returned with status | P1 |

### Category 4: State Transitions

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 83 | Agent creates PR from assignment | AI Agent | Assignment created, GitHub issue exists | Agent pushes code, creates PR | PR linked to task, auto-merge workflow may trigger | P0 |
| 84 | Agent PR passes CI checks | CI | PR created by agent | All checks pass | auto-merge-agent-prs.yml marks ready, approves, merges | P0 |
| 85 | Agent PR fails CI checks | CI | PR created | Checks fail | PR not auto-merged, remains open for human review | P1 |
| 86 | Agent re-assigned to same task | Member | Previous assignment completed | "@claude also fix the logout bug" | New assignment created, new GitHub issue | P1 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 87 | @claude on a task with no GitHub repo | Member | No repo linked to project | "@claude fix it" | Assignment created but agent may fail to find repo context | P1 |
| 88 | Bot-triggering-bot loop prevention | AI Agent | Agent comments on its own issue | Webhook fires | Workflow checks: if author is bot, skip (prevents infinite loops) | P0 |
| 89 | Concurrent @claude and @copilot on same task | Member | Both mentioned | Both trigger simultaneously | Both assignments created, concurrency group prevents parallel GitHub runs | P1 |
| 90 | @claude on closed issue | AI Agent | Issue closed | @claude comment | Workflow reopens the issue to process assignment | P1 |

---

## Feature Area 6: PR Creation from Tasks

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 91 | Create PR from task attempt | Member | Task has an attempt with code changes | Open CreatePRDialog > Fill title, description, target branch > Create | PR created on GitHub, link saved to task | P0 |
| 92 | Create draft PR | Member | Attempt ready | Check "Draft" option > Create | Draft PR created, not ready for review | P1 |
| 93 | Auto-generate PR description | Member | Attempt has changes | Click auto-generate | Description populated from commit messages/diff summary | P1 |
| 94 | Select target branch for PR | Member | Repo has multiple branches | Choose branch from dropdown | PR targets selected branch | P0 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 95 | Create PR with no changes | Member | Attempt has no commits | Try to create PR | Error: no commits to create PR from | P1 |
| 96 | Create PR when branch already has open PR | Member | PR exists for this branch | Try to create another | Error or redirect to existing PR | P1 |
| 97 | GitHub CLI not installed (macOS) | Member | No `gh` CLI | Open CreatePRDialog | Shows setup instructions: Homebrew install guide | P1 |

---

## Feature Area 7: Branch & Merge Operations

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 98 | List branches for a repo | Member | Repo linked, branches exist | Open branch selector | All branches listed, cached for 60s | P0 |
| 99 | Select working branch for attempt | Member | Branches loaded | Select branch | Branch saved as attempt's working branch | P0 |
| 100 | Change target branch | Member | Attempt has target branch set | Use useChangeTargetBranch hook | Target branch updated | P1 |
| 101 | Merge PR | Member | PR is approved and passing | Click Merge | PR merged, branch status + repo queries invalidated | P0 |
| 102 | Rebase branch | Member | Branch behind target | Click Rebase | Branch rebased onto target | P1 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 103 | Merge with conflicts | Member | PR has conflicts | Try to merge | Error: conflicts must be resolved first | P1 |
| 104 | Branch deleted on GitHub but still cached locally | Member | Branch existed 30s ago | Select branch | Stale cache, may error on operations; refetches after cache expires | P2 |
| 105 | Repo with 100+ branches | Member | Large repo | Open branch selector | All branches loaded, may need search/filter | P2 |

---

## Feature Area 8: GitHub Workflows (CI/CD)

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 106 | Claude Code Action triggered by @claude issue | CI | Issue with @claude mention | Workflow triggers | Claude processes issue, creates PR if needed | P0 |
| 107 | Gemini Action triggered by @gemini issue | CI | Issue with @gemini mention | Workflow triggers | Gemini processes issue | P0 |
| 108 | Auto-merge agent PR | CI | Agent PR, all checks pass | auto-merge-agent-prs.yml | PR marked ready > approved > merged automatically | P0 |
| 109 | Backend deploy on push to main | CI | Code pushed to main | deploy-ecr.yml triggers | Docker build > ECR push > ECS deploy | P0 |
| 110 | Frontend deploy on push to main | CI | Code pushed to main | deploy-frontend.yml triggers | Build > S3 upload > CloudFront invalidation | P0 |
| 111 | Dashboard JSON deploy | CI | Dashboard JSON changed in push | deploy-dashboards.yml | Grafana API upload | P1 |
| 112 | DORA metrics ingestion | CI (scheduled) | Monday 6:30 UTC | dora-metrics-ingestion.yml | Commits, PRs, pipelines data ingested | P1 |
| 113 | Security scan | CI (scheduled) | On schedule or manual trigger | security-scan.yml | npm audit + cargo audit results | P1 |

### Category 5: Edge Cases

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 114 | Claude workflow: existing PR already open for issue | CI | PR exists | @claude trigger | Workflow checks for existing PR, comments on existing instead of creating new | P0 |
| 115 | Auto-merge: checks don't pass within 10 minutes | CI | Agent PR, checks slow | Wait timeout | Workflow exits without merging, PR stays open | P1 |
| 116 | Concurrent workflow runs for same issue | CI | Multiple @claude mentions | Concurrency group | Only one workflow runs at a time per issue number | P0 |
| 117 | Agent PR from non-agent author | Attacker | Manual PR with bot-like title | auto-merge-agent-prs.yml | Author check fails — only copilot-swe-agent[bot] and claude[bot] qualify | P0 |

---

## Feature Area 9: OAuth Security & Token Management

### Category 8: Security

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 118 | OAuth state JWT prevents CSRF | Attacker | No valid state token | Craft fake callback URL | State JWT verification fails, connection not created | P0 |
| 119 | GitHub grant revocation on disconnect (IKA-687) | Member | Connected | Disconnect | OAuth grant revoked on GitHub via API, consent screen forced on reconnect | P0 |
| 120 | Token not leaked across users (IKA-670) | Member | Rupesh connected | Sebastian queries /settings/github | Returns null — unique index on user_id, scoped queries | P0 |
| 121 | Webhook HMAC-SHA256 verification | Attacker | No webhook secret | POST /github/webhook with fake payload | Signature verification fails (constant-time compare), 401 | P0 |
| 122 | OAuth popup opened from user gesture (popup blocker bypass) | System | Popup blocker active | Click Connect | window.open called synchronously in click handler, survives blockers | P0 |
| 123 | Stale OAuth token used for API calls | Member | Token expired 1hr ago | Fetch available repos | GitHub returns 401, backend returns 502, UI shows reconnect prompt | P1 |
| 124 | SQL injection via repo_full_name | Attacker | Connected | Link repo with malicious name | Parameterized queries prevent injection | P0 |
| 125 | XSS via GitHub username display | Attacker | Connected with crafted username | View settings | React escapes HTML in text content, no XSS | P0 |
| 126 | Rate limiting on connection endpoints | Attacker | Valid token | Spam POST /settings/github | Rate limiter middleware prevents abuse | P1 |

---

## Feature Area 10: GitLab Mirror (Same Patterns)

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 127 | Connect GitLab account | Member | No GitLab connection | Click Connect GitLab > Authorize | Connection created with gitlab_username | P0 |
| 128 | Connect GitLab to specific project | Member | User has GitLab connection | Connect in project settings | Project-GitLab junction row created | P0 |
| 129 | Disconnect GitLab from project | Member | Project linked to GitLab | Disconnect | Junction row deleted, user token preserved | P0 |
| 130 | Link GitLab repository to workspace | Member | Connected | Select from available GitLab projects | Repository linked | P0 |
| 131 | Self-hosted GitLab instance | Member | Company uses self-hosted GitLab | Connect with custom gitlab_url | Connection uses custom URL instead of gitlab.com | P1 |

---

## Feature Area 11: MCP Integration with GitHub

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 132 | list_repos via MCP | External Agent | MCP connected with vk_ key | Call list_repos tool | Returns all repos accessible to the user | P0 |
| 133 | start_workspace_session with repo context | External Agent | Repo linked to project | Call start_workspace_session | Session includes repo context for agent work | P1 |

---

## Feature Area 12: Project-Level Connection Scoping (IKA-690)

### Category 1: Happy Path

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 134 | First-time connect: OAuth + auto-link | Member | No user connection, no project link | Click Connect in Project A | OAuth flow > token created > project_github_connections row inserted | P0 |
| 135 | Already connected: just link to project | Member | User has OAuth token, project not linked | Click Connect in Project B | POST /settings/projects/{B}/github > junction row created, no OAuth needed | P0 |
| 136 | Disconnect from one project, others unaffected | Member | Linked to Project A, B, C | Disconnect from Project A | Only Project A's junction row deleted, B and C still show Connected | P0 |
| 137 | User deletes global connection: all project links cascade | Member | Linked to 3 projects | DELETE /settings/github (user-level) | github_connections row deleted, CASCADE removes all project_github_connections rows | P0 |

### Category 7: Data Integrity

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 138 | Delete project: junction row cleaned up | Admin | Project has GitHub link | Delete project | project_github_connections row CASCADE deleted | P0 |
| 139 | ON CONFLICT: re-linking same project | Member | Already linked | POST link again | UPSERT: connection_id updated, linked_at refreshed, no duplicate | P1 |
| 140 | Unique constraint: one connection per project | System | Project A linked to Rupesh's connection | Sebastian tries to link his connection to same Project A | UNIQUE(project_id) — overwrite or conflict depending on UPSERT behavior | P1 |

### Category 8: Security

| # | Scenario | Actor | Precondition | Action | Expected Result | Priority |
|---|----------|-------|-------------|--------|-----------------|----------|
| 141 | User can only read their own project connection | Member | Rupesh linked to Project A | Sebastian calls GET /settings/projects/{A}/github | Returns null (user_id filter in backend) | P0 |
| 142 | User can only unlink their own connection | Member | Rupesh linked to Project A | Sebastian calls DELETE /settings/projects/{A}/github | 403 Forbidden "not your connection" | P0 |

---

## Implementation Checklist

- [ ] All P0 scenarios implemented
- [ ] All P0 scenarios tested (manual or automated)
- [ ] All P1 scenarios implemented
- [ ] Edge cases have proper error messages
- [ ] Destructive actions have confirmation dialogs
- [ ] Empty states have helpful UI
- [ ] Security scenarios verified with cross-user testing
- [ ] CASCADE deletes verified for all foreign key relationships
- [ ] OAuth grant revocation tested on disconnect
- [ ] Bot-loop prevention verified in GitHub workflows
