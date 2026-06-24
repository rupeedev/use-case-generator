# Use Case Generator Plugin

Analyzes feature prompts and generates comprehensive use case scenarios before coding starts.

## Usage

```
/use-case-generator:generate-use-cases <feature description>
```

### Examples

```
/use-case-generator:generate-use-cases GitHub OAuth connection and repo linking
/use-case-generator:generate-use-cases Team invitation system with role-based access
/use-case-generator:generate-use-cases API key management for MCP authentication
/use-case-generator:generate-use-cases Document sharing with view/edit permissions
```

## What It Generates

For any feature, the plugin identifies scenarios across 8 dimensions:

1. **Happy Path** — Core CRUD operations
2. **Scope Variations** — User/Project/Team/Org level differences
3. **Permissions** — Owner, Admin, Member, Viewer, Guest, Unauthenticated
4. **State Transitions** — Connect/Disconnect, Enable/Disable, Active/Expired
5. **Edge Cases** — Empty states, duplicates, race conditions, timeouts
6. **UI/UX States** — Loading, error, empty, success, confirmation dialogs
7. **Data Integrity** — Cascading deletes, orphaned references, cleanup
8. **Security** — Token expiry, injection, unauthorized access, CSRF

Each scenario includes: Actor, Precondition, Action, Expected Result, and Priority (P0/P1/P2).

## Install

```bash
# From Claude Code:
/plugin install use-case-generator

# Or load locally (from the cloned repo root):
claude --plugin-dir .
```
