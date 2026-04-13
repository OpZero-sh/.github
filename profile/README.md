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
| **Orchestration** | Claude Code runs in terminal, browser, mobile — but they're separate unobservable islands | CodeZ: unified surface + Claude chat as the orchestration and observability layer for Claude Code agents |
| **Testing** | Manual QA or fragile test scripts | UAT: 46 MCP tools, agents write and run acceptance tests |
| **Context limits** | Agent chokes on large outputs, loses history | token-5-0: vaults payloads, keeps summaries on the beat |
| **Operability** | Each tool is an island, nothing composes | Shared data layer, shared auth, shared MCP protocol end-to-end |

---

## Philosophy

**Solve each problem once, well, and level up.**

MCP auth was a mess — every server reimplemented OAuth from scratch. So we solved it once: MCPAuthKit. ~600 lines, full spec, deploy in five minutes. Then we moved on.

That solution became the auth layer for CodeZ, which solved the next problem: Claude Code sessions are islands. Now Claude chat orchestrates Claude Code agents through MCP, authenticated by the thing we already built.

Every repo in this org exists because we hit a wall, solved it properly, and used that solution as the foundation for the next layer. Nothing is half-finished. Nothing is a demo. Each piece runs in production, works standalone, and composes with everything else.

The payoff is compounding speed. CodeZ is the most sophisticated thing in this org — a self-hosted web UI with 17 MCP tools, real-time streaming, mobile PWA, agent orchestration — and it was the fastest to build. It was dogfooding itself from an iPhone's Safari within the first hour. That hour was the only stretch the machine that built it was physically touched. Within 24 hours, Claude Code agents were being orchestrated from Claude iOS chat. That's what happens when auth, deploy, testing, and infra are already solved layers you can stand on.

The pattern:
1. **Hit a real problem** — not a hypothetical one
2. **Solve it completely** — implement the full spec, not 80%
3. **Ship it as infrastructure** — standalone, reusable, no vendor lock-in
4. **Build the next layer on top** — each solution unlocks the one above it, and the next thing ships faster than the last

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
| **OpZero.sh** `private` | Agentic deployment platform. Ship to Cloudflare, Vercel, or Netlify from any AI agent. | [Details](#opzerosh) |
| **[MCPAuthKit](https://github.com/opzero-sh/MCPAuthKit)** | OAuth 2.1 for MCP servers. One Cloudflare Worker. Five minutes. | [Details](#mcpauthkit) |
| **[CodeZ](https://github.com/opzero-sh/CodeZ)** | Unified Claude Code surface. Claude chat orchestrates Claude Code agents via MCP. | [Details](#codez) |
| **CodeZ Hub** `coming soon` | Multi-machine Claude Code hub on Cloudflare Edge. Route sessions across devices and cloud containers. | [Details](#codez-hub) |
| **[OpZ_CLI](https://github.com/opzero-sh/OpZ_CLI)** | Terminal CLI + local MCP server for Claude Code. The local counterpart to the hosted platform. | [Details](#opz_cli) |
| **[skillZ](https://github.com/opzero-sh/skillZ)** | Declarative agent skills for Claude Code, Cursor, Windsurf, and 20+ AI agents. | [Details](#skillz) |
| **[uat](https://github.com/opzero-sh/uat)** | AI-native test engine: 46 MCP tools for browser, API, and MCP testing. | [Details](#uat) |
| **[token-5-0](https://github.com/opzero-sh/token-5-0)** | Context window police. Vaults oversized outputs, keeps compact summaries. | [Details](#token-5-0) |
| **backend** `private` | `@opzero/db` — shared Drizzle schema, multi-provider abstraction, migrations. | [Details](#backend) |
| **mcp-authkit-vercel** `private` | MCPAuthKit variant for Vercel Edge Functions + Turso. | [Details](#mcp-authkit-vercel) |
| **Infra** `private` | Workspace control plane, IaC (OpenTofu), dev containers, agent orchestration. | [Details](#infra) |

---

### OpZ_CLI

**[github.com/opzero-sh/OpZ_CLI](https://github.com/opzero-sh/OpZ_CLI)** — The local, hands on surface for OpZ.

OpZero.sh is the hosted platform. OpZ_CLI is the local counterpart — a terminal CLI with all the OpZero tools, plus a local MCP server designed to be used directly with Claude Code. Where the hosted MCP server (`@opzero/mcp` on the platform) serves remote clients, the CLI's MCP server runs on your machine alongside your agent, with no network round-trip and no auth overhead.

| Package | npm | What it does |
|---------|-----|-------------|
| `opzero` | [![npm](https://img.shields.io/npm/v/opzero)](https://www.npmjs.com/package/opzero) | CLI binary — deploy, login, projects, domains, rollback, templates, and more |
| `@opzero/core` | [![npm](https://img.shields.io/npm/v/@opzero/core)](https://www.npmjs.com/package/@opzero/core) | API client library — `OpZeroClient`, `AuthManager`, types |
| `@opzero/mcp` | [![npm](https://img.shields.io/npm/v/@opzero/mcp)](https://www.npmjs.com/package/@opzero/mcp) | 26-tool MCP server (full) + 10-tool Claude Code plugin (focused) |

```bash
npx opzero deploy ./dist             # deploy from terminal
curl -fsSL https://opzero.sh/install-mcp.sh | bash   # add to Claude Code
```

### skillZ

**[github.com/opzero-sh/skillZ](https://github.com/opzero-sh/skillZ)** — Official agent skills for 20+ AI agents.

Declarative SKILL.md playbooks that any compatible agent can follow. No SDK, no runtime — just markdown with YAML frontmatter.

| Skill | What it does |
|-------|-------------|
| `deploy-to-opzero` | Multi-step deploy with target selection (Cloudflare, Vercel, Netlify) |
| `opzero-quick-start` | Zero-to-live in 60 seconds |
| `opzero-mcp-setup` | MCP server configuration guide |
| `multi-cloud-deploy` | Platform comparison + decision guide |
| `static-site-best-practices` | Static site optimization |

```bash
npx skills add opzero-sh/skillZ
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

**[github.com/opzero-sh/CodeZ](https://github.com/opzero-sh/CodeZ)** — Claude chat as an orchestration layer for Claude Code agents.

Claude Code already runs in the terminal, in the browser, and on mobile. The problem isn't access — it's that these surfaces are islands. Sessions don't cross devices, you can't direct one agent from another, and there's no unified view of what your agents are doing across projects.

CodeZ solves this by exposing Claude Code sessions as an MCP server. Any MCP client — including Claude chat on iOS, desktop, or web — can connect and get full programmatic control: spawn sessions, send prompts, approve permissions, watch output, track costs. **Claude chat becomes the orchestration layer for Claude Code agents.**

```
  Claude Chat (iOS / Desktop / Web)          CodeZ              Claude Code Agents
       |                                       |                       |
       |  "Start a session in ~/api and        |                       |
       |   fix the auth bug"                   |                       |
       |  ──── MCP: create_session ──────────> |                       |
       |                                       | spawn claude          |
       |                                       |────────────────────> [agent 1]
       |  "Also refactor the tests in ~/web"   |                       |
       |  ──── MCP: create_session ──────────> |                       |
       |                                       | spawn claude          |
       |                                       |────────────────────> [agent 2]
       |                                       |                       |
       |  ──── MCP: poll_events ─────────────> |  stream from both     |
       |  <──── real-time output ──────────────|<──────────────────── [1] [2]
       |                                       |                       |
       |  "Agent 1 wants to edit server.ts,    |                       |
       |   approve it"                         |                       |
       |  ──── MCP: respond_permission ──────> |────────────────────> [agent 1]
       |                                       |                       |
       |  ──── MCP: get_observability ───────> |                       |
       |  <──── cost: $0.42 across 2 sessions ─|                       |
```

The web UI is a parallel surface — mobile-first PWA with voice input, subagent team grid, session search, and real-time streaming — but the MCP server is the primitive. Anything that speaks MCP can orchestrate Claude Code through CodeZ.

Self-hosted. Runs on your machine, tunneled through Cloudflare. No API key required — uses your Claude Max subscription via OAuth. Nothing phones home.

### CodeZ Hub

`coming soon` — Multi-machine Claude Code hub on Cloudflare Edge.

CodeZ runs on one machine. CodeZ Hub is the next layer — a Cloudflare Workers Agent that federates CodeZ instances across multiple machines and on-demand cloud containers behind a single MCP endpoint.

```
            MCP Clients
            ┌──────────┬──────────┬──────────┐
            │Claude.ai │Claude CLI│ Mobile   │
            └────┬─────┴────┬─────┴────┬─────┘
                 │  HTTP/SSE │          │
                 └───────────┼──────────┘
                             │
╔════════════════════════════╧═════════════════════════════╗
║  Cloudflare Edge                                        ║
║                                                         ║
║  ┌─────────────────────────────────────────────────┐    ║
║  │  Worker Entry (codez.open0p.com/mcp)            │    ║
║  └──────────────────────┬──────────────────────────┘    ║
║                         │                               ║
║  ┌──────────────────────▼──────────────────────────┐    ║
║  │  CodeZero Hub  (extends Agent)                  │    ║
║  │                                                 │    ║
║  │  ┌─────────────┐ ┌──────────┐ ┌─────────────┐  │    ║
║  │  │ MCP Server  │ │ SQLite   │ │ State Sync  │  │    ║
║  │  │ (native)    │ │ machines │ │ to clients  │  │    ║
║  │  │             │ │ sessions │ │             │  │    ║
║  │  │ @callable() │ │ routing  │ │             │  │    ║
║  │  └──────┬──────┘ └──────────┘ └─────────────┘  │    ║
║  │         │                                       │    ║
║  │  ┌──────▼──────────────────────────────────┐    │    ║
║  │  │ WebSocket Manager                       │    │    ║
║  │  │  resolve(slug) → machineId → connection │    │    ║
║  │  └──┬──────────────┬───────────────────┬───┘    │    ║
║  └─────┼──────────────┼───────────────────┼────────┘    ║
║        │              │                   │             ║
║  ┌─────▼─────┐  ┌─────▼─────┐  ┌─────────▼─────────┐  ║
║  │AgentWork- │  │AgentWork- │  │  Cloud Containers  │  ║
║  │flow       │  │flow       │  │                    │  ║
║  │           │  │           │  │ ┌────────────────┐ │  ║
║  │ prompt    │  │ prompt    │  │ │Node+Claude Code│ │  ║
║  │ ↓        │  │ ↓        │  │ │(on-demand)     │ │  ║
║  │ permission│  │ permission│  │ └────────────────┘ │  ║
║  │ gate      │  │ gate      │  │ ┌────────────────┐ │  ║
║  │ ↓        │  │ ↓        │  │ │Node+Claude Code│ │  ║
║  │ checkpoint│  │ checkpoint│  │ │(on-demand)     │ │  ║
║  └─────┬─────┘  └─────┬─────┘  └─────────────────┘  ║
║        │              │              ▲               ║
╚════════╪══════════════╪══════════════╪═══════════════╝
         │ WS           │ WS           │ fetch
         │ (outward)    │ (outward)    │
    ┌────▼────┐    ┌────▼────┐        Infra
    │ codez   │    │ codez   │        project
    │ agent   │    │ agent   │        provisions
    │         │    │         │        these
    │ Claude  │    │ Claude  │
    │ Code    │    │ Code    │
    │ procs   │    │ procs   │
    ├─────────┤    ├─────────┤
    │ MacBook │    │ Work PC │
    │ (home)  │    │ (office)│
    └─────────┘    └─────────┘
```

One MCP endpoint. Any client — Claude.ai, Claude CLI, mobile — connects to the hub. The hub routes sessions to the right machine via WebSocket, manages permission gates and checkpoints as Agent Workflows, and spins up cloud containers for on-demand compute when your local machines are asleep. Authenticated by MCPAuthKit. Built on Cloudflare Agents SDK.

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
npx skills add opzero-sh/skillZ
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
- **Skills:** `npx skills add opzero-sh/skillZ`
- **AuthKit:** [authkit.open0p.com](https://authkit.open0p.com)
