# Samuel System — Runtime Reference
_Ground truth: Obsidian/Samuel/CURRENT_STATE.md + FORGE_FREEZE.md (2026-03-08)_

> Samuel is the *world* (the hive), not a single computer. Each machine is a city-node.

---

## Node Topology (Current)

| City | Machine | Tailscale IP | Status | Role |
|------|---------|-------------|--------|------|
| **forge** | MacBook M4 Pro Max | `100.97.220.115` | **PRIMARY** | All Samuel ops while Capital offline |
| **sanctuary** | MBP i9 | `100.100.202.16` | Online — backup | Always-on backup; needs small rebuild to be fully useful |
| **capital** (Core) | Mac mini (Ubuntu) | `100.112.13.70` | **OFFLINE** | Hardware failure — rebuild pending |
| Home Assistant | Mac mini bare metal | `100.99.5.79` | Working | HAOS, rebuilt bare metal 2026-03-08 |

---

## Services — Forge (PRIMARY, 100.97.220.115)

| Port | Service | Status | Launchd label |
|------|---------|--------|---------------|
| 5100 | Samuel MCP (`python -m samuel`) | ✅ active | `com.samuel.local.mcp` |
| 5101 | Bridge | ✅ active | `com.samuel.local.bridge` |
| 5120 | Dev Hive coordinator | ❌ **RETIRED / DISABLED** | `com.samuel.local.devhive` |
| 5130 | LiteLLM proxy | ✅ active | `com.samuel.local.litellm` |
| 1234 | LM Studio (sole inference runtime) | ✅ active | LaunchAgent |
| 6333 | Qdrant (Docker) | ⚠️ stopped/retired | Docker |
| 18789 | OpenClaw gateway (ws) | ✅ active, always-on | LaunchAgent |

> **Dev Hive is RETIRED on Forge** — `com.samuel.local.devhive` disabled. Do NOT attempt to use Dev Hive coordinator on Forge. Dev Hive coordinator runs on Sanctuary only (backup at `100.100.202.16:5120`).

> **Ollama is uninstalled from Forge.** All inference runs through LM Studio only.

### LM Studio Models — Forge (CURRENT as of 2026-03-08)

| Model ID | Use | Notes |
|----------|-----|-------|
| `qwen3.5-35b-a3b` | **Primary agent model** | GGUF, MoE 3B active params, 64k ctx |
| `text-embedding-mxbai-embed-large-v1` | Embeddings | 1024d, cosine distance |

> Previous models (`devstral-small-2-2512`, `nomic-embed-text-v1.5`, `mistral-small-3.2`, `gemma-3-12b`, `gemma-3-4b`) are **all replaced**. Do not reference them.

### LiteLLM Aliases (:5130) — Active vs Stale

| Alias | Routes to | Status |
|-------|-----------|--------|
| `samuel-fast` | `qwen3.5-35b-a3b` — Forge LM Studio | ✅ active |
| `sanctuary-assistant` | `qwen3:14b` — Sanctuary Ollama | ✅ active |
| `capital-fast` | `qwen3:1.7b` — Capital Ollama | ❌ offline |
| `forge-coder`, `forge-assistant`, `forge-deep` | (stale — Ollama removed) | ⚠️ update needed |

---

## Services — Sanctuary (backup, 100.100.202.16)

| Port | Service | Status |
|------|---------|--------|
| 5120 | **Dev Hive backup coordinator** | ✅ active |
| 5130 | LiteLLM proxy | active |
| 5190 | Dev Hive Worker MCP | active |
| 11434 | Ollama — `qwen3:14b` always-on | ✅ active |

---

## Services — Capital (OFFLINE — 100.112.13.70)
Hardware failure. Brain (`:8020`) and Gate (`:8011`) not running — governance gate bypassed while Capital is down.
Will run when rebuilt: Dev Hive (:5120), Samuel MCP (:5100), LiteLLM (:5130), samuel_gate (:8011), samuel_brain (:8020), Ollama qwen3:1.7b.

---

## OpenClaw — Samuel's Agent Runtime

OpenClaw (`/opt/homebrew/bin/openclaw`, v2026.3.7) is Samuel's consciousness layer running on Forge.

| Field | Value |
|-------|-------|
| Workspace | `/Users/mikefrethy/Documents/GitHub/Obsidian` |
| Primary model | `qwen3.5-35b-a3b` via LM Studio `:1234` |
| Memory | **SQLite-vec** at `~/.openclaw/memory/main.sqlite` — 178 chunks, 1024d mxbai (active memory) |
| Gateway | `ws://127.0.0.1:18789` — LaunchAgent, always-on |
| Dashboard | `http://127.0.0.1:18789/` |

> **Qdrant is stopped and retired.** OpenClaw now uses sqlite-vec for semantic memory. `watch_vault.py` is also killed/inactive. The Qdrant Docker container still exists but is not running.

---

## MCP Servers (Claude Desktop / Claude Code)

| Server | Transport | Endpoint | Tools |
|--------|-----------|----------|-------|
| `samuel` | HTTP streamable | `http://127.0.0.1:5100/mcp` | read_config, get_entity_state, health, dev hive ops, audit |
| `homeassistant` | SSE via mcp-proxy | `http://100.99.5.79:8123/mcp_server/sse` | HassTurnOn/Off, HassLightSet, media, climate, lists |
| `homeassistant-ha` | stdio — node | `~/homeassistant-mcp/dist/src/index.js` | get_state, list_entities, list_areas, get_area_entities, call_service, get_error_log, list_packages, read_package, search_config |
| `rock-rms` | stdio — npx | mcp-remote → `rock-mcp-server.oneandall.workers.dev/mcp` | RockRMS people, events, giving |

`homeassistant-ha` env: `HASS_HOST=http://100.99.5.79:8123`, `REPO_PATH=/Users/mikefrethy/Documents/GitHub/ha-config`

---

## Key URLs

| URL | What |
|-----|------|
| `http://127.0.0.1:5100/mcp` | Samuel MCP — primary interface |
| `http://127.0.0.1:5101/health` | Bridge health |
| `http://127.0.0.1:5130` | LiteLLM proxy |
| `http://127.0.0.1:18789/` | OpenClaw dashboard |
| `http://100.100.202.16:5120/health` | Sanctuary Dev Hive backup health |
| `http://100.100.202.16:5190/mcp` | Sanctuary worker MCP |
| `http://10.20.99.2:8123` | Home Assistant (LAN) |
| `http://100.99.5.79:8123` | Home Assistant (Tailscale) |

---

## Three-Repo Architecture (Samuel layer only)

| Repo | Local path | Owns |
|------|------------|------|
| `samuel-system` | `/Users/mikefrethy/Documents/GitHub/samuel-system` | Dev Hive, MCP server, bridge, LiteLLM, worker engine, governance |
| `ha-config` | `/Users/mikefrethy/Documents/GitHub/ha-config` | HA YAML packages, Lovelace — deployed via `deploy_ha_config.sh` |
| `Obsidian` | `/Users/mikefrethy/Documents/GitHub/Obsidian` | Samuel's memory, identity, sessions, decisions; OpenClaw workspace |

Full 5-repo map: `mfrethy-oneandall/memory/repos.md`

Canonical docs entry points:
- `samuel-system/specs/HIVE_CHEATSHEET.md` (may have stale model/port info — defer to Obsidian)
- `samuel-system/specs/HIVE_REPOS.md`
- `Obsidian/Samuel/CURRENT_STATE.md` ← **most authoritative**
- `Obsidian/Samuel/FORGE_FREEZE.md` ← exact Forge baseline

---

## LLM Routing (while Capital offline)

| Role | Node | Model |
|------|------|-------|
| Agent / reasoning / code | Forge LM Studio | `qwen3.5-35b-a3b` |
| Embeddings | Forge LM Studio | `mxbai-embed-large-v1` |
| Always-on backup | Sanctuary Ollama | `qwen3:14b` |
| Cloud escalation | Claude / Gemini | Extreme complexity only — **never PII / church data** |

---

## Operating Rules

1. **Dev Hive coordinator = Sanctuary (:5120)** while Capital offline. Forge's devhive is disabled.
2. **Primary MCP interface = Samuel at :5100** — all hive ops go through this.
3. **Local model first** — Forge → Sanctuary → Capital. Cloud only when local confidence fails.
4. **Capital protection** — when rebuilt: coordinator-only, `qwen3:1.7b`, low memory pressure.
5. **PII safety** — RockRMS and HA data never sent to cloud. Use stewardship gates.
6. **Tailscale discipline** — use Tailscale IPs for all node-to-node ops.

---

## Capital Rebuild Checklist

1. Install LM Studio → load `qwen3.5-35b-a3b` + `mxbai-embed-large-v1`
2. Install Docker → run Qdrant container
3. Clone Obsidian vault → reindex
4. Clone samuel-system → install .venv → configure .env
5. Install samuel systemd services (dev-hive, mcp, bridge, litellm, brain, gate)
6. Update routing: `SAMUEL_ASSISTANT_ROUTE_ORDER=capital,forge,sanctuary`
7. Update Claude Desktop samuel → `http://100.112.13.70:5100/mcp`
