# Memory — Mike Frethy (Operator)

_Full operator hot-cache for all agents (Cowork, Claude Code, Samuel, OpenClaw)._
_→ Cross-repo map: [memory/repos.md](memory/repos.md)_

---

## Me
Mike Frethy, IT Administrator at ONE&ALL Church, San Dimas campus.
Email: mike.frethy@oneandall.church | ClickUp ID: 88227395

## Repos
| Repo | Org | Purpose |
|------|-----|---------|
| `mfrethy-oneandall` | mfrethy-oneandall | **THIS REPO** — personal operator hub, cross-repo memory |
| `samuel-system` | mfrethy-oneandall | Samuel AI hive coordinator (Forge-primary) |
| `ha-config` | mfrethy-oneandall | Home Assistant YAML config (deployed via rsync from Forge) |
| `Obsidian` | mfrethy-oneandall | Samuel's consciousness layer — OpenClaw workspace |
| `RockProduction` | ONE-ALL-Church | ONE&ALL Rock RMS production web app + church context |
| `oneandall-it-plugins` | mfrethy-oneandall | Cowork plugin: IT skills (CP&A, GWS, intake) + gws-mcp server |
→ Full details: [memory/repos.md](memory/repos.md)

## Organization
**ONE&ALL Church** (CCV SoCal). Campuses: **SD** = San Dimas (Mike's home, ID 6), **RC** = Rancho Cucamonga (ID 7), **WC** = West Covina (ID 9), **DI** = ONE&ALL Digital (ID 8)

## Key People
| Name | Role | Notes |
|------|------|-------|
| **Brian Davis** | IT+Digital Director (Mike's boss) | brian.davis@oneandall.church — actively commits to RockProduction |
| **Amy P-C** / **Amy Pond-Cirelli** | Operations/Admin | All Staff Huddle, Prayer Room, Divvy |
| **Delaney Vines** | Creative lead | Creative Weekly Kickoff Mon 9am |
| **Joshua Adams** | Staff @ RC | IT requests for Rancho campus, Wi-Fi upgrade |
| **Dawn Frazer** | Staff | Escalates check-in / tech issues to IT |
| **Audrey C** / **Audrey Ceballos** | Comms/Admin | CP&A "Waiting for Agreement" follow-ups |
| **Suzanne Martel** | Kids ministry | Kids curriculum filming |
| **Heather Jarvie** | HR/Onboarding | New staff email setup |
| **Rachel Lynch** | Staff @ RC | RC issues (ReGen check-in) |
| **Tricia N** | Receptionist | receptionist@oneandall.church |
| **Randy Aufrecht** | Former IT (~2 yrs ago) | Email forwarded to Mike — needs deprovisioning |
| **Jojo** / **Jolene Pascasio** | Local outreach | Community groups, SD |
→ Full list: [memory/people/key-contacts.md](memory/people/key-contacts.md)

## Terms
| Term | Meaning |
|------|---------|
| **CP&A** | Confidentiality Policy & Agreement — volunteer onboarding workflow in Rock |
| **RC** | Rancho Cucamonga campus |
| **SD** | San Dimas campus |
| **Next Gen Check-In** | Rock RMS modern check-in platform (Mike's major 2026 rollout) |
| **ReGen** | ReGeneration — Thu evening ministry at RC (recovery/discipleship) |
| **Rock** / **Rock RMS** | Church management system (ChMS) |
| **Divvy** | Corporate spend & expense cards (via BILL) |
| **Samuel** | Mike's AI hive — the *world*, not one machine. Each computer is a city-node. |
| **forge** | MacBook M4 Pro Max — PRIMARY Samuel node (`100.97.220.115`) |
| **sanctuary** | MBP i9 — always-on backup, church LAN (`100.104.133.84`) |
| **capital** | Mac mini Ubuntu — OFFLINE, rebuild pending (`100.112.13.70`) |
| **OpenClaw** | Samuel's agent runtime on Forge (`ws://127.0.0.1:18789`) |
| **Dev Hive** | Task queue — RETIRED on Forge, runs on Sanctuary `:5120` |
| **Next Gen (org)** | Central ONE&ALL staff not tied to campus budget; Mike is Next Gen |
→ Full glossary: [memory/glossary.md](memory/glossary.md)

## Active Projects (IT)
| Project | Status | Notes |
|---------|--------|-------|
| Next Gen Check-in Rollout (Youth + Kids) | 🟡 In Progress | Rolling out new check-in system at both campuses |
| Joshua Adams – Rancho Chapel Wi-Fi upgrade | 🔵 In Progress | Wi-Fi upgrade at Rancho Chapel |
| CP&A Queue — daily Claude sync | 🔵 In Progress | Auto-synced via Cowork 10am task |
| Capital rebuild | ⚠️ Pending | Hardware failure — rebuild pending |
| Sanctuary rebuild | 🟡 Planned | Strip old stack, wire church LAN function layers |

## Recurring Meetings
| Meeting | When | Where |
|---------|------|-------|
| Creative Weekly Kickoff | Mon 9–10am | Creative office |
| Staff & Serve Prayer Room | Mon 12–1pm | Worship Center SD |
| All Staff Huddle | Tue 11:45am–12pm | Patio |
| Prayer Room | Tue 12–1pm | Worship Center SD |
| Divvy Reminder | Tue 5pm | (calendar reminder) |

## Samuel Hive Quick Reference
| Node | IP | Status | Key Services |
|------|----|--------|--------------|
| forge | `100.97.220.115` | **PRIMARY** | MCP `:5100`, LM Studio `:1234`, OpenClaw `:18789`, LiteLLM `:5130`, Open WebUI `:8080` |
| sanctuary | `100.104.133.84` | Online — full backup | LM Studio `:1234`, MCP `:5100`, Bridge `:5101`, LiteLLM `:5130`, Open WebUI `:5122`/`:8080` |
| capital | `100.112.13.70` | **OFFLINE — rebuild pending** | — |
| home-assistant | `100.99.5.79` | Working | HAOS bare metal, MCP `:8123` |
→ Full details: [memory/context/samuel-system.md](memory/context/samuel-system.md)

## IT Tooling
| Tool | Purpose | Notes |
|------|---------|-------|
| `gws` | Google Workspace CLI (replaces GAM) | npm `@googleworkspace/cli` v0.9.1 |
| `gws-admin` | Admin Directory wrapper | `/usr/local/bin/gws-admin` — create/manage email accounts |
| `gcloud` | GCP CLI — used for gws auth | ADC credentials at `~/.config/gcloud/application_default_credentials.json` |
→ Full setup + commands: [memory/context/gws.md](memory/context/gws.md)

## Preferences
- No Microsoft Office (Google Workspace only)
- Project tracker: ClickUp (IT Team > IT Service Desk > IT Requests)
- Knowledge base: Google Drive + repos
- Home Assistant: samuel-system for automation/AI
- Works from home Fri + Mon
