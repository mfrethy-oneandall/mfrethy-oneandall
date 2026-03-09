# Sanctuary Rebuild Plan

## Node
MacBook Pro i9 | Tailscale: `100.100.202.16` | Always-on (church office, SD campus)

---

## Current State (March 2026)

Running legacy Samuel stack:

| Port | Service | Keep? |
|------|---------|-------|
| 5120 | Dev Hive backup coordinator | ✅ Keep |
| 5190 | Dev Hive Worker MCP | ✅ Keep |
| 5130 | LiteLLM proxy (old routes, stale) | ⚠️ Reconfigure or remove |
| 11434 | Ollama `qwen3:14b` | ✅ Keep |
| 3000 | Unknown | 🔍 Investigate first |
| 5122 | Unknown | 🔍 Investigate first |

---

## Physical Advantage

Sanctuary sits on the church office LAN at SD campus. This gives Samuel (and agents) direct LAN access to church appliances — no Tailscale hop required for on-site hardware.

---

## Planned Function Layers (Church LAN)

| Appliance | What | Notes |
|-----------|------|-------|
| Fortinet FortiGate | Network/firewall management | All ONE&ALL campus locations |
| UniFi / Ruckus | Wi-Fi controller access | SD and RC campus APs |
| RockRMS / RockMCP | Church management (local access) | Direct LAN or via MCP worker |
| NanoMDM / Apple MDM | Apple device management | Church Macs/iPads |
| O&A Printers | Print queue / status | Ricoh/Konica/etc at church offices |

---

## Rebuild Steps

1. **Investigate unknowns first:**
   - What is running on port 3000? (`lsof -i :3000` on Sanctuary)
   - What is running on port 5122? (`lsof -i :5122` on Sanctuary)

2. **Strip old stack:**
   - Disable/remove LiteLLM proxy service (old routing, Ollama-based, stale Forge routes)
   - Leave Dev Hive `:5120` and Worker MCP `:5190` running

3. **Keep core intact:**
   - Ollama + `qwen3:14b` — Sanctuary's always-on inference layer
   - Dev Hive backup coordinator — Forge's devhive is retired; Sanctuary owns this
   - Dev Hive Worker MCP — worker execution endpoint

4. **Add church LAN function layers:**
   - Fortinet MCP or REST API wrapper (FortiGate API)
   - UniFi MCP or API wrapper
   - NanoMDM integration endpoint
   - Printer management / status endpoint
   - RockRMS local health monitor (ping + availability check)

5. **Update routing on Forge:**
   - Remove stale LiteLLM aliases: `forge-coder`, `forge-assistant`, `forge-deep` (Ollama uninstalled from Forge)
   - Keep `sanctuary-assistant` → `qwen3:14b` `:11434`
   - Update `samuel-system/specs/hive_nodes.md` with new Sanctuary role

6. **SSH access:**
   - `ssh sanctuary` or `ssh 100.100.202.16`
   - Repo: `/home/mikefrethy/samuel-system`

---

## Operating Model After Rebuild

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
