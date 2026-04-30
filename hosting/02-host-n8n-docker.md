# Hosting n8n with Docker — Step-by-Step Guide

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Covers:** Docker single-container, Docker Compose, and cloud VPS deployment.

---

## Why Docker for n8n?

| Advantage | Detail |
|-----------|--------|
| **Isolation** | n8n runs in its own container — won't conflict with other software |
| **Portability** | Same setup works on Windows, Mac, Linux, cloud VPS |
| **Easy updates** | Pull new image, restart container |
| **Data persistence** | Volumes keep your workflows even when container restarts |
| **Reproducible** | docker-compose.yml captures your entire setup |

---

## Prerequisites — Install Docker

### Windows
1. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Download **Docker Desktop for Windows**
3. Run installer — accept defaults
4. After install: Docker Desktop opens → wait for the whale icon to appear in the taskbar and turn green
5. Verify:
```cmd
docker --version
# Docker version 26.x.x

docker run hello-world
# Should print: Hello from Docker!
```

**⚠️ Windows WSL2 required:** Docker Desktop requires WSL2. If prompted, follow the link to install WSL2 — it takes 2–3 minutes.

**⚠️ Hyper-V must be enabled:** If Docker fails to start, check that Hyper-V is enabled in Windows Features.

### macOS
1. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Download for Mac (choose **Apple Silicon** for M1/M2/M3 or **Intel** for older Macs)
3. Drag Docker to Applications → open it
4. Allow Docker to install helper tools when prompted
5. Wait for Docker menu bar icon to stop animating (fully started)

```bash
docker --version
docker run hello-world
```

### Linux (Ubuntu/Debian)
```bash
# Install Docker Engine
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker

# Install Docker Compose
sudo apt-get install docker-compose-plugin

# Verify
docker --version
docker compose version
```

---

## Method 1 — Quick Single Container (Testing)

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

**What each part means:**
```
docker run          → Start a new container
-it                 → Interactive terminal (so you can see logs)
--rm                → Remove container when it stops
--name n8n          → Give the container a name
-p 5678:5678        → Map port 5678 on your machine to port 5678 in container
-v ~/.n8n:/home/...  → Mount local folder to persist data
docker.n8n.io/...   → The n8n Docker image
```

Open: [http://localhost:5678](http://localhost:5678)

**⚠️ This is for testing only.** `--rm` means all data is lost when you stop the container. For persistent use, use Method 2.

---

## Method 2 — Docker Compose (Recommended for Local/VPS)

### Step 1 — Create Project Folder

```bash
# Mac/Linux
mkdir ~/n8n && cd ~/n8n

# Windows (Command Prompt)
mkdir C:\n8n && cd C:\n8n
```

### Step 2 — Create docker-compose.yml

Create a file called `docker-compose.yml`:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=change-this-password
      - N8N_ENCRYPTION_KEY=change-this-32-char-key-xxxxxxxx
      - WEBHOOK_URL=http://localhost:5678
      - GENERIC_TIMEZONE=Asia/Kolkata
      - TZ=Asia/Kolkata
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
    driver: local
```

**⚠️ Change these values before starting:**
- `N8N_BASIC_AUTH_PASSWORD` → Set a strong password
- `N8N_ENCRYPTION_KEY` → Set a random 32-character string (generate at [randomkeygen.com](https://randomkeygen.com))
- `GENERIC_TIMEZONE` → Your timezone (e.g., `Asia/Dubai` for UAE, `Asia/Kolkata` for India)

### Step 3 — Start n8n

```bash
# Start in background (detached mode)
docker compose up -d

# Check status
docker compose ps
# Should show: n8n   running   0.0.0.0:5678->5678/tcp

# View logs
docker compose logs n8n

# Follow logs in real-time
docker compose logs -f n8n
```

Open: [http://localhost:5678](http://localhost:5678)

---

## Method 3 — n8n + PostgreSQL (Production Setup)

SQLite is fine for development, but PostgreSQL is recommended for production (better performance, backup support, concurrent access).

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: n8n-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n-db-password
      - POSTGRES_DB=n8n
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
    ports:
      - "5678:5678"
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=change-this-password
      - N8N_ENCRYPTION_KEY=change-this-32-char-key-xxxxxxxx
      - WEBHOOK_URL=https://your-domain.com
      - GENERIC_TIMEZONE=Asia/Kolkata
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n-db-password
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  postgres_data:
  n8n_data:
```

---

## Method 4 — n8n + Ollama (Local AI, Both in Docker)

Run both n8n and Ollama in the same Docker Compose stack:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=change-this
      - N8N_ENCRYPTION_KEY=change-this-32char-key-xxxxxxxx
      - GENERIC_TIMEZONE=Asia/Kolkata
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - ollama

  ollama:
    image: ollama/ollama
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    # Uncomment below if you have NVIDIA GPU:
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: all
    #           capabilities: [gpu]

volumes:
  n8n_data:
  ollama_data:
```

**Pull a model after starting:**
```bash
docker exec -it ollama ollama pull llama3.1:8b
```

**In n8n Ollama credential → Base URL:**
```
http://ollama:11434
```
(Use the Docker service name `ollama` — Docker's internal DNS resolves it)

---

## Common Docker Commands for n8n

```bash
# Start n8n
docker compose up -d

# Stop n8n
docker compose down

# Restart n8n
docker compose restart n8n

# Update to latest n8n version
docker compose pull
docker compose up -d

# View real-time logs
docker compose logs -f n8n

# Access n8n container shell (for debugging)
docker exec -it n8n sh

# Backup n8n data volume
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/n8n-backup-$(date +%Y%m%d).tar.gz /data

# Restore n8n data
docker run --rm -v n8n_data:/data -v $(pwd):/backup alpine \
  tar xzf /backup/n8n-backup-YYYYMMDD.tar.gz -C /
```

---

## Setting Up HTTPS with Nginx Reverse Proxy

For production or public access, always use HTTPS.

### Step 1 — Add nginx to docker-compose.yml

```yaml
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
```

### Step 2 — nginx configuration file

Create `nginx/conf.d/n8n.conf`:
```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    location / {
        proxy_pass http://n8n:5678;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 86400s;
    }
}
```

### Step 3 — Get SSL certificate
```bash
docker run --rm -v $(pwd)/certbot/conf:/etc/letsencrypt \
  -v $(pwd)/certbot/www:/var/www/certbot \
  certbot/certbot certonly --webroot \
  -w /var/www/certbot \
  -d your-domain.com \
  --email your@email.com \
  --agree-tos
```

Update `WEBHOOK_URL` in docker-compose.yml to `https://your-domain.com`.

---

## Updating n8n in Docker

```bash
# Pull latest image
docker compose pull n8n

# Restart with new image (data is preserved in volume)
docker compose up -d n8n

# Verify new version
docker exec -it n8n n8n --version
```

⚠️ **Before every major update:** Backup your data volume.

---

## Troubleshooting Docker n8n

| Problem | Cause | Fix |
|---------|-------|-----|
| Port 5678 already in use | Another app using the port | Change `-p 5679:5678` to use port 5679 |
| n8n container keeps restarting | Config error | Run `docker compose logs n8n` to see the error |
| Cannot connect to Ollama | Docker network issue | Use `http://ollama:11434` not `localhost` |
| Workflows lost after restart | Volume not configured | Ensure `-v n8n_data:/home/node/.n8n` is in compose file |
| Credentials not working after move | Encryption key changed | Restore original `N8N_ENCRYPTION_KEY` |
| Webhook URL wrong | `WEBHOOK_URL` not set | Set `WEBHOOK_URL=https://your-domain.com` in environment |
| Timezone wrong on schedules | TZ not set | Add `GENERIC_TIMEZONE=Asia/Kolkata` and `TZ=Asia/Kolkata` |
