# Samuel System — Runtime Reference
_Machine-readable ground truth. No historical content. Updated: 2026-03-17._

---

## 1. Node Topology

| Node | Machine | Tailscale IP | Status | Role |
|------|---------|-------------|--------|------|
| **capital** | MacBook Pro i9 64GB, Ubuntu 24.04 | `100.127.78.110` | **PRIMARY inference** | Home node — all `samuel-*` aliases route here first |
| **sanctuary** | MBP i9 32GB, Ubuntu 24.04 | `100.100.202.16` | **First fallback** | Work node — IT triage, always-on backup |
| **forge** | MacBook M4 Pro Max 64GB, macOS | `100.97.220.115` | **Last-resort / dev** | Dev station — travels with Mike |
| **home-assistant** | Mac mini bare metal, HAOS | `100.99.5.79` | Working | HAOS 2026.3.1, MCP `:8123` |

Node roles (env `SAMUEL_NODE_ROLE`): Capital=`home`, Sanctuary=`work`, Forge=`dev`

---

## 2. Capital Services (PRIMARY — 100.127.78.110)

| Port | Service | Notes |
|------|---------|-------|
| `:1234` | LM Studio | allura-8b + qwen3.5-4b + qwen3-14b (JIT) + nomic |
| `:5100` | Samuel MCP | systemd user service |
| `:5101` | Samuel Bridge | systemd user service |
| `:5130` | LiteLLM proxy | Tri-node routing — Capital primary |
| `:18789` | OpenClaw gateway | Moved from Forge with routing flip |

**Runtime path**: `~/samuel-worker`
**SSH**: `mikefrethy@100.127.78.110` (Tailscale)

---

## 3. LM Studio Model Stacks

### Capital (x86 GGUF — 64GB)

| Model ID | Context | Role | Load Policy |
|----------|---------|------|-------------|
| `allura-forge_llama-3.3-8b-instruct` | 16384 | Assistant — no thinking overhead | Always resident |
| `qwen/qwen3.5-4b` | 16384 | Fast baseline | Always resident |
| `qwen/qwen3-14b` | 16384 | Heavy reasoning (local GGUF) | JIT (TTL 600s) |
| `nomic-embed-text-v1.5` | 2048 | 768d embeddings (RAG) | Always resident |

### Sanctuary (x86 GGUF — 32GB)

| Model ID | Context | Role | Load Policy |
|----------|---------|------|-------------|
| `allura-forge_llama-3.3-8b-instruct` | 16384 | Assistant fallback | Always resident |
| `qwen/qwen3.5-4b` | 16384 | Fast fallback | Always resident |
| `nomic-embed-text-v1.5` | 2048 | 768d embeddings (RAG) | Always resident |

### Forge (MLX — 64GB)

| Model ID | Context | Role | Load Policy |
|----------|---------|------|-------------|
| `qwen3.5-4b-mlx` | 32768 | Fast baseline | Always resident |
| `qwen3.5-9b-mlx` | 32768 | Mid-tier dev | Resident |
| `qwen/qwen3.5-35b-a3b` | 65536 | Deep reasoning, codegen | JIT (TTL 600s) |
| `nomic-embed-text-v1.5` | 2048 | 768d embeddings | Resident |

---

## 4. LiteLLM Aliases (`:5130`) — v4 Tri-node

Capital primary → Sanctuary fallback → Forge last-resort

| Alias | Primary | Fallback 1 | Fallback 2 |
|-------|---------|------------|------------|
| `samuel-fast` | capital-fast (qwen3.5-4b) | sanctuary-fast | forge-fast |
| `samuel-assistant` | capital-assistant (allura-8b) | sanctuary-assistant | forge-fast |
| `samuel-dev` | capital-assistant (allura-8b) | sanctuary-assistant | forge-mid |
| `samuel-deep` | capital-backup (qwen3-14b) | forge-heavy | forge-mid |
| `samuel-coder` | capital-backup (qwen3-14b) | forge-heavy | forge-mid |
| `samuel-embed` | capital-embed (nomic 768d) | sanctuary-embed | forge-embed |

Note: `samuel-vision` is REMOVED — no VL models in the current stack.

---

## 5. OpenClaw

| Field | Value |
|-------|-------|
| Version | v2026.3.7 |
| Node | **Capital** (moved from Forge with routing flip) |
| Memory backend | sqlite-vec — 768d nomic embeddings |
| Workspace | Obsidian vault (synced via Syncthing) |

---

## 6. Forge Services (LAST-RESORT / DEV — 100.97.220.115)

| Port | Service | Launchd Plist | Purpose |
|------|---------|---------------|---------|
| `:1234` | LM Studio | (LM Studio app) | MLX models — last-resort inference |
| `:5100` | Samuel MCP | `com.samuel.local.mcp` | MCP interface |
| `:5101` | Samuel Bridge | `com.samuel.local.bridge` | REST bridge + approval comms |
| `:5130` | LiteLLM proxy | `com.samuel.local.litellm` | Multi-model router |
| `:8080` | Open WebUI | `com.samuel.local.openwebui` | Chat interface |

---

## 7. Sanctuary Services (FALLBACK — 100.100.202.16)

| Port | Service | Notes |
|------|---------|-------|
| `:1234` | LM Studio | allura-8b + qwen3.5-4b + nomic |
| `:5100` | Samuel MCP | First fallback |
| `:5101` | Samuel Bridge | First fallback |
| `:5130` | LiteLLM proxy | First fallback |
| `:5122` | Open WebUI proxy | nginx auth-inject → bridge |
| `:8080` | Open WebUI | Docker container |

**Runtime path**: `/home/mikefrethy/samuel-worker`
**IT triage**: `SAMUEL_IT_TRIAGE_ENABLED=1` (morning 10am + midday 1pm)
**SSH**: `mikefrethy@100.100.202.16` (Tailscale SSH)

---

## 8. Vault Sync (Syncthing)

Forge sends read-only to Ubuntu nodes. Syncthing v1.30.0 (Ubuntu), v2.0.15 (Forge).

| Folder | Forge → Capital | Forge → Sanctuary |
|--------|----------------|-------------------|
| `obsidian-vault` | Synced | Synced |
| `ha-config` | Synced | Not shared |

Capital RAG root: `SAMUEL_RAG_ROOT_HA=/home/mikefrethy/ha-config`

---

## 9. HA Connection

Samuel connects to Home Assistant via **real-time WebSocket subscriber** (`samuel/ha_websocket.py`).

| Field | Value |
|-------|-------|
| Implementation | `samuel/ha_websocket.py` |
| Reconnect | Auto-reconnect with exponential backoff |
| Latency | <1s (real-time event stream) |

---

## 10. RETIRED — Do Not Reference

| Component | Status |
|-----------|--------|
| Ollama | RETIRED — not installed on any node. |
| Qdrant | RETIRED — replaced by OpenClaw sqlite-vec. |
| Dev Hive `:5120` | RETIRED — disabled on all nodes. |
| Worker MCP `:5190` | RETIRED — not running anywhere. |
| Robot City `:5110` | RETIRED — visualizer deprecated. |
| `samuel-vision` alias | RETIRED — no VL models in stack. |
| `qwen/qwen3-8b` on Sanctuary | RETIRED — replaced by qwen3.5-4b + allura-8b. |
| `qwen/qwen3-14b` on Sanctuary | RETIRED — 14b is now local on Capital only. |
| `mxbai-embed-large-v1` (1024d) | RETIRED — replaced by nomic-embed-text-v1.5 (768d). |
| Shadow polling (HA) | RETIRED — replaced by WebSocket subscriber. |

---

## 11. Repo Paths

| Repo | Forge Path | Purpose |
|------|-----------|---------|
| `samuel-system` | `~/Documents/GitHub/samuel-system` | Canonical source — MCP server, bridge, LiteLLM, governance |
| `ha-config` | `~/Documents/GitHub/ha-config` | HA YAML packages + Lovelace — deployed via `deploy_ha_config.sh` |
| `Obsidian` | `~/Documents/GitHub/Obsidian` | Samuel's memory, identity, sessions; OpenClaw workspace |

**Capital/Sanctuary runtime path**: `~/samuel-worker` (deployed clone of `samuel-system`)

---

## 12. Deploy

### Capital / Sanctuary — Systemd (Ubuntu)

```bash
cd ~/samuel-worker && git pull --ff-only origin main
systemctl --user restart samuel-mcp.service
systemctl --user restart samuel-bridge.service
systemctl --user restart samuel-litellm.service
```

### Forge — Launchd (macOS)

```bash
launchctl unload ~/Library/LaunchAgents/com.samuel.local.mcp.plist
launchctl load  ~/Library/LaunchAgents/com.samuel.local.mcp.plist

launchctl unload ~/Library/LaunchAgents/com.samuel.local.bridge.plist
launchctl load  ~/Library/LaunchAgents/com.samuel.local.bridge.plist

launchctl unload ~/Library/LaunchAgents/com.samuel.local.litellm.plist
launchctl load  ~/Library/LaunchAgents/com.samuel.local.litellm.plist
```
