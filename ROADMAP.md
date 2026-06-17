# OpZ Roadmap

**North star:** one platform — **`code.opzero.sh`** — where a user logs in once, sees every machine they own, pair-programs with Claude (or any agent) across those machines, and ships the result to production. UI at `code.opzero.sh/`, hosted MCP connector at `code.opzero.sh/mcp`, a single OAuth layer (MCPAuthKit) behind both.

The pieces already exist as standalone layers (see the [org overview](./profile/README.md)). This roadmap is about **unifying them into one product surface** and then **opening it to agents beyond Claude Code**.

> Status legend: ✅ shipped · 🟡 in progress · ⚪ planned
> Last verified against the running system: **2026-06-17**

---

## Where we are today (verified)

- ✅ **Deploy MCP is the mature core.** `opzero.sh` ships to Cloudflare / Vercel / Netlify via 26 MCP tools or the CLI. This is production and the most battle-tested layer.
- ✅ **MCPAuthKit is the auth layer.** OAuth 2.1 for MCP on a single Cloudflare Worker (`authkit.open0p.com`), D1-backed access tokens, refresh-token rotation with replay-family revocation. CodeZ, the hub, and the deploy MCP all authenticate through it.
- ✅ **Shared hub-client package across CodeZero and CodeZ.** The hub client was extracted into `@opzero/codez-hub-client` (`file:` linked from CodeZero), so the source (CodeZero) and the distribution surface (CodeZ) share one transport/agent implementation.
- 🟡 **CodeZero → CodeZ packaging pipeline.** Intended roles: **CodeZero is the source** (private, where dev happens, what runs on machines); **CodeZ is where the app is packaged and distributed** (public release surface — containers, published builds, docs). They drifted once and were force-synced to the same tip as a one-time cleanup — *that sync is not the model*. Still needed: a defined release/packaging step that flows CodeZero → CodeZ so they stop drifting and never need a manual re-sync again.
- ✅ **Hub federation MVP is live.** `codez-hub` runs a per-user Durable Object on Cloudflare Edge; machines register over WebSocket, the MCP client lists/drives them, and a **machine picker** now lets a remote client create a session on a chosen machine. **Machine-wake** endpoint + MCP tool just landed.
- 🟡 **Custom-hostname cutover in flight.** `opzero-router` (catch-all dispatcher) + Cloudflare SaaS custom hostname work is migrating the hub from `code.open0p.com` → **`code.opzero.sh`**.
- 🟡 **MCPAuthKit rebrand** (`rebrand-authzero` branch) and device-code grant in progress.

The gap between "today" and the north star is **consolidation + a real hosted UI + multi-agent support** — the phases below.

---

## Phase 1 — One domain, one login 🟡

Collapse the three entrypoints (CodeZ UI, hub MCP, deploy MCP) onto a single authenticated origin under **`opzero.sh`** — without sacrificing the reliability that has kept the Cloudflare-hosted services up.

- 🟡 Finish SaaS custom-hostname routing so **`code.opzero.sh`** terminates at `opzero-router` → hub.
- ⚪ Serve the **CodeZ web UI at `code.opzero.sh/`** and the **hosted MCP connector at `code.opzero.sh/mcp`** from the same origin.
- ⚪ Make **MCPAuthKit the single OAuth** for both the UI session and the `/mcp` connector (one consent, one token family per user).
- ⚪ Retire `*.open0p.com` hostnames once the `opzero.sh` subdomains are authoritative; keep redirects.
- ⚪ Publish `code.opzero.sh/mcp` as a **claude.ai custom connector** (hosted MCP entrypoint) end-to-end.

**Done when:** a user adds `code.opzero.sh/mcp` as a connector on claude.ai, logs in via MCPAuthKit, and lands in their own hub DO with zero per-machine token wiring.

### Domain consolidation — hosting split (Vercel apex + Cloudflare subdomains)

The tension: **`opzero.sh` is on Vercel** (`ns1/ns2.vercel-dns.com`, apex + `www` → Vercel), while the reliable, always-on services (MCPAuthKit, codez-hub) are **Cloudflare Workers** under `*.open0p.com`. Cloudflare has been more reliable for these, so the goal is **"everything under `opzero.sh`," not "everything on Vercel."**

**Approach — do not migrate the Workers off Cloudflare.** Keep the marketing site/apex on Vercel and expose each Worker under an `opzero.sh` subdomain via **CNAME → Cloudflare for SaaS custom hostname**, which is exactly what `code.opzero.sh` already does (it CNAMEs to `code.open0p.com` and is served by `opzero-router`).

- 🟡 `code.opzero.sh` → already CNAME'd to the CF worker (custom hostname live).
- ⚪ `authkit.opzero.sh` → **repoint to the MCPAuthKit worker** via CNAME + CF custom hostname. *(Currently still resolves to Vercel IPs — it's a placeholder, not the worker.)*
- ⚪ Add the custom hostnames to Cloudflare for SaaS on the worker side and provision certs (watch the CAA gotcha already documented in the hub RUNBOOK).
- ⚪ Keep `opzero.sh` / `www.opzero.sh` on Vercel for the site; only service subdomains move to CF.
- ⚪ Decision to record: **subdomain split** (chosen) vs. path-based `opzero.sh/mcp` (would require Vercel→CF rewrites/proxy on the apex — extra hop, weaker reliability). Subdomains win on reliability and simplicity.

**Done when:** `authkit.opzero.sh` and `code.opzero.sh` both serve their Cloudflare Workers, the site stays on Vercel, and nothing depends on `open0p.com`.

---

## Phase 2 — See and drive every machine ⚪

The hosted UI becomes the operator console.

- ⚪ **Machine dashboard** at `code.opzero.sh`: connected machines (online/offline, specs, repos), live/idle session counts — the data the hub already exposes.
- ⚪ Per-machine **session list**: start new, resume idle, abort, fork; mirror read-only sessions owned elsewhere.
- ⚪ **Wake** sleeping machines from the UI (wire the new machine-wake tool into the front end).
- ⚪ **Services & health panel** in CodeZero (local WIP exists, stashed `2026-06-17`) — surface hub / MCP / deploy-service status to the operator. Re-land and merge upstream.
- ⚪ Robust **reconnect / token-refresh** for the machine agent so a machine never silently drops (root cause of the 2026-06 outage: expired token + revoked refresh family, no auto-recovery).

### Auth: replace minted machine tokens with MCPAuthKit user login

**Preference:** machines should authenticate by the **user logging in through MCPAuthKit (OAuth)**, not by hand-minted long-lived `mat_` tokens inserted into D1. The direct-mint shortcut is a reliability *stopgap*, not the model — it's opaque, unrevocable per-user, and bypasses consent.

- ⚪ Make the machine-agent **OAuth login the default and reliable path**: detect refresh failure / family revocation and **re-prompt login automatically** instead of silently dropping offline.
- ⚪ Ship the **device-code grant** (MCPAuthKit `migrations/002_device_codes.sql` is already in flight) so headless machines/containers can complete login without a local browser — removing the very reason direct-mint was used.
- ⚪ One MCPAuthKit consent issues the token family for **both** the UI session and the machine agent under the same `user_id`.
- ⚪ Migrate the current stopgap: replace the minted `opz-2.local` token (`CODEZ_HUB_TOKEN` in `.env`, expiry 2030) with an OAuth-issued session once the reliable login path lands; then retire direct-mint for user-owned machines.

**Done when:** connecting a new machine is "log in with MCPAuthKit" end-to-end (browser or device-code), with auto re-login on expiry — no manual token handling.

**Done when:** from claude.ai or `code.opzero.sh`, a user can see all their machines and start/observe/stop sessions on any of them.

---

## Phase 3 — Orchestrate, then deploy, in one flow ⚪

Fuse the operator surface with the mature deploy MCP so "build" and "ship" are one authenticated session.

- ⚪ Expose the **deploy MCP tools through the same `code.opzero.sh/mcp` connector** (or a federated namespace) so orchestration and deployment share one auth + one session.
- ⚪ Claude on claude.ai acts as **orchestrator**: pair-programs across machines, then calls deploy tools to ship to a chosen provider — no context switch, no second login.
- ⚪ Thread **provider/target selection** through the session (which machine built it → where it deploys).
- ⚪ Capture deploy results + live URLs back into the session timeline.

**Done when:** "build this on my laptop, then deploy it to Cloudflare" is a single claude.ai conversation against one connector.

---

## Phase 4 — Any agent, not just Claude Code ⚪

Generalize the session layer so the orchestrator can manage heterogeneous agents.

- ⚪ Define an **agent-session adapter interface** (spawn, prompt, stream events, permissions, abort) decoupled from Claude Code specifics.
- ⚪ Ship a **Codex session adapter** as the second implementation.
- ⚪ Let an orchestrator run a **Claude Code session and a Codex session side-by-side** on the same machine, visible together in the dashboard.
- ⚪ Normalize the **event/transcript schema** across agents so observability and token-5-0 vaulting work uniformly.
- ⚪ Extend `codez setup` to detect/configure multiple agent backends.

**Done when:** the machine dashboard shows mixed Claude + Codex sessions and the orchestrator drives both through the same MCP surface.

---

## Phase 5 — Operate it like a product ⚪

- ⚪ **Teams / multi-tenant:** more than one user per org, shared machines, scoped tokens.
- ⚪ **Observability & audit** across sessions (uat for verification, token-5-0 for context discipline, audit log of orchestrate + deploy actions).
- ⚪ **Billing / usage** metering on the hosted connector.
- ⚪ Harden MCPAuthKit (post-`authzero` rebrand): device-code grant, per-connector scopes, rotation tooling.

---

## Cross-cutting / per-repo workstreams

| Repo | Near-term focus |
|------|-----------------|
| **codez-hub** | `code.opzero.sh` custom-hostname cutover; agent reconnect/refresh hardening; multi-agent routing |
| **CodeZero (source)** | Re-land services/health panel; host UI at `code.opzero.sh/`; agent-session adapter interface (Phase 4) |
| **CodeZ (package/distribute)** | Formalize the CodeZero → CodeZ release/packaging pipeline so the distribution surface tracks source without manual git syncs |
| **MCPAuthKit** | Finish `authzero` rebrand + device-code grant; single-consent for UI + `/mcp` |
| **OpZero.sh (deploy MCP)** | Federate deploy tools into the unified connector (Phase 3) |
| **OpZ_cli / skillz** | `codez setup` multi-agent detection; deploy + orchestrate playbooks |
| **uat / token-5-0** | Plug into the unified session event schema for verification + context vaulting |

---

## Operational notes

- **Hub host:** currently `wss://code.open0p.com/ws`; target `wss://code.opzero.sh/ws`. Machine agents read `CODEZ_HUB_URL` + `CODEZ_HUB_TOKEN` (env preferred over the OAuth browser flow).
- **Machine identity:** the hub keys each connection to a per-user Durable Object via the `mat_` token's `user_id` in MCPAuthKit D1. The machine agent and the MCP client **must resolve to the same `user_id`** or the operator sees an empty hub.
- **Token longevity:** short-lived access tokens that fail to auto-refresh are the main reliability risk for always-on machine agents. The *interim* mitigation is a long-lived directly-minted `mat_` token (used to restore `opz-2.local` on 2026-06-17), but the **target model is MCPAuthKit user login with auto re-login** (see Phase 2 auth section) — direct-mint is being retired for user-owned machines.
- **Hosting:** site/apex on Vercel (`opzero.sh`); always-on services (MCPAuthKit, codez-hub) stay on Cloudflare Workers, exposed under `*.opzero.sh` via CNAME + CF custom hostnames. `open0p.com` is the legacy host being phased out.
