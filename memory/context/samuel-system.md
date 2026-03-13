# Samuel System — Runtime Reference
_Machine-readable ground truth. No historical content. Updated: 2026-03-12._

---

## 1. Node Topology

| Node | Machine | Tailscale IP | Status | Role |
|------|---------|-------------|--------|------|
| **forge** | MacBook M4 Pro Max 64GB, macOS | `100.97.220.115` | **PRIMARY** | All Samuel ops, LM Studio, OpenClaw |
| **sanctuary** | MBP i9 32GB, Ubuntu 24.04 | `100.104.133.84` | **BACKUP** | Full mirror of Forge services |
| **capital** | Mac mini 8GB, Ubuntu | `100.112.13.70` | **OFFLINE** | Hardware failure — rebuild pending |
| **home-assistant** | Mac mini bare metal, HAOS | `100.99.5.79` | Working | HAOS 2026.3.1, MCP `:8123` |

---

## 2. Forge Services (PRIMARY — 100.97.220.115)

| Port | Service | Launchd Plist | Purpose |
|------|---------|---------------|---------|
| `:1234` | LM Studio | (LM Studio app) | Sole inference runtime — 4-model Qwen3 stack |
| `:5100` | Samuel MCP | `com.samuel.local.mcp` | Primary MCP interface — all hive ops |
| `:5101` | Samuel Bridge | `com.samuel.local.bridge` | REST bridge layer |
| `:5130` | LiteLLM proxy | `com.samuel.local.litellm` | Multi-model router with aliases |
| `:8080` | Open WebUI | `com.samuel.local.openwebui` | Chat interface |
| `:18789` | OpenClaw gateway (ws) | `ai.openclaw.gateway.plist` | Agent runtime, always-on |

---

## 3. LM Studio Model Stack (Forge + Sanctuary — identical)

| Model ID | Format | Context | Role | Load Policy |
|----------|--------|---------|------|-------------|
| `qwen3-1.7b-mlx` | MLX 4bit | 4096 | Reflex/dispatcher, `/no_think`, no tool calls | Always pinned |
| `qwen/qwen3-vl-4b` | MLX | 8192 | Tool calls, HA commands, vision, agentic loops | Always pinned |
| `qwen/qwen3.5-35b-a3b` | GGUF MoE | 64k | Deep reasoning, codegen, novel proposals | JIT (~15min evict) |
| `text-embedding-mxbai-embed-large-v1` | GGUF | — | 1024d embeddings (OpenClaw sqlite-vec) | Resident |

---

## 4. LiteLLM Aliases (`:5130`)

| Alias | Routes To | Model |
|-------|-----------|-------|
| `samuel-fast` | Forge LM Studio | `qwen3-1.7b-mlx` |
| `samuel-assistant` | Forge LM Studio | `qwen/qwen3-vl-4b` |
| `samuel-deep` / `samuel-coder` | Forge LM Studio | `qwen/qwen3.5-35b-a3b` |

---

## 5. OpenClaw

| Field | Value |
|-------|-------|
| Version | v2026.3.7 |
| Binary | `/opt/homebrew/bin/openclaw` |
| Gateway | `ws://127.0.0.1:18789` |
| Dashboard | `http://127.0.0.1:18789/` |
| Memory backend | sqlite-vec at `~/.openclaw/memory/main.sqlite` — 1024d mxbai embeddings |
| Workspace | `/Users/mikefrethy/Documents/GitHub/Obsidian` |
| Node | Forge only |

---

## 6. Sanctuary Services (BACKUP — 100.104.133.84)

| Port | Service | Notes |
|------|---------|-------|
| `:1234` | LM Studio | Same 4-model Qwen3 stack as Forge |
| `:5100` | Samuel MCP | Mirror of Forge |
| `:5101` | Samuel Bridge | Mirror of Forge |
| `:5130` | LiteLLM proxy | Mirror of Forge |
| `:5122` | Open WebUI proxy | — |
| `:8080` | Open WebUI | — |

**Runtime path**: `/home/mikefrethy/samuel-worker`

**SSH to Forge**: `ssh forge` via `~/.ssh/id_ed25519_forge` (target: `mikefrethy@100.97.220.115`)

---

## 7. Capital (OFFLINE — 100.112.13.70)

Status: **OFFLINE** — hardware failure, rebuild pending.

When rebuilt: same LM Studio + model stack baseline as Forge. No Ollama. No Dev Hive.

---

## 8. HA Connection

Samuel connects to Home Assistant via **real-time WebSocket subscriber** (`samuel/ha_websocket.py`).

| Field | Value |
|-------|-------|
| Implementation | `samuel/ha_websocket.py` |
| Env var | `SAMUEL_HA_WS` (default: enabled) |
| Reconnect | Auto-reconnect with exponential backoff |
| Latency | <1s (real-time event stream) |
| Protocol | HA WebSocket API — subscribes to all state changes and events |

---

## 9. RETIRED — Do Not Reference

| Component | Status |
|-----------|--------|
| Ollama | RETIRED — do not reference. Not installed on any node. |
| Qdrant | RETIRED — do not reference. Replaced by OpenClaw sqlite-vec. |
| Dev Hive `:5120` | RETIRED — do not reference. Disabled on all nodes. |
| Worker MCP `:5190` | RETIRED — do not reference. No longer running anywhere. |
| Robot City `:5110` | RETIRED — do not reference. Visualizer deprecated. |
| `devstral-small-2-2512` | RETIRED — do not reference. Removed from LM Studio. |
| `nomic-embed-text-v1.5` | RETIRED — do not reference. Replaced by mxbai-embed-large-v1. |
| Shadow polling (HA) | RETIRED — do not reference. Replaced by WebSocket subscriber. |
| `watch_vault.py` | RETIRED — do not reference. Killed/inactive. |
| `index_vault.py` | RETIRED — do not reference. Killed/inactive. |

---

## 10. Repo Paths

| Repo | Forge Path | Purpose |
|------|-----------|---------|
| `samuel-system` | `~/Documents/GitHub/samuel-system` | Canonical source — MCP server, bridge, LiteLLM, governance |
| `ha-config` | `~/Documents/GitHub/ha-config` | HA YAML packages + Lovelace — deployed via `deploy_ha_config.sh` |
| `Obsidian` | `~/Documents/GitHub/Obsidian` | Samuel's memory, identity, sessions; OpenClaw workspace |

**Sanctuary runtime path**: `~/samuel-worker` (deployed clone of `samuel-system`)

---

## 11. Deploy

### Forge — Launchd (macOS)

```bash
launchctl unload ~/Library/LaunchAgents/com.samuel.local.mcp.plist
launchctl load  ~/Library/LaunchAgents/com.samuel.local.mcp.plist

launchctl unload ~/Library/LaunchAgents/com.samuel.local.bridge.plist
launchctl load  ~/Library/LaunchAgents/com.samuel.local.bridge.plist

launchctl unload ~/Library/LaunchAgents/com.samuel.local.litellm.plist
launchctl load  ~/Library/LaunchAgents/com.samuel.local.litellm.plist

launchctl unload ~/Library/LaunchAgents/ai.openclaw.gateway.plist
launchctl load  ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

### Sanctuary — Systemd (Ubuntu)

```bash
systemctl --user restart samuel-mcp.service
systemctl --user restart samuel-bridge.service
systemctl --user restart samuel-litellm.service
```
