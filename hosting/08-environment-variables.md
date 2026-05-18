# n8n Environment Variables — Complete Reference

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026
> Every environment variable explained: what it does, why it matters, what breaks if it's wrong.

---

## How to Set Environment Variables

Set these before starting n8n. Three ways:

**Option A — .env file (recommended for Docker):**
```env
N8N_ENCRYPTION_KEY=your-key
GENERIC_TIMEZONE=Asia/Kolkata
```

**Option B — docker-compose.yml environment block:**
```yaml
environment:
  - N8N_ENCRYPTION_KEY=your-key
  - GENERIC_TIMEZONE=Asia/Kolkata
```

**Option C — Shell export (npm installs):**
```bash
export N8N_ENCRYPTION_KEY="your-key"
export GENERIC_TIMEZONE="Asia/Kolkata"
n8n start
```

**Windows (cmd session):**
```cmd
set N8N_ENCRYPTION_KEY=your-key
set GENERIC_TIMEZONE=Asia/Kolkata
n8n start
```

---

## Authentication

| Variable | Default | Required? | What It Does | What Happens If Wrong/Missing |
|----------|---------|-----------|-------------|-------------------------------|
| `N8N_BASIC_AUTH_ACTIVE` | `false` | Yes (production) | Enables a username/password login gate for the n8n UI. Without this, anyone who can reach your URL can use your n8n instance | If `false` on a public URL: your instance is completely open. Anyone can view workflows, credentials, and execution history |
| `N8N_BASIC_AUTH_USER` | — | Required if auth active | The username for the login prompt | If empty while auth is active: n8n may start but no one can log in |
| `N8N_BASIC_AUTH_PASSWORD` | — | Required if auth active | The password for the login prompt | Use a strong password (16+ chars). Weak passwords on public instances get brute-forced |
| `N8N_USER_MANAGEMENT_DISABLED` | `false` | Optional | Disables built-in user management (email invites, multiple users) | Only set to `true` if you are using Basic Auth and want a simpler single-user setup |

---

## Network & URL

| Variable | Default | Required? | What It Does | Why It Matters |
|----------|---------|-----------|-------------|----------------|
| `N8N_HOST` | `localhost` | Yes (if external access needed) | The network interface n8n listens on. `localhost` = only your machine. `0.0.0.0` = all interfaces | If left as `localhost` on a server, no one can reach n8n from outside |
| `N8N_PORT` | `5678` | Optional | The port n8n runs on | Change if 5678 is already taken. Update `WEBHOOK_URL` to match |
| `N8N_PROTOCOL` | `http` | Yes (production) | `http` or `https`. Tells n8n what protocol to use when generating URLs | If set to `http` behind an HTTPS reverse proxy, webhook URLs will be wrong |
| `WEBHOOK_URL` | Auto-detected | **Critical** | The public base URL n8n uses when generating webhook URLs shown in the editor | If not set: n8n guesses using `N8N_HOST:N8N_PORT` — on a VPS this generates `http://localhost:5678` which is unreachable from the internet |
| `N8N_EDITOR_BASE_URL` | Same as WEBHOOK_URL | Optional | Separate URL for the editor UI if different from the webhook URL | Only needed if editor and webhook traffic go through different domains |

**Example for a VPS with domain `n8n.yourdomain.com`:**
```bash
WEBHOOK_URL=https://n8n.yourdomain.com
N8N_PROTOCOL=https
N8N_HOST=0.0.0.0
N8N_PORT=5678
```

---

## Security & Encryption

| Variable | Default | Required? | What It Does | Why It Matters |
|----------|---------|-----------|-------------|----------------|
| `N8N_ENCRYPTION_KEY` | Random (on first start) | **Critical** | A 32-character secret key used to encrypt all credentials stored in n8n's database | This is the most important variable. If you lose it or change it, all saved credentials become permanently unreadable. Back it up immediately. Never change it once set |
| `N8N_SECURE_COOKIE` | `true` | Optional | Sets the `Secure` flag on cookies, requiring HTTPS | Set to `false` only for local HTTP development. On any public URL, leave as `true` |

**How to generate a strong encryption key:**
```bash
# Linux/Mac
openssl rand -hex 32
# Example: a8f3b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1

# Windows PowerShell
[System.Convert]::ToBase64String([System.Security.Cryptography.RandomNumberGenerator]::GetBytes(32))
```

---

## Database

| Variable | Default | When to Change | What It Does |
|----------|---------|---------------|-------------|
| `DB_TYPE` | `sqlite` | When you need production reliability | `sqlite` = local file. `postgresdb` = PostgreSQL. SQLite is fine for personal use but struggles with concurrent executions |
| `DB_POSTGRESDB_HOST` | — | When DB_TYPE=postgresdb | Hostname of your PostgreSQL server. Use `localhost` if same machine, or service name `postgres` in Docker Compose |
| `DB_POSTGRESDB_PORT` | `5432` | Rarely | PostgreSQL port |
| `DB_POSTGRESDB_DATABASE` | `n8n` | Optional | The database n8n should use |
| `DB_POSTGRESDB_USER` | `root` | Required for Postgres | The database user |
| `DB_POSTGRESDB_PASSWORD` | — | Required for Postgres | The database password |
| `DB_POSTGRESDB_SCHEMA` | `public` | Optional | PostgreSQL schema name |

---

## Timezone

| Variable | Default | Required? | What It Does |
|----------|---------|-----------|-------------|
| `GENERIC_TIMEZONE` | Server timezone (usually UTC) | **Yes — always set this** | Controls how Schedule Trigger interprets times. If server is UTC and you set "9 AM", it fires at 9 AM UTC = 2:30 PM IST |
| `TZ` | System default | Yes (Docker) | System timezone for the container. Set the same value as `GENERIC_TIMEZONE` |

**Timezone values for common regions:**
```
India (IST):   Asia/Kolkata
UAE (GST):     Asia/Dubai
UK (GMT/BST):  Europe/London
US East:       America/New_York
US West:       America/Los_Angeles
Singapore:     Asia/Singapore
```

---

## Execution & Storage

| Variable | Default | What It Does |
|----------|---------|-------------|
| `EXECUTIONS_PROCESS` | `main` | `main` = executions run in the main process. `own` = each execution gets its own process (more stable, uses more RAM) |
| `EXECUTIONS_DATA_SAVE_ON_ERROR` | `all` | `all` = save data even on failures. `none` = never save. `all` recommended for debugging |
| `EXECUTIONS_DATA_SAVE_ON_SUCCESS` | `all` | Same but for successful executions. Set to `none` in production to save disk space once workflows are stable |
| `EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS` | `true` | Whether to save manual (test) executions |
| `EXECUTIONS_DATA_MAX_AGE` | `336` (14 days) | Hours to keep execution data. Set lower (e.g., `72`) to keep disk usage low |
| `N8N_DEFAULT_BINARY_DATA_MODE` | `default` | `default` = store binary data in memory. `filesystem` = store on disk. Use `filesystem` for large files |
| `N8N_BINARY_DATA_STORAGE_PATH` | `/home/node/.n8n` | Where binary files are stored. Only used when mode is `filesystem` |

---

## Paths & Storage

| Variable | Default | What It Does |
|----------|---------|-------------|
| `N8N_USER_FOLDER` | `~/.n8n` | Where n8n stores its data: database, credentials, config. Change if disk is small |
| `N8N_CUSTOM_EXTENSIONS` | — | Path to a folder containing custom n8n nodes installed manually |

---

## Email (Password Reset, User Invites)

| Variable | What It Does |
|----------|--------------|
| `N8N_EMAIL_MODE` | `smtp` to enable email sending |
| `N8N_SMTP_HOST` | SMTP server hostname |
| `N8N_SMTP_PORT` | SMTP port (587 for TLS, 465 for SSL) |
| `N8N_SMTP_USER` | SMTP username (usually the email address) |
| `N8N_SMTP_PASS` | SMTP password |
| `N8N_SMTP_SENDER` | From address for n8n emails |
| `N8N_SMTP_SSL` | `true` or `false` |

---

## Complete .env Template

Copy this as a starting point. Replace every placeholder.

```env
# ============================================================
# SECURITY — generate once and never change
# ============================================================
N8N_ENCRYPTION_KEY=paste-your-openssl-rand-hex-32-output-here

# ============================================================
# AUTHENTICATION — always enable on any non-localhost instance
# ============================================================
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=ChangeThisToAStrongPassword!

# ============================================================
# TIMEZONE — always set explicitly
# ============================================================
GENERIC_TIMEZONE=Asia/Kolkata
TZ=Asia/Kolkata

# ============================================================
# NETWORK — set WEBHOOK_URL to your public domain
# ============================================================
N8N_PROTOCOL=https
N8N_HOST=0.0.0.0
N8N_PORT=5678
WEBHOOK_URL=https://n8n.yourdomain.com

# ============================================================
# DATABASE — use sqlite for testing, postgresdb for production
# ============================================================
DB_TYPE=sqlite
# DB_TYPE=postgresdb
# DB_POSTGRESDB_HOST=postgres
# DB_POSTGRESDB_PORT=5432
# DB_POSTGRESDB_DATABASE=n8n
# DB_POSTGRESDB_USER=n8n
# DB_POSTGRESDB_PASSWORD=StrongDBPassword!

# ============================================================
# EXECUTION DATA RETENTION
# ============================================================
EXECUTIONS_DATA_MAX_AGE=168
EXECUTIONS_DATA_SAVE_ON_SUCCESS=all
EXECUTIONS_DATA_SAVE_ON_ERROR=all
```

---

## Official Reference

Full list: https://docs.n8n.io/hosting/configuration/environment-variables/
