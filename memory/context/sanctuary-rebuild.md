# Sanctuary — Current State & Church LAN Role

## Node
MacBook Pro i9 32GB | Ubuntu 24.04 | Tailscale: `100.104.133.84` | Always-on (church office, SD campus)

_Rebuild complete as of 2026-03-12. This doc describes current running state._

---

## Active Services

| Port | Service | Notes |
|------|---------|-------|
| 1234 | LM Studio | Same 4-model Qwen3 stack as Forge |
| 5100 | Samuel MCP | Mirror of Forge |
| 5101 | Samuel Bridge | Mirror of Forge |
| 5130 | LiteLLM proxy | Same aliases as Forge |
| 5122 | Open WebUI proxy | — |
| 8080 | Open WebUI | — |

**Runtime path**: `/home/mikefrethy/samuel-worker`

**SSH to Forge**: `ssh forge` via `~/.ssh/id_ed25519_forge` (target: `mikefrethy@100.97.220.115`)

---

## Physical Advantage

Sanctuary sits on the church office LAN at SD campus. This gives Samuel (and agents) direct LAN access to church appliances — no Tailscale hop required for on-site hardware.

---

## Planned Church LAN Function Layers

These are not yet implemented but represent the intended future expansion:

| Appliance | What | Notes |
|-----------|------|-------|
| Fortinet FortiGate | Network/firewall management | All ONE&ALL campus locations |
| UniFi / Ruckus | Wi-Fi controller access | SD and RC campus APs |
| RockRMS / RockMCP | Church management (local access) | Direct LAN or via MCP worker |
| NanoMDM / Apple MDM | Apple device management | Church Macs/iPads |
| O&A Printers | Print queue / status | Ricoh/Konica/etc at church offices |

---

## Operating Model (Future Church LAN Queries)

```
Samuel (Forge) → Sanctuary via Tailscale
  → asks: "what's the Fortinet status at RC?"
  → Sanctuary queries FortiGate REST API on church LAN
  → returns status to Samuel/agent

Samuel (Forge) → Sanctuary via Tailscale
  → asks: "check-in system health at SD?"
  → Sanctuary pings Rock RMS on church LAN
  → returns availability
```
