# OpZero

**Your AI builds it. We put it on the internet.**

Build with Claude, ChatGPT, Cursor, Codex, or any AI agent. OpZero deploys your app or site to a real URL, on a real host, in seconds.

[![Website](https://img.shields.io/badge/opzero.sh-000?style=for-the-badge&logo=vercel&logoColor=white)](https://opzero.sh)
[![npm](https://img.shields.io/npm/v/opzero?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/opzero)
[![MCP](https://img.shields.io/npm/v/@opzero/mcp?style=for-the-badge&label=MCP&logo=npm&logoColor=white)](https://www.npmjs.com/package/@opzero/mcp)

---

## Architecture

```
                          +-----------------+
                          |   AI  Agents    |
                          | Claude, Cursor, |
                          | Copilot, Codex  |
                          +--------+--------+
                                   |
                    +--------------+--------------+
                    |              |              |
               +---------+  +---------+  +------------+
               |  Skills |  |   MCP   |  |    CLI     |
               | (SKILL  |  | Server  |  |  opzero    |
               |   .md)  |  | 26 tools|  |  deploy    |
               +---------+  +----+----+  +-----+------+
                    |             |              |
                    +-------------+--------------+
                                  |
                          +-------+-------+
                          |   @opzero/    |
                          |     core      |
                          | (API client)  |
                          +-------+-------+
                                  |
                          +-------+-------+
                          |   OpZero.sh   |
                          |   Platform    |
                          | Next.js + API |
                          +-------+-------+
                                  |
              +-------------------+-------------------+
              |                   |                   |
     +--------+-------+  +-------+--------+  +-------+--------+
     | Cloudflare     |  |    Vercel      |  |    Netlify     |
     |   Pages        |  |                |  |                |
     +----------------+  +----------------+  +----------------+
              |                   |                   |
              +-------------------+-------------------+
                                  |
                        +---------+---------+
                        |  *.opzero.sh      |
                        |  Live URLs        |
                        +-------------------+
```

### Auth Layer

```
+-----------------+       +-----------------------+
|   MCPAuthKit    |       |  mcp-authkit-vercel   |
| CF Workers + D1 |       | Vercel Edge + Turso   |
|   OAuth 2.1     |       |     OAuth 2.1         |
| RFC 9728/8414   |       |   RFC 9728/8414       |
+-----------------+       +-----------------------+
        |                           |
        +----------+  +------------+
                   |  |
            +------+--+------+
            |  MCP Clients   |
            | (Claude, etc.) |
            +-----------------+
```

### Data & Infrastructure

```
+-------------+     +-------------+     +-----------+
|  @opzero/db |---->|    Neon      |     |   Infra   |
|   Drizzle   |     |  PostgreSQL  |     | Workspace |
|   Schema    |     | (Serverless) |     |  Control  |
+-------------+     +-------------+     |   Plane   |
                                        +-----------+
```

---

## Repositories

### Deploy & Platform

| Repo | Description |
|------|-------------|
| **[cli](https://github.com/opzero-sh/cli)** | Monorepo: `opzero` CLI, `@opzero/core` API client, `@opzero/mcp` server (26 tools + Claude Code plugin). TypeScript, Bun. |
| **[skills](https://github.com/opzero-sh/skills)** | Declarative agent skills for Claude Code, Cursor, Windsurf, and 20+ AI agents. Deploy to any cloud with one command. |
| **OpZero.sh** `private` | The platform. Next.js 16, React 19, Tailwind 4, Drizzle ORM, Neon PostgreSQL, Stripe billing, Vercel hosting. |

### Auth — [MCPAuthKit](https://github.com/opzero-sh/MCPAuthKit)

Every MCP server builder hits the same wall: the OAuth spec is brutal. MCPAuthKit is the other side of that wall.

A single Cloudflare Worker (~600 lines) + D1 database that handles the **entire** MCP OAuth specification — discovery, registration, consent, tokens — so your MCP server doesn't have to. Your server's only job: validate the Bearer token.

```
Claude / ChatGPT              MCPAuthKit              Your MCP Server
     |                            |                        |
     |  POST /mcp (no token)      |                        |
     |-------------------------------------------------------->|
     |  401 + WWW-Authenticate    |                        |
     |<--------------------------------------------------------|
     |                            |                        |
     |  Discovery + Registration  |                        |
     |--------------------------->|                        |
     |  Consent + PKCE exchange   |                        |
     |--------------------------->|                        |
     |  { access_token }          |                        |
     |<---------------------------|                        |
     |                            |                        |
     |  POST /mcp (Bearer token)  |                        |
     |-------------------------------------------------------->|
     |  [tools response]          |                        |
     |<--------------------------------------------------------|
```

| Spec | Status |
|------|--------|
| RFC 9728 — Protected Resource Metadata | Complete |
| RFC 8414 — Authorization Server Metadata | Complete |
| RFC 7591 — Dynamic Client Registration | Complete |
| OAuth 2.1 — Authorization code + PKCE (S256) | Complete |
| Token refresh, revocation, consent UI, multi-tenant | Complete |

Running in production at [authkit.open0p.com](https://authkit.open0p.com). Also available as a Vercel Edge variant (`mcp-authkit-vercel` `private`).

### Agent Tooling — [CodeZ](https://github.com/opzero-sh/CodeZ)

Self-hosted web UI for Claude Code. Use it from your phone, tablet, or any browser — no API key required. Your Claude Max subscription handles auth via OAuth.

CodeZ spawns the `claude` CLI in `stream-json` duplex mode, keeps stdin open across turns so the context cache stays warm, and streams everything back to the browser via SSE. It also exposes a remote MCP server (17 tools) so **Claude chat can orchestrate Claude Code sessions** — your AI talks to your AI.

```
  Phone / Browser                CodeZ (Bun server)              Claude Code CLI
       |                              |                               |
       |  HTTPS (Cloudflare Tunnel)   |                               |
       |----------------------------->|                               |
       |                              |  spawn claude --stream-json   |
       |                              |------------------------------>|
       |                              |  stdout events (live stream)  |
       |       SSE (real-time)        |<------------------------------|
       |<-----------------------------|                               |
       |                              |                               |
       |  Send prompt                 |  stdin (context cache warm)   |
       |----------------------------->|------------------------------>|
       |                              |                               |


  Claude Chat (iOS / Desktop / Web)
       |
       |  MCP Streamable HTTP + MCPAuthKit OAuth
       |
       v
  CodeZ MCP Server (17 tools)
       |
       +-- create_session      — spawn sessions in any project
       +-- send_prompt         — send prompts to running sessions
       +-- poll_events         — watch real-time streaming output
       +-- respond_permission  — approve/deny tool permission requests
       +-- get_observability   — cost and token tracking
       +-- ...12 more          — abort, fork, dispose, search, state
```

Mobile-first (iPhone-optimized PWA), voice input, command palette, subagent team grid, cyberpunk UI. Runs on your machine, tunneled through Cloudflare — nothing phones home.

| Repo | Description |
|------|-------------|
| **[uat](https://github.com/opzero-sh/uat)** | AI-native test engine: 46 MCP tools for browser automation, API testing, and MCP server verification. Playwright + Bun. |
| **[token-5-0](https://github.com/opzero-sh/token-5-0)** | The token police. Claude Code plugin that vaults oversized outputs to local SQLite and keeps compact summaries on the beat. |

### Infrastructure

| Repo | Description |
|------|-------------|
| **backend** `private` | `@opzero/db` — shared Drizzle schema, provider abstraction (Neon/Postgres/Supabase/SQLite), migrations, DB management MCP server. |
| **Infra** `private` | Workspace control plane, dev containers, IaC (OpenTofu), SOPS+age secrets, Railway deployment, agent team orchestration. |

---

## Get Started

### Install the CLI

```bash
curl -fsSL https://opzero.sh/install.sh | bash
```

### Or use via npx

```bash
npx opzero deploy
```

### Add MCP server to Claude Code

```bash
curl -fsSL https://opzero.sh/install-mcp.sh | bash
```

### Add agent skills

```bash
npx skills add opzero-sh/skills
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 16, React 19, Tailwind CSS 4 |
| Backend | Bun, TypeScript, Drizzle ORM |
| Database | Neon PostgreSQL (serverless) |
| Auth | OAuth 2.1 (MCPAuthKit) |
| Hosting | Vercel, Cloudflare Pages, Netlify |
| Agent Protocol | MCP (Model Context Protocol) |
| Testing | Playwright, `@opzero/uat` |
| IaC | OpenTofu, Railway |

---

## Links

- **Website:** [opzero.sh](https://opzero.sh)
- **CLI:** [npm/opzero](https://www.npmjs.com/package/opzero)
- **MCP Server:** [npm/@opzero/mcp](https://www.npmjs.com/package/@opzero/mcp)
- **Skills:** `npx skills add opzero-sh/skills`
- **AuthKit:** [authkit.open0p.com](https://authkit.open0p.com)
