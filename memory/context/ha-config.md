# Home Assistant — Runtime Reference
_Ground truth: ha-config/docs/current_state.md (2026-03-08)_

---

## Infrastructure

| Field | Value |
|-------|-------|
| Host | Mac mini — **bare metal HAOS** (migrated from VirtualBox, Mar 2026) |
| LAN | `http://10.20.99.2:8123` |
| Tailscale | `http://100.99.5.79:8123` |
| SSH | `root@10.20.99.2` port `22222`, key `~/.ssh/ha_root_sync_key` |
| Git on HA | **None** — no git sync on the HA box. Config deployed only via `deploy_ha_config.sh` from Forge. |
| HA MCP (native) | `http://100.99.5.79:8123/mcp_server/sse` — action tools (lights, media, climate) |
| HA MCP (agentic) | `~/homeassistant-mcp/` — inspection tools (get_state, list_entities, call_service, etc.) |

---

## Active Packages

All behavior lives in `packages/`. Deployed via allowlist in `deploy_ha_config.sh`.

| Package | Owns |
|---------|------|
| `house_mode.yaml` | House mode state machine, presence state, sunrise reset |
| `presence.yaml` | Multi-signal presence detection, re-entry logic |
| `system_flags.yaml` | `is_dark` flag, sunset/sunrise triggers, Christmas flag |
| `porch.yaml` | Porch lighting + Christmas cycle + quiet-hours override |
| `upstairs_hallway.yaml` | Hallway motion automation + manual override timer |
| `stairs_safety_assist.yaml` | Stair safety motion + manual override timer |
| `master_bath_motion.yaml` | Bathroom motion lighting (day/night modes) |
| `upstairs_routines.yaml` | Going Upstairs + Living Room Dim buttons and scripts |
| `tv_watching.yaml` | Fire TV auto-dim + restore |
| `morning_routine.yaml` | Morning wake-up Fire TV + lights automation |
| `lighting_standards.yaml` | Brightness/CT helpers + normalization automations |
| `samuel_voice.yaml` | Voice pipeline capture, `script.samuel_respond`, `script.samuel_speak` |
| `mic_test.yaml` | ESPHome mic test node helpers, automation, pass rate sensor |

**Deleted / retired packages** (not active, not deployed):
- `samuel_panel.yaml`, `samuel_commands.yaml`, `morning_health.yaml` — Samuel conversational panel
- `nathan_*` stack — Nathan governance stack (entire thing removed)
All governance/intelligence is now in `samuel-system`, not HA.

---

## Core State Entities

| Entity | Purpose |
|--------|---------|
| `input_select.house_mode` | Primary home state: Home / Away / Night / Vacation / Guest |
| `input_boolean.is_dark` | Sunset/sunrise dark flag |
| `input_boolean.quiet_hours` | Night/quiet mode flag |
| `input_boolean.goodnight` | Bedtime trigger |
| `binary_sensor.anyone_home` | Aggregate presence |
| `input_text.samuel_last_heard` | Raw voice STT transcription (observability only) |
| `input_text.samuel_last_response` | Last Samuel TTS output (≤255 chars) |

---

## Key Entity IDs (post bare metal rebuild)

Zigbee entity IDs changed on bare metal rebuild. Current canonical IDs:

| Area | Entity |
|------|--------|
| Living Room | `light.zb_bulb_living_room_1`, `light.zb_bulb_living_room_2` |
| Upstairs Hallway | `light.zb_bulb_upstairs_hall_router` |
| Front Porch | `light.front_porch_light_bulb`, `light.front_porch_front_light` |
| Front Room | `light.front_room_front_reading_light` |
| Master Bath | `light.master_bathroom_light` |
| Master Bedroom | `light.master_bedroom_light_2` |
| Owen's Room | `light.smart_lighting_tunable_white_and_color` |

> `switch.fireplace_lights_socket_1` — **NOT YET PAIRED** on bare metal. Do not reference until confirmed.

---

## Voice Pipeline

```
User speaks → ESPHome INMP441 mic → STT → esphome.samuel_voice_heard event
→ samuel_voice_pipeline_capture automation (samuel_voice.yaml)
  → input_text.samuel_last_heard  (raw STT, observability)
  → script.samuel_respond         (Alexa TTS — Living Room Echo)
```

Voice path is **local-only** — does not call a remote Brain endpoint. Samuel Brain (Capital) is offline anyway.

**ESPHome Mic Test Node:**
- Board: ESP32-WROOM-32 + INMP441 I2S MEMS mic
- Pins: GPIO32 (SCK), GPIO33 (WS), GPIO25 (SD)
- Event: `esphome.samuel_voice_heard` with transcribed text on `on_stt_end`
- Status: software verified — check `sensor.mic_test_node_connected` for hardware status

---

## Deployment

`deploy_ha_config.sh` — allowlist-based rsync + SSH + restart, run from Forge.

To add a new package: (1) create the file, (2) add to `ALLOWED_FILES` in `deploy_ha_config.sh`, (3) run the script.

Post-deploy: script writes commit hash to `~/.samuel/ha_runtime_commit`, waits 30s for HA restart, checks entity registration, pushes commit hash to `input_text.ha_deployed_commit` via REST API.

Deploy command: `cd /Users/mikefrethy/Documents/GitHub/ha-config && bash deploy_ha_config.sh`

---

## Samuel ↔ HA Architecture Principle

Samuel does **not** embed REST commands inside HA packages. HA is an appliance. Samuel talks to it via the **Home Infrastructure Bridge** in `samuel-system` (MCP tools wrapping HA REST/WebSocket API). No samuel_brain or samuel_gate services live in this repo — all intelligence is in `samuel-system`.
