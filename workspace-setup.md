# Workspace setup

How to lay out the OpZero repos locally so an agent landing in the workspace can find cross-repo context fast.

## Clone the org

```bash
mkdir opzero-sh && cd opzero-sh

# Org profile first — holds the cross-repo CLAUDE.md and agents.md.
# Rename on clone so it sorts above project directories and isn't hidden.
gh repo clone opzero-sh/.github _org-profile

# Then every project repo:
for repo in \
  backend CodeZ codez-container codez-hub CodeZero Infra \
  mcp-authkit-vercel MCPAuthKit OpZ_cli OpZero.sh skillz token-5-0 uat
do
  gh repo clone "opzero-sh/$repo"
done
```

Result:

```
opzero-sh/
├── _org-profile/        ← this repo (org-level CLAUDE.md + agents.md)
├── backend/
├── CodeZ/
├── codez-container/
├── codez-hub/
├── CodeZero/
├── Infra/
├── mcp-authkit-vercel/
├── MCPAuthKit/
├── OpZ_cli/
├── OpZero.sh/
├── skillz/
├── token-5-0/
└── uat/
```

## Recommended reading order for agents

1. **`_org-profile/CLAUDE.md`** — product map, architecture, auth chain, domains, DB IDs, cross-repo deps, conventions, "do not touch" list.
2. **`_org-profile/agents.md`** — common task recipes (deploy, typecheck, mint tokens) and integration boundaries between CodeZero ↔ CodeZ Hub.
3. **`<project>/CLAUDE.md`** — project-specific constraints, read before editing any repo.

## Optional: workspace-root CLAUDE.md

If you run agents from the `opzero-sh/` directory (not from inside a project), drop a one-screen `CLAUDE.md` at the workspace root that just points at `_org-profile/CLAUDE.md`. The org profile's CLAUDE.md assumes the agent already knows to read it — a root pointer removes that assumption.

## Directory name caveats

The org profile docs use shorter names than the actual repo/directory names:

| Docs say | Actual directory |
|----------|------------------|
| `cli/` | `OpZ_cli/` |
| `skills/` | `skillz/` |

Translate these when following paths from `_org-profile/CLAUDE.md`.
