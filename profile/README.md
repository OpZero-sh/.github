# OpZ

**Production infrastructure for the agentic era.**

AI agents can write code. What they can't do — yet — is ship it, authenticate it, test it, observe it, or operate it without human glue. OpZ is a composable platform that fills every gap between "the agent wrote the code" and "it's running in production with guardrails."

10 repositories. Auth, deploy, test, observe, orchestrate — each one standalone, all of them wired together.

[![Website](https://img.shields.io/badge/opzero.sh-000?style=for-the-badge&logo=vercel&logoColor=white)](https://opzero.sh)
[![npm](https://img.shields.io/npm/v/opzero?style=for-the-badge&logo=npm&logoColor=white)](https://www.npmjs.com/package/opzero)
[![MCP](https://img.shields.io/npm/v/@opzero/mcp?style=for-the-badge&label=MCP&logo=npm&logoColor=white)](https://www.npmjs.com/package/@opzero/mcp)

### What OpZ solves

| Pain point | Without OpZ | With OpZ |
|-----------|-------------|----------|
| **Deployment** | Agent writes code, human copies it to a host | Agent deploys to Cloudflare/Vercel/Netlify via MCP or CLI |
| **Auth** | Every MCP server re-implements OAuth from scratch | MCPAuthKit: one Worker, full spec, five minutes |
| **Remote access** | Claude Code is locked to your terminal | CodeZ: phone, tablet, browser — agents control agents |
| **Testing** | Manual QA or fragile test scripts | UAT: 46 MCP tools, agents write and run acceptance tests |
| **Context limits** | Agent chokes on large outputs, loses history | token-5-0: vaults payloads, keeps summaries on the beat |
| **Operability** | Each tool is an island, nothing composes | Shared data layer, shared auth, shared MCP protocol end-to-end |

---

## Architecture

```
                    +---------------------------------------------+
                    |              AI  Agents                      |
                    |   Claude, Cursor, Copilot, Codex, ChatGPT   |
                    +----------------------+----------------------+
                                           |
              +----------------------------+----------------------------+
              |                            |                            |
        +-----------+               +------------+               +-----------+
        |  Skills   |               | MCP Server |               |    CLI    |
        | (SKILL.md)|               |  26 tools  |               |  opzero   |
        +-----------+               +-----+------+               +-----+-----+
              |                           |                            |
              +---------------------------+----------------------------+
                                          |
                  +-----------------------+-----------------------+
                  |                                               |
           +------+-------+                              +-------+--------+
           | MCPAuthKit   |                              |  @opzero/core  |
           | OAuth 2.1    |                              |  (API client)  |
           | (CF Worker   |                              +-------+--------+
           |  or Vercel)  |                                      |
           +------+-------+                              +-------+--------+
                  |                                      |   OpZero.sh    |
                  |    authenticates                     |   Platform     |
                  +------------------------------------->| Next.js + API  |
                                                        +-------+--------+
                                                                |
                            +-----------------------------------+------------------+
                            |                                   |                  |
                   +--------+-------+                  +--------+-------+  +-------+--------+
                   | Cloudflare     |                  |    Vercel      |  |    Netlify     |
                   |   Pages        |                  |                |  |                |
                   +--------+-------+                  +--------+-------+  +-------+--------+
                            |                                   |                  |
                            +-----------------------------------+------------------+
                                                        |
                                              +---------+---------+
                                              |  *.opzero.sh      |
                                              |  Live URLs        |
                                              +-------------------+


  +------------------+    +------------------+    +------------------+
  |      CodeZ       |    |       UAT        |    |    token-5-0     |
  | Remote Claude    |    | AI-native test   |    | Context vault    |
  | Code UI (17 MCP  |    | engine (46 MCP   |    | plugin (6 MCP    |
  | tools, mobile)   |    | tools, Playwright|    | tools, SQLite)   |
  +---------+--------+    +--------+---------+    +--------+---------+
            |                      |                       |
            +----------------------+-----------------------+
                                   |
                        +----------+----------+
                        |    @opzero/db       |
                        |  Drizzle + Neon     |
                        |  (shared schema)    |
                        +----------+----------+
                                   |
                        +----------+----------+
                        |      Infra          |
                        |  Control plane,     |
                        |  IaC, secrets,      |
                        |  agent orchestration|
                        +---------------------+
```

---

## Repositories

| Repo | What it does | |
|------|-------------|---|
| **[cli](https://github.com/opzero-sh/cli)** | `opzero` CLI + `@opzero/core` API client + `@opzero/mcp` server (26 tools). TypeScript, Bun. | [Details](#cli) |
| **[skills](https://github.com/opzero-sh/skills)** | Declarative agent skills for Claude Code, Cursor, Windsurf, and 20+ AI agents. | [Details](#skills) |
| **[MCPAuthKit](https://github.com/opzero-sh/MCPAuthKit)** | OAuth 2.1 for MCP servers. One Cloudflare Worker. Five minutes. | [Details](#mcpauthkit) |
| **[CodeZ](https://github.com/opzero-sh/CodeZ)** | Self-hosted web UI for Claude Code. Mobile-first, no API key required. | [Details](#codez) |
| **[uat](https://github.com/opzero-sh/uat)** | AI-native test engine: 46 MCP tools for browser, API, and MCP testing. | [Details](#uat) |
| **[token-5-0](https://github.com/opzero-sh/token-5-0)** | Context window police. Vaults oversized outputs, keeps compact summaries. | [Details](#token-5-0) |
| **OpZero.sh** `private` | Agentic deployment platform. Ship to Cloudflare, Vercel, or Netlify from any AI agent. | [Details](#opzerosh) |
| **backend** `private` | `@opzero/db` — shared Drizzle schema, multi-provider abstraction, migrations. | [Details](#backend) |
| **mcp-authkit-vercel** `private` | MCPAuthKit variant for Vercel Edge Functions + Turso. | [Details](#mcp-authkit-vercel) |
| **Infra** `private` | Workspace control plane, IaC (OpenTofu), dev containers, agent orchestration. | [Details](#infra) |

---

### cli

**[github.com/opzero-sh/cli](https://github.com/opzero-sh/cli)** — Deploy websites from your terminal. Powered by opzero.sh.

A Bun monorepo with three packages:

| Package | npm | What it does |
|---------|-----|-------------|
| `opzero` | [![npm](https://img.shields.io/npm/v/opzero)](https://www.npmjs.com/package/opzero) | CLI binary — deploy, login, projects, domains, rollback, templates, and more |
| `@opzero/core` | [![npm](https://img.shields.io/npm/v/@opzero/core)](https://www.npmjs.com/package/@opzero/core) | API client library — `OpZeroClient`, `AuthManager`, types |
| `@opzero/mcp` | [![npm](https://img.shields.io/npm/v/@opzero/mcp)](https://www.npmjs.com/package/@opzero/mcp) | 26-tool MCP server + focused 10-tool Claude Code plugin |

```bash
npx opzero deploy ./dist    # deploy from terminal
```

### skills

**[github.com/opzero-sh/skills](https://github.com/opzero-sh/skills)** — Official agent skills for 20+ AI agents.

Declarative SKILL.md playbooks that any compatible agent can follow. No SDK, no runtime — just markdown with YAML frontmatter.

| Skill | What it does |
|-------|-------------|
| `deploy-to-opzero` | Multi-step deploy with target selection (Cloudflare, Vercel, Netlify) |
| `opzero-quick-start` | Zero-to-live in 60 seconds |
| `opzero-mcp-setup` | MCP server configuration guide |
| `multi-cloud-deploy` | Platform comparison + decision guide |
| `static-site-best-practices` | Static site optimization |

```bash
npx skills add opzero-sh/skills
```

### MCPAuthKit

**[github.com/opzero-sh/MCPAuthKit](https://github.com/opzero-sh/MCPAuthKit)** — OAuth 2.1 for MCP servers. One Worker. Five minutes.

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

Running in production at [authkit.open0p.com](https://authkit.open0p.com).

### CodeZ

**[github.com/opzero-sh/CodeZ](https://github.com/opzero-sh/CodeZ)** — Self-hosted web UI for Claude Code.

Use Claude Code from your phone, tablet, or any browser — no API key required. Your Claude Max subscription handles auth via OAuth.

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

### uat

**[github.com/opzero-sh/uat](https://github.com/opzero-sh/uat)** — AI-native testing over Model Context Protocol.

46 MCP tools for browser automation, API testing, and MCP server verification. Built on Playwright + Bun. Agents write test plans in natural language; UAT translates them into browser sessions, HTTP calls, and assertions.

Used in production at OpZero for deployment verification.

### token-5-0

**[github.com/opzero-sh/token-5-0](https://github.com/opzero-sh/token-5-0)** — The token police.

Claude Code plugin that patrols your context window. When a tool call returns an oversized payload, token-5-0 books it into a local SQLite vault and keeps a compact summary on the beat. 6 tools: `vault_store`, `vault_retrieve`, `vault_search`, `vault_diff`, `vault_pack`, `vault_stats`.

### OpZero.sh

`private` — **[opzero.sh](https://opzero.sh)** — Your AI builds it. We put it on the internet.

The deployment platform at the center of the OpZ ecosystem. Agents, MCP clients, the CLI, and the web dashboard all converge here to ship static sites and React apps to Cloudflare Pages, Vercel, or Netlify. Deployed sites get live URLs at `*.opzero.sh`.

Next.js 16, React 19, Tailwind 4, Drizzle ORM, Neon PostgreSQL, Stripe billing, Vercel hosting. The CLI, MCP server, and skills all talk to its API via `@opzero/core`.

### backend

`private` — `@opzero/db`, the shared data layer.

Canonical Drizzle schema consumed by OpZero.sh and other services. Provider abstraction supports Neon, Postgres, Supabase, and SQLite. Includes migrations, seed scripts, and a standalone DB management MCP server.

### mcp-authkit-vercel

`private` — MCPAuthKit for the Vercel ecosystem.

Same OAuth 2.1 spec (RFC 9728, 8414, 7591, PKCE S256) built for Vercel Edge Functions + Turso (LibSQL). Drop-in alternative when your infra is on Vercel instead of Cloudflare.

### Infra

`private` — Workspace control plane.

Infrastructure root for the OpZero multirepo workspace. MCP server for agent coordination, dev containers (Node 24, Bun, pnpm), bootstrap/sync/teardown scripts, IaC modules (OpenTofu), SOPS+age encrypted secrets, Railway deployment.

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
