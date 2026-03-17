# Samuel System — Runtime Reference
_Machine-readable ground truth. No historical content. Updated: 2026-03-17._

---

## 1. Node Topology

| Node | Machine | Tailscale IP | Status | Role |
|------|---------|-------------|--------|------|
| **forge** | MacBook M4 Pro Max 64GB, macOS | `100.97.220.115` | **PRIMARY** | All Samuel ops, LM Studio, OpenClaw |
| **sanctuary** | MBP i9 32GB, Ubuntu 24.04 | `100.100.202.16` | **BACKUP** | Full mirror of Forge services |
| **capital** | MacBook Pro i9 64GB, Ubuntu 24.04 | `100.127.78.110` | **ONLINE** | Rebuilt — LM Studio + Samuel services up; no models yet |
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

## 3. LM Studio Model Stack (Forge)

| Model ID | Format | Context | Role | Load Policy |
|----------|--------|---------|------|-------------|
| `qwen3.5-4b-mlx` | MLX 4-bit | 32768 | Fast baseline, agentic loops | Always resident |
| `qwen3.5-9b-mlx` | MLX 4-bit | 32768 | Mid-tier dev, committee | Resident |
| `qwen/qwen3.5-35b-a3b` | GGUF MoE | 65536 | Deep reasoning, codegen, novel proposals | JIT (TTL 600s) |
| `nomic-embed-text-v1.5` | GGUF | 2048 | 768d embeddings (RAG + OpenClaw sqlite-vec) | Resident |

Sanctuary runs reduced set: qwen3-8b (fast), qwen3-14b (JIT), nomic-embed-text-v1.5

---

## 4. LiteLLM Aliases (`:5130`)

| Alias | Routes To | Model |
|-------|-----------|-------|
| `samuel-fast` | Forge LM Studio → sanctuary-fast fallback | `qwen3.5-4b-mlx` |
| `samuel-assistant` | Forge LM Studio → sanctuary-fast fallback | `qwen3.5-4b-mlx` |
| `samuel-dev` | Forge LM Studio → sanctuary-fast fallback | `qwen3.5-9b-mlx` |
| `samuel-deep` | Forge LM Studio → forge-mid → sanctuary-fast | `qwen/qwen3.5-35b-a3b` |
| `samuel-coder` | Forge LM Studio → forge-mid → sanctuary-fast | `qwen/qwen3.5-35b-a3b` |
| `samuel-embed` | Forge LM Studio → sanctuary-embed fallback | `nomic-embed-text-v1.5` |

Note: `samuel-vision` is REMOVED — no VL models in the current stack.

---

## 5. OpenClaw

| Field | Value |
|-------|-------|
| Version | v2026.3.7 |
| Binary | `/opt/homebrew/bin/openclaw` |
| Gateway | `ws://127.0.0.1:18789` |
| Dashboard | `http://127.0.0.1:18789/` |
| Memory backend | sqlite-vec at `~/.openclaw/memory/main.sqlite` — 768d nomic embeddings (via sqlite-vec) |
| Workspace | `/Users/mikefrethy/Documents/GitHub/Obsidian` |
| Node | Forge only |

---

## 6. Sanctuary Services (BACKUP — 100.100.202.16)

| Port | Service | Notes |
|------|---------|-------|
| `:1234` | LM Studio | Reduced Qwen3 stack: qwen3-8b, qwen3-14b, nomic-embed-text-v1.5 |
| `:5100` | Samuel MCP | Mirror of Forge |
| `:5101` | Samuel Bridge | Mirror of Forge |
| `:5130` | LiteLLM proxy | Mirror of Forge |
| `:5122` | Open WebUI proxy | — |
| `:8080` | Open WebUI | — |

**Runtime path**: `/home/mikefrethy/samuel-worker`

**SSH to Forge**: `ssh forge` via `~/.ssh/id_ed25519_forge` (target: `mikefrethy@100.97.220.115`)

---

## 7. Capital (ONLINE — 100.127.78.110)

Status: **ONLINE** — rebuilt 2026-03-17. MBP i9 64GB, Ubuntu 24.04, LAN `10.20.99.107`, Tailscale `100.127.78.110` (hostname: `capital-host`).

**Note:** Old Tailscale device `capital` (`100.112.13.70`) is stale — remove from Tailscale admin panel.

| Port | Service | Status | Notes |
|------|---------|--------|-------|
| `:1234` | LM Studio | Running | No models yet — install TBD |
| `:5100` | Samuel MCP | Running | systemd user service |
| `:5101` | Samuel Bridge | Running | systemd user service |
| `:5130` | LiteLLM | Running | Proxies to Forge/Sanctuary until local models installed |

**Runtime path**: `~/samuel-worker`
**SSH**: `mikefrethy@10.20.99.107` (LAN) or `mikefrethy@100.127.78.110` (Tailscale)
**No models yet** — GGUF (x86, not MLX). Mike will select models after lms is confirmed working.
**RAG disabled** — no Obsidian vault on Capital.

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
| `qwen3-1.7b-mlx` | RETIRED — do not reference. Replaced by qwen3.5-4b-mlx. |
| `qwen/qwen3-vl-4b` | RETIRED — do not reference. Replaced by qwen3.5-4b-mlx. |
| `qwen/qwen3-vl-7b` | RETIRED — do not reference. Replaced by qwen3.5-9b-mlx. |
| `google/gemma-3-12b` | RETIRED — do not reference. Removed from LM Studio. |
| `text-embedding-mxbai-embed-large-v1` / `mxbai-embed-large-v1` (1024d) | RETIRED — do not reference. Replaced by nomic-embed-text-v1.5 (768d). |
| `samuel-vision` alias | RETIRED — do not reference. REMOVED — no VL models in stack. |
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
