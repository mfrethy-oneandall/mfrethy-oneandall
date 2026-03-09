# Repos — Canonical Map

_Cross-repo reference for all agents. Updated: 2026-03-09._

> **Starting point for any agent attached to any repo.**
> Every repo's CLAUDE.md points here for the full picture.

---

## The 5 Repos

### 1. `mfrethy-oneandall` — Personal Operator Hub
**GitHub:** `mfrethy-oneandall/mfrethy-oneandall`
**Local (Forge):** `/Users/mikefrethy/Documents/GitHub/mfrethy-oneandall`
**Purpose:** Canonical operator context + cross-repo memory. Personal agent infrastructure.
**Owns:** `CLAUDE.md` (operator hot-cache), `memory/` (full knowledge base), productivity plugin, personal skills.
**Agent entry point:** Read `CLAUDE.md` → `memory/context/` as needed.

---

### 2. `samuel-system` — Samuel Hive Coordinator
**GitHub:** `mfrethy-oneandall/samuel-system`
**Local (Forge):** `/Users/mikefrethy/Documents/GitHub/samuel-system`
**Local (Sanctuary):** `/home/mikefrethy/samuel-system`
**Purpose:** Samuel's hive coordination — MCP server, bridge, LiteLLM, worker engine, governance.
**Key services:** MCP `:5100`, Bridge `:5101`, LiteLLM `:5130` (Forge). Dev Hive `:5120` (Sanctuary only).
**Note:** Dev Hive is RETIRED on Forge (`com.samuel.local.devhive` disabled). Sanctuary runs it.
**Agent entry point:** `CLAUDE.md` → `specs/HIVE_CHEATSHEET.md`. For current state: `memory/context/samuel-system.md` here.

---

### 3. `ha-config` — Home Assistant Runtime Config
**GitHub:** `mfrethy-oneandall/ha-config`
**Local (Forge):** `/Users/mikefrethy/Documents/GitHub/ha-config`
**Purpose:** HA YAML packages, Lovelace dashboard. Deployed to bare-metal HAOS via `deploy_ha_config.sh` (rsync+SSH from Forge).
**Does NOT contain:** Samuel intelligence, samuel-system services, or external HTTP calls inside packages. HA is an appliance.
**Agent entry point:** `docs/current_state.md` → `docs/ARCHITECTURE.md`. For reference: `memory/context/ha-config.md` here.

---

### 4. `Obsidian` — Samuel's Consciousness Layer
**GitHub:** `mfrethy-oneandall/Obsidian` *(private vault, synced via git)*
**Local (Forge):** `/Users/mikefrethy/Documents/GitHub/Obsidian`
**Purpose:** Samuel's persistent memory, identity, session logs, decisions. OpenClaw workspace.
**Owns:** `Samuel/CURRENT_STATE.md` (most authoritative node state), `Samuel/FORGE_FREEZE.md` (exact Forge baseline), sqlite-vec semantic memory (178 chunks), session logs.
**Note:** This is the *most authoritative* source for Samuel's current state. Prefer it over samuel-system docs when they conflict.
**Agent entry point:** `Samuel/CURRENT_STATE.md` → `Samuel/MEMORY_STACK_FREEZE.md`

---

### 5. `RockProduction` — ONE&ALL Rock RMS Production App
**GitHub:** `ONE-ALL-Church/RockProduction` (org repo — mfrethy-oneandall has access)
**Local (Forge):** `/Users/mikefrethy/Documents/GitHub/RockProduction`
**Purpose:** ONE&ALL's Rock RMS production web application (App_Code, Blocks, Plugins, Themes).
**Active committer:** Brian Davis (IT+Digital Director) — do NOT commit personal agent files here.
**IMPORTANT:** Personal agent files (CLAUDE.md, TASKS.md, memory/, productivity.plugin) are gitignored — kept locally only.
**Church context:** Agent memory files exist locally (untracked) for church IT context. See `.mcp.json` for RockRMS MCP connector.
**Agent entry point:** `CLAUDE.md` → `docs/agents/start-here.md` for Rock context.

---

## Sanctuary — Planned Rebuild Role

Sanctuary (MBP i9, `100.100.202.16`) is the always-on church-network node.
See: [memory/context/sanctuary-rebuild.md](memory/context/sanctuary-rebuild.md)

**Current:** Runs Dev Hive backup `:5120`, Ollama `qwen3:14b` `:11434`, Worker MCP `:5190`.
**Planned:** Strip old LiteLLM stack. Add church LAN function layers: Fortinet (all locations), UniFi/Ruckus, NanoMDM, O&A printers, RockRMS local health monitor.
**Physical advantage:** Sits on church office LAN → direct access to church appliances without Tailscale hop.

---

## Cross-Repo Awareness Rule

Every repo's `CLAUDE.md` should include the pointer:
```
→ Cross-repo map: mfrethy-oneandall/memory/repos.md
   (github.com/mfrethy-oneandall/mfrethy-oneandall)
```
This ensures any agent attached to any single repo can navigate to the full picture.

---

## Keeping This Current

When the system changes (new node, retired service, new repo, role change):
1. Update the relevant repo's own docs
2. Update this file (`mfrethy-oneandall/memory/repos.md`) ← **canonical**
3. Update `samuel-system/specs/HIVE_REPOS.md` ← mirrors this for Samuel context
4. Commit each repo separately
