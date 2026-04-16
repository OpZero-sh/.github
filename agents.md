# OpZero — Agent Guide

Instructions for AI agents working across the OpZero workspace.

## Workspace layout

This is NOT a monorepo with a root package.json. Each subdirectory is
an independent git repo. There is no shared build system or workspace
manager at the root level.

## Before making changes

1. Read the target project's `CLAUDE.md` first — it has project-specific
   constraints, gotchas, and conventions.
2. Read this file for cross-project context (auth chain, domains, DB IDs).
3. If touching auth or token validation, verify SHA-256 byte-compatibility
   between `codez-hub/src/auth/tokens.ts` and `MCPAuthKit/src/worker.js`.

## Common tasks

### Deploy CodeZ Hub
```bash
cd codez-hub && npx wrangler deploy
```

### Deploy MCPAuthKit
```bash
cd MCPAuthKit && npx wrangler deploy
```

### Start CodeZero (with Hub)
```bash
cd CodeZero && CODEZ_HUB_URL="wss://code.open0p.com/ws" bun run start
```
First run without tokens triggers browser OAuth login via MCPAuthKit.

### Start CodeZero (local only)
```bash
cd CodeZero && bun run start
```

### Typecheck
```bash
cd <project> && bun run typecheck   # or ./node_modules/.bin/tsc --noEmit
```

### Mint a test token (via D1)
Insert into MCPAuthKit D1 `access_tokens` table. Hash with SHA-256
base64url. See `codez-hub/src/auth/tokens.ts` for the exact encoding.

## Integration boundaries

When modifying the integration between CodeZero and CodeZ Hub:

- **Hub protocol types** live in `codez-hub/src/protocol/types.ts`
- **Client library** lives in `codez-hub/client/agent.ts` + `client/types.ts`
- **CodeZero integration** lives in `CodeZero/server/hub.ts` + `server/hub-auth.ts`
- **Command handler** in `hub.ts` maps Hub actions to CodeZero's HTTP API
- The Hub injects `slug` into command params via `resolveSessionSlug()`

Changes to the WS protocol or command actions require updates in:
1. `codez-hub/src/protocol/types.ts` (type definitions)
2. `codez-hub/src/protocol/validate.ts` (Zod schemas)
3. `codez-hub/src/mcp/tools.ts` (MCP tool definitions)
4. `codez-hub/src/hub-agent.ts` (DO handler)
5. `CodeZero/server/hub.ts` (command handler)

## Auth provider modes

CodeZero supports three auth providers (set `authProvider` in config.json):

| Mode | Config value | Login flow |
|------|-------------|------------|
| Cookie (default) | `"cookie"` | Username/password form → session cookie |
| Cloudflare Access | `"cf-access"` | Cf-Access-Jwt-Assertion header |
| MCPAuthKit OAuth | `"authkit"` | Auto-redirect to authkit.open0p.com → callback → session cookie |

## Ports

| Port | Service |
|------|---------|
| 4097 | CodeZero server (HTTP + SSE) |
| 4098 | CodeZero MCP server (remote tools) |
| 3100 | Infra workspace MCP server |
