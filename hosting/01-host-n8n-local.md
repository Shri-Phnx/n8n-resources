# Hosting n8n Locally — Step-by-Step Guide

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Covers:** Windows, macOS, and Linux local installs (non-Docker). See `02-host-n8n-docker.md` for Docker/container installs.

---

## Which Install Method Should I Use?

```
Do you have Docker installed?
├── YES → Use Docker method (02-host-n8n-docker.md) — easier to manage
└── NO
    ├── Are you on Windows/Mac just testing? → npm install (this guide)
    ├── Are you on a Linux server? → npm or Docker (this guide + 02)
    └── Want a UI installer? → n8n Desktop (this guide)
```

---

## Method 1 — Windows: Install n8n via npm

### Prerequisites

**Step 1 — Install Node.js**
1. Go to [nodejs.org](https://nodejs.org)
2. Download the **LTS** version (e.g., v20.x)
3. Run the installer — accept all defaults
4. When asked about additional tools (Python, Visual C++): check the box — these are needed for some n8n dependencies
5. After install completes, **restart your computer**

**Verify Node.js installation:**
```
Open Command Prompt (Win + R → type cmd → Enter)
Type: node --version
Expected: v20.x.x

Type: npm --version
Expected: 10.x.x
```

**⚠️ If node command not found:** Node.js was not added to PATH. Reinstall Node.js and ensure "Add to PATH" option is checked.

---

### Step 2 — Install n8n

```cmd
npm install n8n -g
```

**This takes 2–5 minutes.** You'll see a progress bar. Some warnings (`WARN`) are normal — only `ERROR` messages are a problem.

**⚠️ Common Windows errors:**

| Error | Fix |
|-------|-----|
| `EACCES permission denied` | Run Command Prompt **as Administrator** (right-click → Run as admin) |
| `node-gyp rebuild failed` | Run as Admin AND ensure you checked "additional tools" during Node.js install |
| `Cannot find module` | Close cmd, reopen as Admin, run install again |
| `npm ERR! code ENOTFOUND` | Check internet connection / proxy settings |

---

### Step 3 — Start n8n

```cmd
n8n start
```

**Expected output:**
```
n8n ready on 0.0.0.0, port 5678
Version: 1.x.x
Editor is now accessible via:
http://localhost:5678
```

**Open in browser:** [http://localhost:5678](http://localhost:5678)

**First run:** n8n will ask you to create an account (name, email, password). This is a local account — no data is sent to n8n servers.

---

### Step 4 — Keep n8n Running in Background (Windows)

The `n8n start` command runs in the terminal — closing the terminal stops n8n.

**Option A — Use PM2 (recommended):**
```cmd
npm install pm2 -g
pm2 start n8n -- start
pm2 save
pm2 startup
```

Now n8n starts automatically when Windows boots.

**Option B — Quick testing only:**
Minimise the Command Prompt window — don't close it.

---

### Step 5 — Set Data Folder (Optional but Recommended)

By default, n8n stores data in `C:\Users\YourName\.n8n`. To customise:

```cmd
:: Set environment variable before starting
set N8N_USER_FOLDER=D:\n8n-data
n8n start
```

---

## Method 2 — macOS: Install n8n via Homebrew

### Prerequisites

**Step 1 — Install Homebrew (if not installed):**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Step 2 — Install Node.js via Homebrew:**
```bash
brew install node

# Verify
node --version   # v20.x.x
npm --version    # 10.x.x
```

**⚠️ Mac with Apple Silicon (M1/M2/M3):** Some npm packages need Rosetta:
```bash
softwareupdate --install-rosetta
```

---

### Step 3 — Install n8n

```bash
npm install n8n -g
```

**⚠️ EACCES permission error on Mac:**
```bash
# Fix: change npm's default directory
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
npm install n8n -g
```

---

### Step 4 — Start n8n

```bash
n8n start
```

Open: [http://localhost:5678](http://localhost:5678)

**Keep running with PM2:**
```bash
npm install pm2 -g
pm2 start n8n -- start
pm2 save
pm2 startup    # Follow the printed command to enable auto-start
```

---

## Method 3 — Linux Server: Install via npm

```bash
# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify
node --version
npm --version

# Install n8n globally
sudo npm install n8n -g

# Start n8n
n8n start
```

**Run as a system service (systemd):**
```bash
# Create service file
sudo nano /etc/systemd/system/n8n.service
```

Paste:
```ini
[Unit]
Description=n8n Workflow Automation
After=network.target

[Service]
Type=simple
User=ubuntu
Environment=N8N_BASIC_AUTH_ACTIVE=true
Environment=N8N_BASIC_AUTH_USER=admin
Environment=N8N_BASIC_AUTH_PASSWORD=your-secure-password
ExecStart=/usr/bin/n8n start
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Save (Ctrl+X → Y → Enter)

```bash
sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
sudo systemctl status n8n   # Should show: active (running)
```

---

## Environment Variables — Key Configuration

Set these before running `n8n start` to configure your instance.

| Variable | Default | What It Does | Example |
|----------|---------|-------------|--------|
| `N8N_BASIC_AUTH_ACTIVE` | false | Enable basic auth login | `true` |
| `N8N_BASIC_AUTH_USER` | — | Basic auth username | `admin` |
| `N8N_BASIC_AUTH_PASSWORD` | — | Basic auth password | `StrongPass123!` |
| `N8N_HOST` | `localhost` | Host n8n binds to | `0.0.0.0` (for external access) |
| `N8N_PORT` | `5678` | Port n8n runs on | `5678` |
| `N8N_PROTOCOL` | `http` | Protocol | `https` |
| `WEBHOOK_URL` | Auto | Public URL for webhooks | `https://n8n.yourdomain.com` |
| `N8N_ENCRYPTION_KEY` | Random | Key for encrypting credentials | Use a strong 32-char string |
| `DB_TYPE` | `sqlite` | Database type | `postgresdb` for production |
| `N8N_USER_FOLDER` | `~/.n8n` | Where data is stored | `/data/n8n` |

**Setting variables on Windows:**
```cmd
set N8N_PORT=5678
set N8N_BASIC_AUTH_ACTIVE=true
n8n start
```

**Setting on Mac/Linux:**
```bash
export N8N_PORT=5678
export N8N_BASIC_AUTH_ACTIVE=true
n8n start

# Or add to ~/.zshrc or ~/.bashrc for persistence
```

---

## Accessing n8n from Another Device on Your LAN

By default n8n only accepts connections from `localhost`. To access from another device:

```bash
export N8N_HOST=0.0.0.0
n8n start
```

Then open `http://YOUR_MACHINE_IP:5678` from another device.

**Find your machine's IP:**
- Windows: `ipconfig` → look for IPv4 Address
- Mac: `ipconfig getifaddr en0`
- Linux: `hostname -I | awk '{print $1}'`

⚠️ Only expose n8n on your LAN, not the public internet, without setting up authentication and HTTPS.

---

## Updating n8n

```bash
# Stop n8n first (if using PM2)
pm2 stop n8n

# Update
npm update n8n -g

# Or install specific version
npm install n8n@1.x.x -g

# Restart
pm2 start n8n
```

⚠️ Always backup your `~/.n8n` folder before updating:
```bash
cp -r ~/.n8n ~/.n8n-backup-$(date +%Y%m%d)
```
