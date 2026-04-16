# OpZero — Organization Guide

This is the OpZero-sh GitHub org monorepo workspace. Each subdirectory
is an independent repo with its own CLAUDE.md. This file provides the
cross-repo context that individual project guides assume you already know.

## Product map

| Product | Directory | What it does | Runtime |
|---------|-----------|--------------|---------|
| **CodeZero** | `CodeZero/` | Web UI for Claude Code sessions (phone/browser) | Bun + React 19 |
| **CodeZ** | `CodeZ/` | Public fork of CodeZero (no secrets) | Bun + React 19 |
| **CodeZ Hub** | `codez-hub/` | Cloudflare Worker + DO that federates CodeZ instances behind a single MCP endpoint | Cloudflare Workers |
| **MCPAuthKit** | `MCPAuthKit/` | OAuth 2.1 gateway for MCP servers (PKCE, dynamic client registration) | Cloudflare Workers + D1 |
| **OpZero.sh** | `OpZero.sh/` | AI-powered deployment platform with MCP integration | Next.js + Vercel |
| **OpZero CLI** | `cli/` | Terminal CLI for deploying websites | Bun monorepo |
| **Backend** | `backend/` | Shared `@opzero/db` schema package (Drizzle ORM) | Node/TypeScript |
| **Infra** | `Infra/` | Dev container, IaC (OpenTofu), workspace MCP server | Docker + OpenTofu |
| **Token 5-0** | `token-5-0/` | Token budget manager MCP server for Claude Code | Node/TypeScript |
| **UAT** | `uat/` | AI-native test engine (browser, API, MCP) | Bun |
| **Skills** | `skills/` | Agent skills for deployment workflows | Markdown |
| **MCP AuthKit Vercel** | `mcp-authkit-vercel/` | Vercel Edge variant of MCPAuthKit | Vercel + Turso |

## Architecture

```
  Claude.ai / Claude Code / Mobile
       |
       |  MCP (Streamable HTTP)
       v
  CodeZ Hub  (code.open0p.com)
  Cloudflare Worker + Durable Object
       |
       |  WebSocket (per machine)
       v
  CodeZero instances (N machines)
  Bun server :4097 + React SPA
       |
       |  stream-json duplex
       v
  Claude Code CLI
```

### Auth chain

All services authenticate through **MCPAuthKit** (`authkit.open0p.com`):

- **MCP clients** (Claude.ai, Claude Desktop): OAuth 2.1 PKCE → `mat_*` tokens
- **Machine agents** (CodeZero → Hub): OAuth 2.1 PKCE → `mat_*` tokens with `agent:ws` scope
- **CodeZero web UI**: OAuth redirect to AuthKit → session cookie wrapping `mat_*` token
- **Token validation**: SHA-256 hash lookup in D1 `access_tokens` table (zero network calls)

Token prefixes: `mat_` (access, 1hr), `mrt_` (refresh, 30d), `code_` (auth code, 10min), `sak_` (server API key)

### Key domains

| Domain | Service | Platform |
|--------|---------|----------|
| `code.open0p.com` | CodeZ Hub MCP + WS | Cloudflare Workers |
| `authkit.open0p.com` | MCPAuthKit OAuth | Cloudflare Workers |
| `opzero.sh` / `opzero.io` | OpZero.sh platform | Vercel |

Note: `*.opzero.sh` subdomains do NOT have SSL configured on Cloudflare.
Use `*.open0p.com` for all Cloudflare Worker endpoints.

### Databases

| Database | Used by | Type |
|----------|---------|------|
| MCPAuthKit D1 (`d8c71aec-...`) | Hub (cross-binding), MCPAuthKit | Cloudflare D1 (SQLite) |
| CodeZ Hub D1 (`2adfde13-...`) | Hub | Cloudflare D1 (SQLite) |
| Neon Postgres | MCPAuthKit (user auth), OpZero.sh | Neon serverless |

### Cross-repo dependencies

- **CodeZero** imports `codez-hub/client/agent.ts` (HubMachineAgent) and `codez-hub/src/protocol/types.ts` via relative paths (`../../codez-hub/...`)
- **CodeZ** imports the same — both are siblings of `codez-hub/` in this workspace
- **CodeZ Hub** cross-binds MCPAuthKit's D1 for token validation (`AUTH_DB`)
- **MCPAuthKit** authenticates against Neon `authkit_users` table (not D1 `users`)

## Conventions

- **Language**: TypeScript (strict). No `any` unless deliberate cast.
- **Runtime**: Bun for CodeZ/CodeZero, Cloudflare Workers for Hub/AuthKit, Node 24+ for backend.
- **Package managers**: Bun (CodeZ, CLI), pnpm (Infra, backend), npm (MCPAuthKit).
- **Style**: No emojis in source, commits, or docs. Comments explain WHY.
- **Commits**: Conventional commits (`feat:`, `fix:`, `chore:`). Title < 72 chars.
  Always include: `Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>`
- **Git**: Never force-push main. Never rewrite published history. Never push without explicit user approval.

## Things NOT to touch without checking first

- **SHA-256 hash function** in `codez-hub/src/auth/tokens.ts` — must be byte-identical to MCPAuthKit
- **D1 cross-binding IDs** in `codez-hub/wrangler.jsonc` — points to production MCPAuthKit D1
- **DO naming** (`idFromName(userId)`) — changing this orphans all existing Durable Objects
- **`*.opzero.sh` SSL** — broken, use `*.open0p.com` for all Worker endpoints

## CodeZero ↔ CodeZ sync

CodeZero is the primary development repo. CodeZ is the public fork.
Changes go into CodeZero first, then are ported to CodeZ.
CodeZ must remain free of secrets, API keys, and internal-only config.
They share the same codebase but CodeZ is currently ~30 commits behind.
