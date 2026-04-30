# n8n Complete Hosting Guide

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Covers:** Every installation method, every configuration field, every challenge — Windows, macOS, Linux, Docker, Cloud VPS, production setup, security, backups, and best practices.

---

## How to Use This Guide

Read the decision tree below first. Jump to the section that matches your environment. Every field, every command, and every setting is explained with *why* it matters — not just *what* to type.

---

## Part 0 — Which Hosting Method Should You Use?

```
What is your goal?
│
├── Testing / learning / personal use on my laptop
│   ├── I don't want to use Docker → Part 2 (npm, Windows/Mac/Linux)
│   └── I have Docker / want easy management → Part 3 (Docker Compose)
│
├── Running n8n 24/7 on a server I own (NAS, home server)
│   ├── QNAP / Synology NAS → Part 3 (Docker Compose)
│   └── Linux server → Part 4 (Docker on Linux)
│
└── Production / public-facing / accessible from anywhere
    ├── Cloud VPS (DigitalOcean, Hetzner, etc.) → Part 5 (VPS + SSL)
    └── n8n Cloud → Just use cloud.n8n.io (no self-hosting needed)
```

---

## Part 1 — Environment Variables: Every Field Explained

These are the settings that control how n8n behaves. You set them before starting n8n — either as shell `export` commands, in a `.env` file, or in `docker-compose.yml`.

Understanding these is critical. Getting them wrong causes silent failures that are hard to debug.

### Authentication

| Variable | Default | Required? | What It Does | What Happens If Wrong/Missing |
|----------|---------|-----------|-------------|-------------------------------|
| `N8N_BASIC_AUTH_ACTIVE` | `false` | Yes (production) | Enables a username/password login gate for the n8n UI. Without this, **anyone who can reach your URL can use your n8n instance** | If `false` on a public URL: your instance is completely open. Anyone can view workflows, credentials, and execution history |
| `N8N_BASIC_AUTH_USER` | — | Required if auth active | The username for the login prompt | If empty while auth is active: n8n may start but no one can log in |
| `N8N_BASIC_AUTH_PASSWORD` | — | Required if auth active | The password for the login prompt | Use a strong password (16+ chars). Weak passwords on public instances get brute-forced |
| `N8N_USER_MANAGEMENT_DISABLED` | `false` | Optional | Disables the built-in user management system (email invites, multiple users) | Only set to `true` if you are using Basic Auth instead and want a simpler single-user setup |

### Network & URL

| Variable | Default | Required? | What It Does | Why It Matters |
|----------|---------|-----------|-------------|----------------|
| `N8N_HOST` | `localhost` | Yes (if external access needed) | The network interface n8n listens on. `localhost` = only your machine. `0.0.0.0` = all interfaces (accessible from network) | If you leave this as `localhost` on a server, no one can reach n8n from outside — including Docker containers calling the host |
| `N8N_PORT` | `5678` | Optional | The port n8n runs on | Change if port 5678 is already taken by another app. Update `WEBHOOK_URL` to match |
| `N8N_PROTOCOL` | `http` | Yes (production) | `http` or `https`. Tells n8n what protocol to use when generating URLs | If set to `http` but you're behind an HTTPS reverse proxy, webhook URLs will be wrong (http:// instead of https://) |
| `WEBHOOK_URL` | Auto-detected | **Critical** | The public base URL that n8n uses when generating webhook URLs shown in the editor | If not set: n8n guesses using `N8N_HOST:N8N_PORT` — on a VPS this generates `http://localhost:5678` as the webhook URL, which is unreachable from the internet |
| `N8N_EDITOR_BASE_URL` | Same as WEBHOOK_URL | Optional | Separate URL for the editor UI if it's different from the webhook URL | Only needed if your editor and webhook traffic go through different domains/paths |

**Example for a VPS with domain `n8n.yourdomain.com`:**
```bash
WEBHOOK_URL=https://n8n.yourdomain.com
N8N_PROTOCOL=https
N8N_HOST=0.0.0.0
N8N_PORT=5678
```

### Security & Encryption

| Variable | Default | Required? | What It Does | Why It Matters |
|----------|---------|-----------|-------------|----------------|
| `N8N_ENCRYPTION_KEY` | Random (generated on first start) | **Critical** | A 32-character secret key used to encrypt all credentials stored in n8n's database | **This is the most important variable.** If you lose it or change it, all your saved credentials (API keys, passwords, OAuth tokens) become permanently unreadable. Back it up immediately. Never change it once set |
| `N8N_SECURE_COOKIE` | `true` | Optional | Sets the `Secure` flag on cookies, requiring HTTPS | Set to `false` only for local HTTP development. On any public URL, leave as `true` |

**How to generate a strong encryption key:**
```bash
# Linux/Mac
openssl rand -hex 32
# Example output: a8f3b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1

# Windows PowerShell
[System.Convert]::ToBase64String([System.Security.Cryptography.RandomNumberGenerator]::GetBytes(32))
```

### Database

| Variable | Default | When to Change | What It Does |
|----------|---------|---------------|-------------|
| `DB_TYPE` | `sqlite` | When you need production reliability | `sqlite` uses a local file. `postgresdb` uses PostgreSQL. SQLite is fine for personal use but struggles with concurrent executions and large datasets |
| `DB_POSTGRESDB_HOST` | — | When DB_TYPE=postgresdb | The hostname of your PostgreSQL server. Use `localhost` if on same machine, service name `postgres` if in Docker Compose |
| `DB_POSTGRESDB_PORT` | `5432` | Rarely | PostgreSQL port. Default is 5432 |
| `DB_POSTGRESDB_DATABASE` | `n8n` | Optional | The name of the database n8n should use |
| `DB_POSTGRESDB_USER` | `root` | Required for Postgres | The database user |
| `DB_POSTGRESDB_PASSWORD` | — | Required for Postgres | The database password |
| `DB_POSTGRESDB_SCHEMA` | `public` | Optional | PostgreSQL schema name |

### Timezone

| Variable | Default | Required? | What It Does |
|----------|---------|-----------|-------------|
| `GENERIC_TIMEZONE` | Server timezone (usually UTC) | **Yes — always set this** | Controls how Schedule Trigger interprets times. If your server is UTC and you set a schedule for "9 AM", it fires at 9 AM UTC = 2:30 PM IST. Always set this explicitly |
| `TZ` | System default | Yes (Docker) | The system timezone for the container. Set the same value as `GENERIC_TIMEZONE` |

**Timezone values for common regions:**
```
India (IST):   Asia/Kolkata
UAE (GST):     Asia/Dubai
UK (GMT/BST):  Europe/London
US East:       America/New_York
US West:       America/Los_Angeles
Singapore:     Asia/Singapore
```

### Execution & Storage

| Variable | Default | What It Does |
|----------|---------|-------------|
| `EXECUTIONS_PROCESS` | `main` | `main` = executions run in the main n8n process. `own` = each execution gets its own process (more stable, uses more RAM). Change to `own` if workflows crash n8n |
| `EXECUTIONS_DATA_SAVE_ON_ERROR` | `all` | `all` = save execution data even on failures. `none` = never save. `all` is recommended for debugging |
| `EXECUTIONS_DATA_SAVE_ON_SUCCESS` | `all` | Same but for successful executions. Set to `none` in production to save disk space once workflows are stable |
| `EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS` | `true` | Whether to save manual (test) executions. `false` saves storage |
| `EXECUTIONS_DATA_MAX_AGE` | `336` (14 days) | Hours to keep execution data. After this, old executions are deleted. Set lower (e.g., `72`) to keep disk usage low |
| `N8N_DEFAULT_BINARY_DATA_MODE` | `default` | `default` = store binary data in memory. `filesystem` = store on disk. Use `filesystem` if you process large files or many binary uploads |
| `N8N_BINARY_DATA_STORAGE_PATH` | `/home/node/.n8n` | Where binary files are stored on disk. Only used when `N8N_DEFAULT_BINARY_DATA_MODE=filesystem` |

### Paths & Storage

| Variable | Default | What It Does |
|----------|---------|-------------|
| `N8N_USER_FOLDER` | `~/.n8n` | Where n8n stores its data: `database.sqlite`, credentials, config. Change to a different drive if disk is small |
| `N8N_CUSTOM_EXTENSIONS` | — | Path to a folder containing custom n8n nodes installed manually |

### Email (for password reset, user invites)

| Variable | What It Does |
|----------|-------------|
| `N8N_EMAIL_MODE` | `smtp` to enable email sending |
| `N8N_SMTP_HOST` | SMTP server hostname |
| `N8N_SMTP_PORT` | SMTP port (587 for TLS, 465 for SSL) |
| `N8N_SMTP_USER` | SMTP username (usually the email address) |
| `N8N_SMTP_PASS` | SMTP password |
| `N8N_SMTP_SENDER` | From address for n8n emails |
| `N8N_SMTP_SSL` | `true` or `false` |

---

## Part 2 — Windows: Install Without Docker (npm)

### Who This Is For
You want to run n8n directly on Windows without Docker. Suitable for learning, personal automation, and development.

---

### Step 1 — Install Node.js

**Why:** n8n is a Node.js application. Node.js is the runtime that executes it.

1. Open your browser and go to: **https://nodejs.org**
2. You will see two buttons: **LTS** (Long Term Support) and **Current**. Click **LTS** — it is the stable version recommended for production use
3. The download starts automatically (file name like `node-v20.x.x-x64.msi`)
4. Double-click the downloaded `.msi` file to start the installer
5. Click **Next** through the welcome screen
6. **Accept the license agreement** → click **Next**
7. **Destination Folder:** Leave as default (`C:\Program Files\nodejs\`) unless you have a reason to change it → **Next**
8. **Custom Setup:** Leave all components selected → **Next**
9. **Tools for Native Modules:** ⚠️ **Check this box** — "Automatically install the necessary tools. Note that this also installs Chocolatey." This installs Python and Visual C++ Build Tools which some n8n dependencies need. If you skip this, certain community nodes may fail to install
10. Click **Install** → allow the UAC prompt → wait 2–5 minutes
11. A PowerShell window may open separately to install the additional tools. **Let it complete — do not close it**
12. **Restart Windows** after everything completes

**Verify the install:**
```cmd
Win + R → type cmd → Enter
node --version
```
Expected: `v20.x.x` (or whatever LTS you downloaded)

```cmd
npm --version
```
Expected: `10.x.x`

**⚠️ Troubleshooting:**

| Problem | Cause | Fix |
|---------|-------|-----|
| `node` is not recognized | Node.js not added to PATH | Restart your Command Prompt. If still failing, search "Environment Variables" in Start → Edit system environment variables → PATH → verify `C:\Program Files\nodejs\` is listed |
| `npm` is not recognized | Same as above | Same fix |
| Version shows old version | Old Node.js was already installed | Uninstall old Node.js from Windows Programs, restart, reinstall |

---

### Step 2 — Install n8n

**Why:** This downloads n8n and all its dependencies from the npm registry and makes the `n8n` command available globally.

1. Open **Command Prompt as Administrator**: Press `Win`, type `cmd`, right-click **Command Prompt** → **Run as administrator**
2. You'll see the prompt change to show `Administrator:`
3. Run:
```cmd
npm install n8n -g
```

**What `-g` means:** Install globally — available from any folder, not just the current directory.

**Expected output:** A long list of packages being downloaded. Normal progress looks like:
```
npm warn deprecated ...
npm warn ...
added 874 packages in 2m
```
Warnings are fine. Only `npm ERR!` lines are actual errors.

**⚠️ Troubleshooting:**

| Error | Cause | Fix |
|-------|-------|-----|
| `EACCES: permission denied` | Not running as admin | Close cmd, reopen as Administrator |
| `node-gyp rebuild failed` | Build tools missing | Re-run Node.js installer → check "Tools for Native Modules" → restart |
| `npm ERR! code ENOTFOUND` | No internet / proxy blocking | Check internet connection; if on corporate VPN, configure npm proxy: `npm config set proxy http://proxy:port` |
| `gyp ERR! stack Error: not found: make` | Build tools not installed | Run as admin: `npm install --global --production windows-build-tools` |

---

### Step 3 — Configure n8n (Set Environment Variables)

**Why:** Before starting n8n, you need to tell it how to behave — what port to use, what timezone, and most importantly, a stable encryption key.

**On Windows, set environment variables in the same Command Prompt session before starting n8n:**

```cmd
:: REQUIRED: Set a stable encryption key (generate one and save it)
set N8N_ENCRYPTION_KEY=your-32-character-secret-key-here

:: REQUIRED: Set your timezone
set GENERIC_TIMEZONE=Asia/Kolkata

:: OPTIONAL: Enable authentication (recommended if others can reach your machine)
set N8N_BASIC_AUTH_ACTIVE=true
set N8N_BASIC_AUTH_USER=admin
set N8N_BASIC_AUTH_PASSWORD=YourStrongPassword123!

:: OPTIONAL: Change port if 5678 is busy
set N8N_PORT=5678
```

**⚠️ Important:** These `set` commands only last for the current Command Prompt session. If you close and reopen, they're gone. For permanent settings:
1. Search "environment variables" in Windows Start
2. Click "Edit the system environment variables"
3. Click "Environment Variables" button
4. Under "User variables", click "New" for each variable
5. Variable name: `N8N_ENCRYPTION_KEY`, Variable value: your key

---

### Step 4 — Start n8n

```cmd
n8n start
```

**What you should see:**
```
n8n ready on 0.0.0.0, port 5678
Version: 1.x.x

Editor is now accessible via:
http://localhost:5678

Press "o" to open in Browser, "ctrl + c" to quit
```

**Open n8n:** Go to http://localhost:5678 in your browser

**First-time setup:**
1. n8n shows an onboarding screen asking for your name, email, and password
2. This creates a local account — no data is sent to n8n.io
3. Complete the setup and you're in the editor

**⚠️ Troubleshooting first start:**

| Problem | Fix |
|---------|-----|
| `EADDRINUSE: address already in use :::5678` | Port 5678 is taken. Run: `netstat -ano | findstr :5678` → find the PID → `taskkill /PID <pid> /F`. Or change port: `set N8N_PORT=5679` |
| Browser shows "This site can't be reached" | n8n hasn't fully started yet. Wait 30 seconds and refresh |
| White screen or JS errors in browser | Hard refresh: Ctrl+Shift+R |
| n8n exits immediately | Check the error in the terminal. Usually a missing dependency or port conflict |

---

### Step 5 — Keep n8n Running (Windows Service with PM2)

**Why:** Closing the Command Prompt kills n8n. PM2 runs n8n in the background and restarts it automatically.

```cmd
:: Install PM2
npm install pm2 -g

:: Start n8n under PM2
pm2 start n8n -- start

:: Save the PM2 process list so it survives restarts
pm2 save

:: Configure PM2 to start automatically on Windows boot
pm2 startup
:: PM2 prints a command to run — copy and run it
```

**Useful PM2 commands:**
```cmd
pm2 list           :: Show all running processes
pm2 logs n8n       :: View n8n logs
pm2 restart n8n    :: Restart n8n
pm2 stop n8n       :: Stop n8n
pm2 delete n8n     :: Remove from PM2
```

---

## Part 3 — macOS: Install Without Docker (npm)

### Step 1 — Install Homebrew (Package Manager)

**Why:** Homebrew is the standard package manager for macOS. It makes installing and updating software reliable.

```bash
# Open Terminal (Cmd + Space → type Terminal → Enter)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

The installer will:
- Ask for your Mac password (for sudo access)
- Install Xcode Command Line Tools if not present (takes 5–10 min)
- Install Homebrew itself

**⚠️ Apple Silicon (M1/M2/M3) Macs:** Homebrew installs to `/opt/homebrew/` instead of `/usr/local/`. After install, run:
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

---

### Step 2 — Install Node.js

```bash
brew install node

# Verify
node --version    # v20.x.x
npm --version     # 10.x.x
```

**⚠️ If you get Rosetta errors on Apple Silicon:**
```bash
softwareupdate --install-rosetta --agree-to-license
```

---

### Step 3 — Fix npm Permissions (Prevents Future Errors)

**Why:** By default, npm's global packages folder requires root access, causing `EACCES` errors.

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

Now npm can install global packages without `sudo`.

---

### Step 4 — Install n8n

```bash
npm install n8n -g
```

---

### Step 5 — Configure and Start n8n

```bash
# Set in current terminal session
export N8N_ENCRYPTION_KEY="your-32-character-key-here"
export GENERIC_TIMEZONE="Asia/Kolkata"
export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER=admin
export N8N_BASIC_AUTH_PASSWORD="YourStrongPassword!"

# Start
n8n start
```

**For permanent environment variables on Mac:**
```bash
# Add to ~/.zshrc (zsh, default since macOS Catalina)
nano ~/.zshrc

# Add these lines:
export N8N_ENCRYPTION_KEY="your-32-character-key-here"
export GENERIC_TIMEZONE="Asia/Kolkata"

# Save (Ctrl+X → Y → Enter)
source ~/.zshrc
```

**Keep running with PM2:**
```bash
npm install pm2 -g
pm2 start n8n -- start
pm2 save
pm2 startup   # Copy and run the printed command
```

---

## Part 4 — Linux Server: Install Without Docker (npm)

### Step 1 — Install Node.js 20.x

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# RHEL/CentOS/Fedora
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y nodejs

# Verify
node --version
npm --version
```

### Step 2 — Install n8n

```bash
sudo npm install n8n -g
```

### Step 3 — Create Systemd Service (Run as Daemon)

**Why:** systemd manages n8n as a proper Linux service — auto-starts on boot, auto-restarts on crash, proper logging.

```bash
# Create the service file
sudo nano /etc/systemd/system/n8n.service
```

Paste this — **replace every value in angle brackets `<>` with your actual values:**

```ini
[Unit]
Description=n8n Workflow Automation
After=network.target
Wants=network-online.target

[Service]
# The user that runs n8n — use a dedicated non-root user for security
Type=simple
User=<your-linux-username>
Group=<your-linux-username>

# Working directory
WorkingDirectory=/home/<your-linux-username>

# Every environment variable n8n needs
Environment=N8N_ENCRYPTION_KEY=<your-32-char-key>
Environment=GENERIC_TIMEZONE=Asia/Kolkata
Environment=TZ=Asia/Kolkata
Environment=N8N_HOST=0.0.0.0
Environment=N8N_PORT=5678
Environment=N8N_PROTOCOL=https
Environment=WEBHOOK_URL=https://<your-domain.com>
Environment=N8N_BASIC_AUTH_ACTIVE=true
Environment=N8N_BASIC_AUTH_USER=admin
Environment=N8N_BASIC_AUTH_PASSWORD=<strong-password>
Environment=EXECUTIONS_DATA_MAX_AGE=168

# The command to run
ExecStart=/usr/bin/n8n start

# Restart behavior
Restart=always
RestartSec=10

# Resource limits
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

Save: `Ctrl+X → Y → Enter`

```bash
# Reload systemd to pick up the new service file
sudo systemctl daemon-reload

# Enable auto-start on boot
sudo systemctl enable n8n

# Start n8n now
sudo systemctl start n8n

# Check status — look for "active (running)"
sudo systemctl status n8n

# Follow live logs
sudo journalctl -u n8n -f
```

**Useful service commands:**
```bash
sudo systemctl restart n8n    # Restart after config changes
sudo systemctl stop n8n       # Stop
sudo systemctl disable n8n    # Remove from auto-start
```

---

## Part 5 — Docker: All Platforms (Windows, Mac, Linux)

### Step 1 — Install Docker

#### Windows
1. Go to: **https://www.docker.com/products/docker-desktop**
2. Click **Download for Windows**
3. Run `Docker Desktop Installer.exe`
4. When asked about WSL2: click the link and follow Microsoft's WSL2 install guide. This is required. WSL2 is a lightweight Linux environment inside Windows that Docker needs
5. After WSL2 is installed, continue the Docker installer
6. **Restart Windows** when prompted
7. After restart, Docker Desktop opens automatically
8. Wait for the whale icon in the taskbar to stop animating and show "Docker Desktop is running"

**Verify:**
```cmd
docker --version
docker run hello-world
```
Expected: The hello-world message prints without errors.

**⚠️ Windows Troubleshooting:**
| Problem | Fix |
|---------|-----|
| "Docker Desktop requires WSL2" | Follow: https://aka.ms/wsl2install |
| "Hardware virtualization not enabled" | Restart PC → enter BIOS (usually Del or F2) → find Virtualization/VT-x → Enable it → Save |
| Docker Desktop won't start after install | Try running as Administrator |

#### macOS
1. Go to: **https://www.docker.com/products/docker-desktop**
2. Click **Download for Mac**
3. Choose **Apple Silicon** (M1/M2/M3) or **Intel Chip** based on your Mac
4. How to check: Apple menu → About This Mac → Chip: "Apple M..." = Apple Silicon; "Intel" = Intel
5. Open the `.dmg` → drag Docker to Applications
6. Open Docker from Applications → authorize when asked
7. Wait for Docker to start (menu bar icon stops animating)

#### Linux
```bash
# Install Docker Engine (the server, no GUI needed)
curl -fsSL https://get.docker.com | sh

# Add your user to the docker group (allows running docker without sudo)
sudo usermod -aG docker $USER
newgrp docker    # Apply group change without logout

# Install Docker Compose plugin
sudo apt-get install -y docker-compose-plugin

# Verify
docker --version
docker compose version
docker run hello-world
```

---

### Step 2 — Create Your Project Folder

```bash
# Mac/Linux
mkdir ~/n8n-stack && cd ~/n8n-stack

# Windows Command Prompt
mkdir C:\n8n-stack && cd C:\n8n-stack

# Windows PowerShell
New-Item -ItemType Directory -Path C:\n8n-stack
Set-Location C:\n8n-stack
```

**Why this matters:** All your Docker Compose files, environment files, and nginx configs live here. Keep it organized.

---

### Step 3 — Create Your .env File

**Why `.env` instead of hardcoding in docker-compose.yml:** The `.env` file is never committed to Git (add it to `.gitignore`). Your `docker-compose.yml` can be shared publicly; your `.env` stays secret.

Create a file called `.env` in your project folder:

```env
# Authentication
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=ChangeThisToAStrongPassword!

# Encryption — GENERATE THIS AND NEVER CHANGE IT
# Generate with: openssl rand -hex 32
N8N_ENCRYPTION_KEY=paste-your-generated-32-char-key-here

# Timezone — replace with your timezone
GENERIC_TIMEZONE=Asia/Kolkata
TZ=Asia/Kolkata

# Your public URL (used for webhook URLs shown in editor)
# For local use: http://localhost:5678
# For public domain: https://n8n.yourdomain.com
WEBHOOK_URL=http://localhost:5678

# Database (leave as-is for local use, change for production)
DB_TYPE=sqlite

# Execution data retention (168 = 7 days)
EXECUTIONS_DATA_MAX_AGE=168
```

---

### Step 4 — Create docker-compose.yml

Create `docker-compose.yml` in the same folder:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    # What this line does: pulls the official n8n image from n8n's registry
    # :latest = always use the newest version
    # For pinned version (recommended in production): docker.n8n.io/n8nio/n8n:1.45.0

    container_name: n8n
    # Names the container 'n8n' so you can reference it by name
    # Without this: Docker gives it a random name like 'loving_turing'

    restart: unless-stopped
    # What this means:
    # - always: always restart, even on deliberate docker stop
    # - unless-stopped: restart on failure/reboot, but NOT if you stop it manually
    # - on-failure: only restart if exits with error
    # - no: never auto-restart
    # RECOMMENDATION: use 'unless-stopped' for n8n

    ports:
      - "5678:5678"
    # Format: "HOST_PORT:CONTAINER_PORT"
    # Left side (5678): port on YOUR machine
    # Right side (5678): port inside the container
    # Example: - "8080:5678" means you access n8n at localhost:8080 but it runs on 5678 inside
    # If port 5678 is taken on your machine, change the left number only

    env_file:
      - .env
    # Load all variables from the .env file you created above

    volumes:
      - n8n_data:/home/node/.n8n
    # Format: NAMED_VOLUME:PATH_INSIDE_CONTAINER
    # /home/node/.n8n is where n8n stores everything inside the container
    # n8n_data is a Docker-managed named volume on your host
    # WHY THIS MATTERS: Without this line, ALL your workflows, credentials,
    # and executions are DELETED when the container restarts
    # Always include this line

    # OPTIONAL: Use a host folder instead of named volume:
    # - ./data:/home/node/.n8n
    # This maps the ./data folder on YOUR machine to the container
    # Easier to browse and backup manually

# Define the named volumes used above
volumes:
  n8n_data:
    driver: local
    # 'local' = stored on the local machine's filesystem
    # Location (Linux): /var/lib/docker/volumes/n8n-stack_n8n_data/_data
    # Location (Windows/Mac): Managed by Docker Desktop VM
```

---

### Step 5 — Start n8n

```bash
# Start in detached (background) mode
docker compose up -d

# What -d does: runs containers in background, returns you to the prompt
# Without -d: containers run in foreground, logs stream to terminal, Ctrl+C stops everything

# Verify it started
docker compose ps
```

Expected output:
```
NAME    IMAGE              STATUS        PORTS
n8n     .../n8n:latest     Up 3 seconds  0.0.0.0:5678->5678/tcp
```

```bash
# View logs (useful for debugging)
docker compose logs n8n

# Follow logs in real-time (Ctrl+C to stop following)
docker compose logs -f n8n

# View last 50 lines
docker compose logs --tail=50 n8n
```

Open: **http://localhost:5678**

---

### Step 6 — Docker with PostgreSQL (Production-Grade Setup)

**Why PostgreSQL instead of SQLite:**
- SQLite locks the database file during writes — can cause errors under concurrent load
- PostgreSQL handles many simultaneous executions efficiently
- PostgreSQL supports proper backups, replication, and long-term growth
- Required for n8n Queue Mode (multiple worker instances)

Update your `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: n8n-postgres
    restart: unless-stopped
    environment:
      # These create the database on first start
      POSTGRES_USER: n8n          # Database username
      POSTGRES_PASSWORD: ${DB_POSTGRESDB_PASSWORD}   # From .env
      POSTGRES_DB: n8n            # Database name
    volumes:
      - postgres_data:/var/lib/postgresql/data
      # This stores the actual database files
      # NEVER delete this volume — you lose all your data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U n8n"]
      # Checks if Postgres is ready to accept connections
      interval: 5s
      timeout: 5s
      retries: 5
      # n8n will wait for Postgres to be healthy before starting

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    env_file:
      - .env
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy
        # Waits for Postgres to pass its healthcheck before n8n starts
        # Without this: n8n starts before Postgres is ready and crashes

volumes:
  postgres_data:
  n8n_data:
```

Add to `.env`:
```env
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
# Use the Docker service name 'postgres' — Docker's internal DNS resolves it
# If Postgres was on a separate server: use that server's IP or hostname
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=GenerateAStrongPasswordHere!
DB_POSTGRESDB_SCHEMA=public
```

---

### Step 7 — Docker with Ollama (Local AI Stack)

Run n8n and Ollama in the same Docker Compose stack so they can talk to each other:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    env_file:
      - .env
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - ollama
    # n8n starts after Ollama is running

  ollama:
    image: ollama/ollama
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
      # Expose Ollama API on your host machine too (optional)
      # Remove this if you only want n8n to access Ollama internally
    volumes:
      - ollama_data:/root/.ollama
      # Stores downloaded models — VERY important to persist
      # Models are 2-40 GB each; you don't want to re-download them every restart

    # ---- NVIDIA GPU support (uncomment if you have an NVIDIA GPU) ----
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: all
    #           capabilities: [gpu]
    # Prerequisite: Install NVIDIA Container Toolkit first:
    # https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html

volumes:
  n8n_data:
  ollama_data:
```

**After starting, pull a model:**
```bash
docker exec -it ollama ollama pull llama3.1:8b
# This downloads the model INTO the ollama_data volume
# It only needs to be done once per model
```

**In n8n Ollama Chat Model credential:**
```
Base URL: http://ollama:11434
```
Do NOT use `localhost` — inside Docker, `ollama` is the correct hostname (Docker's internal DNS).

---

## Part 6 — Cloud VPS: Production Setup with HTTPS

### Step 1 — Choose and Create Your VPS

**Recommended VPS providers and plans:**

| Provider | Plan | Price | Good For |
|----------|------|-------|----------|
| **Hetzner** | CX22 (2 vCPU, 4 GB RAM) | ~€4/month | Best value in Europe |
| **DigitalOcean** | Basic 2 GB Droplet | $12/month | Easy to use, good docs |
| **AWS EC2** | t3.small | ~$15/month | If you need AWS ecosystem |
| **Contabo** | VPS S | ~€5/month | Cheapest storage option |
| **Vultr** | 2 GB Cloud Compute | $12/month | Global locations |

**Minimum spec for n8n only:** 1 vCPU, 1 GB RAM (2 GB recommended)  
**With PostgreSQL:** 2 vCPU, 2 GB RAM  
**With Ollama 3B model:** 4 vCPU, 8 GB RAM  

**Create a Hetzner VPS (as an example):**
1. Sign up at hetzner.com
2. New Project → Add Server
3. Location: choose nearest to you
4. Image: **Ubuntu 22.04**
5. Type: CX22 (2 vCPU, 4 GB RAM)
6. SSH Keys: Add your public key (more secure than password)
7. Create & Buy → note your server IP

---

### Step 2 — Connect to Your VPS

```bash
# Mac/Linux terminal or Windows Terminal/PowerShell
ssh root@YOUR_SERVER_IP

# Example
ssh root@157.90.123.45
```

You'll see a welcome message. You're now inside your server.

---

### Step 3 — Secure the Server

```bash
# Update all packages first — critical for security
apt update && apt upgrade -y

# Create a non-root user (running as root is risky)
adduser n8n-user
# Type a password when prompted
# Press Enter for all other questions

# Give sudo access
usermod -aG sudo n8n-user

# Copy your SSH key to the new user
mkdir /home/n8n-user/.ssh
cp ~/.ssh/authorized_keys /home/n8n-user/.ssh/
chown -R n8n-user:n8n-user /home/n8n-user/.ssh
chmod 700 /home/n8n-user/.ssh
chmod 600 /home/n8n-user/.ssh/authorized_keys

# Disable root SSH login (prevents brute force on root)
nano /etc/ssh/sshd_config
# Find the line: PermitRootLogin yes
# Change to: PermitRootLogin no
# Save: Ctrl+X → Y → Enter

systemctl restart sshd

# Configure firewall
ufw allow 22      # SSH — ALWAYS allow this first or you'll lock yourself out
ufw allow 80      # HTTP (for Let's Encrypt certificate verification)
ufw allow 443     # HTTPS
# Note: We are NOT allowing 5678 directly — nginx will proxy traffic
ufw enable
# Type 'y' when asked

ufw status
# Should show: Status: active with rules for 22, 80, 443
```

**Reconnect as new user:**
```bash
exit
ssh n8n-user@YOUR_SERVER_IP
```

---

### Step 4 — Install Docker on the VPS

```bash
# Install Docker
curl -fsSL https://get.docker.com | sh

# Add your user to docker group (no sudo needed for docker commands)
sudo usermod -aG docker $USER
newgrp docker

# Install Docker Compose plugin
sudo apt-get install -y docker-compose-plugin

# Verify
docker --version
docker compose version
```

---

### Step 5 — Point Your Domain to the VPS

**Why you need a domain:** Let's Encrypt SSL certificates require a domain name. You cannot get a certificate for an IP address.

1. Buy a domain from Namecheap, Cloudflare Registrar, or any registrar
2. Go to your domain's DNS settings
3. Add an **A Record**:
   - Name: `n8n` (for subdomain `n8n.yourdomain.com`)
   - Value: Your VPS IP address
   - TTL: 3600 (or "Auto")
4. Wait 5–30 minutes for DNS to propagate
5. Verify: `ping n8n.yourdomain.com` — should reply from your VPS IP

---

### Step 6 — Create the Production Stack

```bash
mkdir ~/n8n-production && cd ~/n8n-production
mkdir nginx ssl
```

Create `.env`:
```env
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=VeryStrongPasswordHere!

# Generate: openssl rand -hex 32
N8N_ENCRYPTION_KEY=your-generated-32-char-hex-key

GENERIC_TIMEZONE=Asia/Kolkata
TZ=Asia/Kolkata

N8N_PROTOCOL=https
N8N_HOST=0.0.0.0
N8N_PORT=5678
WEBHOOK_URL=https://n8n.yourdomain.com

DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=AnotherStrongDBPassword!

EXECUTIONS_DATA_MAX_AGE=168
DOMAIN_NAME=n8n.yourdomain.com
SSL_EMAIL=your@email.com
```

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: ${DB_POSTGRESDB_PASSWORD}
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U n8n"]
      interval: 5s
      timeout: 5s
      retries: 5

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    expose:
      - "5678"
      # expose makes port available to other containers but NOT to the host
      # We use 'expose' instead of 'ports' because nginx will handle external traffic
    env_file:
      - .env
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      # nginx handles all external traffic on 80 and 443
      # and proxies it to n8n on port 5678 internally
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./ssl:/etc/letsencrypt
      - ./certbot-www:/var/www/certbot
    depends_on:
      - n8n

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./ssl:/etc/letsencrypt
      - ./certbot-www:/var/www/certbot
    # certbot is used one-time to get the SSL cert, then for renewal
    # We run it manually, not as a service
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"

volumes:
  postgres_data:
  n8n_data:
```

---

### Step 7 — Configure nginx

Create `nginx/n8n.conf`:

```nginx
# HTTP server — only handles Let's Encrypt verification, then redirects to HTTPS
server {
    listen 80;
    server_name n8n.yourdomain.com;

    # This location is needed for Let's Encrypt certificate verification
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # All other HTTP traffic redirects to HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS server — handles all production traffic
server {
    listen 443 ssl;
    server_name n8n.yourdomain.com;

    # SSL certificate files generated by certbot
    ssl_certificate /etc/letsencrypt/live/n8n.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/n8n.yourdomain.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;

    location / {
        proxy_pass http://n8n:5678;
        # Forward traffic to the n8n container on port 5678
        # 'n8n' is resolved by Docker's internal DNS to the n8n container

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        # Required for WebSocket support (n8n uses WebSockets for the editor)

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        # These pass the real client IP to n8n

        proxy_read_timeout 86400s;
        # 24 hours — needed for long-running workflows that hold the connection
        # Without this: nginx cuts the connection after 60 seconds
        # affecting long AI operations and approval workflows

        proxy_send_timeout 86400s;
        client_max_body_size 100M;
        # Maximum file upload size. Increase if you need to upload large files to n8n
    }
}
```

---

### Step 8 — Get SSL Certificate

```bash
# Start nginx first (needs to run for certificate verification)
docker compose up -d nginx

# Get the certificate
docker compose run --rm certbot certonly --webroot \
  -w /var/www/certbot \
  -d n8n.yourdomain.com \
  --email your@email.com \
  --agree-tos \
  --no-eff-email

# Expected output:
# Successfully received certificate.
# Certificate is saved at: /etc/letsencrypt/live/n8n.yourdomain.com/fullchain.pem

# Now start everything
docker compose up -d
```

Open: **https://n8n.yourdomain.com** ✅

---

### Step 9 — Set Up Automatic SSL Renewal

Let's Encrypt certificates expire every 90 days. The certbot service in the compose file handles renewal automatically. Add a cron job as a safety net:

```bash
crontab -e
# Add this line:
0 3 * * * cd ~/n8n-production && docker compose run --rm certbot renew && docker compose exec nginx nginx -s reload
# Runs at 3 AM daily, renews if needed, reloads nginx to pick up new cert
```

---

## Part 7 — Common Challenges & Solutions

### Connecting to Services

| Scenario | Problem | Solution |
|----------|---------|----------|
| n8n in Docker, Ollama on host (Windows/Mac) | `localhost` points to container, not host | Use `http://host.docker.internal:11434` |
| n8n in Docker, Ollama on host (Linux) | `host.docker.internal` not available | Add to compose: `extra_hosts: ["host.docker.internal:host-gateway"]` |
| n8n Cloud, Ollama on your laptop | n8n Cloud can't reach your laptop | Use ngrok: `ngrok http 11434` → use the HTTPS URL |
| Both in same Docker Compose | — | Use the service name: `http://ollama:11434` |

### Data & Storage

| Problem | Cause | Fix |
|---------|-------|-----|
| Workflows lost after container restart | No volume mounted | Add `-v n8n_data:/home/node/.n8n` to compose |
| Credentials show "cannot decrypt" | `N8N_ENCRYPTION_KEY` changed | Restore original key from backup |
| Database grows very large | Too much execution data | Enable pruning: `EXECUTIONS_DATA_MAX_AGE=72` |
| Running out of disk space on VPS | Docker images + logs | `docker system prune -a` removes unused resources |

### Webhooks

| Problem | Cause | Fix |
|---------|-------|-----|
| Webhook URL shows `http://localhost:5678` | `WEBHOOK_URL` not set | Set `WEBHOOK_URL=https://your-domain.com` |
| Test webhook works but production doesn't | Workflow not active | Click the toggle to activate the workflow |
| Stripe/GitHub can't reach webhook | VPN/NAT blocking | Move n8n to a public VPS or use ngrok |
| Duplicate events | Service retries on timeout | Set Response Mode: Immediately, handle idempotency |

### Scheduling

| Problem | Cause | Fix |
|---------|-------|-----|
| Schedule fires at wrong time | Timezone not set | Set `GENERIC_TIMEZONE=Asia/Kolkata` |
| Schedule doesn't fire after VPS reboot | n8n not set to restart | Add `restart: unless-stopped` to Docker compose |

---

## Part 8 — Backup & Recovery

### What to Back Up

| What | Why | How Often |
|------|-----|----------|
| `N8N_ENCRYPTION_KEY` | Losing this = losing all credentials permanently | Once (and store in password manager) |
| n8n_data Docker volume | Workflows, credentials, executions | Daily |
| .env file | All configuration | After every change |
| postgres_data volume (if using Postgres) | Workflow data if using Postgres DB | Daily |

### Backup Commands

```bash
# Backup n8n data volume to a .tar.gz file
docker run --rm \
  -v n8n-production_n8n_data:/source \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/n8n-$(date +%Y%m%d-%H%M).tar.gz -C /source .

# Backup Postgres database
docker exec postgres pg_dump -U n8n n8n > backups/n8n-db-$(date +%Y%m%d).sql

# Automated daily backup (add to crontab)
0 2 * * * cd ~/n8n-production && docker run --rm -v n8n-production_n8n_data:/source -v $(pwd)/backups:/backup alpine tar czf /backup/n8n-$(date +\%Y\%m\%d).tar.gz -C /source . && find $(pwd)/backups -name 'n8n-*.tar.gz' -mtime +7 -delete
# This: backs up daily at 2 AM and deletes backups older than 7 days
```

### Restore

```bash
# Stop n8n first
docker compose stop n8n

# Restore from backup
docker run --rm \
  -v n8n-production_n8n_data:/target \
  -v $(pwd)/backups:/backup \
  alpine tar xzf /backup/n8n-20260430.tar.gz -C /target

# Start n8n
docker compose start n8n
```

---

## Part 9 — Updating n8n

```bash
cd ~/n8n-production

# Pull new image
docker compose pull n8n

# Backup before updating (always)
docker run --rm -v n8n-production_n8n_data:/source -v $(pwd)/backups:/backup \
  alpine tar czf /backup/pre-update-$(date +%Y%m%d).tar.gz -C /source .

# Recreate n8n container with new image (data volume is preserved)
docker compose up -d n8n

# Verify new version running
docker exec -it n8n n8n --version

# Check logs for any migration issues
docker compose logs -f n8n
```

---

## Part 10 — Best Practices Summary

### Security
```
✅ Always enable N8N_BASIC_AUTH_ACTIVE on any non-localhost instance
✅ Use HTTPS — never HTTP for production
✅ Store N8N_ENCRYPTION_KEY in a password manager
✅ Set unique webhook secrets for each webhook
✅ Keep n8n updated — security patches ship in every release
✅ Restrict n8n to known IP ranges where possible
✅ Never export workflow JSON files that include credentials
```

### Performance
```
✅ Use PostgreSQL for production (not SQLite)
✅ Enable execution data pruning (EXECUTIONS_DATA_MAX_AGE=168)
✅ Use 'Only Save Errors' for execution data in stable workflows
✅ Add Retry On Fail (3 tries, 5s) to all external API nodes
✅ Add Wait nodes between rate-limited API calls
```

### Reliability
```
✅ Set restart: unless-stopped on all Docker containers
✅ Set GENERIC_TIMEZONE — never rely on server default
✅ Set WEBHOOK_URL explicitly — never let n8n guess
✅ Create a global Error Trigger workflow to alert on failures
✅ Back up the n8n_data volume daily
✅ Test restores — a backup you haven't tested isn't a backup
```

### Workflow Design
```
✅ Name every node descriptively (F2 to rename)
✅ Add notes to complex nodes (Settings tab)
✅ Use n8n Variables for reusable values (URLs, IDs)
✅ Pin test data to avoid re-calling APIs during development
✅ Use Execute Once for notification/alert nodes
✅ Test with 1 item, then 10, then activate for full data
```
