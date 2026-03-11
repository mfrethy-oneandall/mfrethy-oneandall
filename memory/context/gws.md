# Google Workspace CLI (gws) — Setup & Reference
_Last updated: 2026-03-10_

## Overview
`gws` (`@googleworkspace/cli`) replaces GAM as the primary Google Workspace admin CLI. It is a Rust binary distributed via npm, built on Google's Discovery Service (dynamic command surface).

**GAM is going away. Use `gws` and `gws-admin` for all GWS operations.**

## Installation
```bash
npm install -g @googleworkspace/cli   # installs as 'gws'
# Current version: 0.9.1
# Binary: /Users/mikefrethy/.nvm/versions/node/v24.14.0/bin/gws
```

## Auth Setup (ADC file — keychain workaround)
gws 0.9.1 has a macOS keychain bug — it writes `credentials.enc` but never stores the decryption key in the keychain, making credentials unreadable. Workaround: authenticate via a Python OAuth flow that writes directly to the ADC file path.

**NEVER run `gws auth logout`** — it deletes `application_default_credentials.json` (the shared ADC file), breaking gws-admin and gws-mcp. If accidentally run, follow the re-auth steps below.

**DO NOT use `gcloud auth application-default login` for re-auth** — gcloud only grants GCP scopes (cloud-platform, sqlservice.login) and ignores Workspace API scopes even when you specify them via `--scopes`.

**To re-authenticate (token expiry or accidental logout):**
```bash
/usr/bin/python3 ~/Documents/GitHub/oneandall-it-plugins/gws_reauth.py
```
Opens a browser → sign in as mike.frethy@oneandall.church → approve ALL permissions. Saves token to `~/.config/gcloud/application_default_credentials.json` with all required scopes.

**Permanent env var in `~/.zshrc`:**
```bash
export GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE=/Users/mikefrethy/.config/gcloud/application_default_credentials.json
```

**GCP project:** `gam-project-hcgi6` (OAuth client ID `979496069444-...`)
**Client secret:** `~/.config/gws/client_secret.json`

**Known warning (non-fatal):** `gws gmail` prints "failed to decrypt token_cache.json" on every call — gws creates its own encrypted token cache that it can't decrypt due to the keychain bug. This is cosmetic: gws falls back to the ADC file and returns exit 0. gws-mcp only surfaces stderr to Cowork on non-zero exit, so Cowork never sees it.

## Common gws Commands
```bash
# Send email as mike.frethy@oneandall.church
gws gmail +send --to user@example.com --subject "Subject" --body "Body text"

# Reply, forward
gws gmail +reply --message-id <id> --body "Reply text"
gws gmail +forward --message-id <id> --to user@example.com

# Triage inbox
gws gmail +triage

# Drive, Sheets, Docs, etc.
gws drive files list
gws sheets spreadsheets get --params '{"spreadsheetId":"ID"}'
gws calendar events list --params '{"calendarId":"primary"}'
```

## Admin Directory — gws-admin wrapper
The Admin SDK Directory API (`admin:directory_v1`) is NOT yet a named service in gws 0.9.1, and the escape hatch syntax (`gws admin:directory_v1 ...`) has a parser bug. A Python wrapper is installed at `/usr/local/bin/gws-admin`.

```bash
# List all @oneandall.church accounts
gws-admin users list

# Get a specific user
gws-admin users get mike.frethy@oneandall.church

# Create a new email account
gws-admin users create \
  --first Jane --last Doe \
  --email jane.doe@oneandall.church \
  --password TempPass123! \
  --change-password

# Suspend / restore / delete
gws-admin users suspend user@oneandall.church
gws-admin users restore user@oneandall.church
gws-admin users delete user@oneandall.church
```

`gws-admin` uses `/usr/bin/python3` with `google-auth` + `google-api-python-client` installed to `~/Library/Python/3.9/`. It reads from the same ADC credentials file.

## Known Issues / Limitations
- **Keychain bug**: `gws auth login` writes encrypted credentials but loses the keychain key — always use ADC file approach above
- **Admin Directory escape hatch broken**: `gws admin:directory_v1` parser bug in 0.9.1 — use `gws-admin` instead
- **No domain-wide delegation**: gws has no DWD support; not needed for current use case
- **HTML email / CC/BCC**: `gws gmail +send` is plain text only; use raw API for HTML (`gws gmail users messages send --json '...'`)

---

## gws-mcp — MCP Server (Always-On in Claude Desktop)

A FastMCP Python server wrapping `gws` and `gws-admin` for use in Claude Desktop and Cowork sessions.

**Source:** `~/Library/Application Support/Claude/gws-mcp/`
**Venv:** `~/Library/Application Support/Claude/gws-mcp/.venv/` (Python 3.13, built via `uv sync`)
**Stable plugin source:** `~/Documents/GitHub/oneandall-it-plugins/3.1.0/mcp-servers/gws-mcp/`

### claude_desktop_config.json entry
```json
"gws": {
  "command": "/opt/homebrew/bin/uv",
  "args": ["run", "--directory",
    "/Users/mikefrethy/Library/Application Support/Claude/gws-mcp",
    "python", "-m", "gws_mcp"],
  "env": {
    "GWS_DOMAIN": "oneandall.church",
    "GWS_SENDER": "mike.frethy@oneandall.church",
    "GWS_GMAIL_BIN": "/Users/mikefrethy/.nvm/versions/node/v24.14.0/bin/gws",
    "GWS_ADMIN_BIN": "gws-admin",
    "GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE": "/Users/mikefrethy/.config/gcloud/application_default_credentials.json"
  }
}
```

**IMPORTANT: GWS_GMAIL_BIN must be the full nvm path** — Claude Desktop spawns with minimal PATH that doesn't include nvm.

### Tools exposed
| Tool | Description |
|------|-------------|
| `gws_user_get` | Look up a GWS user by email |
| `gws_user_list` | List all users in domain |
| `gws_user_create` | Create new GWS account (first, last, email, password, change_password_on_login) |
| `gws_user_suspend` | Suspend a user account |
| `gws_user_restore` | Restore a suspended account |
| `gws_gmail_send` | Send email with full CC/BCC via RFC 2822 (to, subject, body, cc?, bcc?) |
| `gws_gmail_triage` | Show unread inbox summary |

### Rebuild venv (if needed)
```bash
cd ~/Library/Application\ Support/Claude/gws-mcp
/opt/homebrew/bin/uv sync
```
