# n8n Database Guide — Complete Reference

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Source:** [n8n Official Docs](https://docs.n8n.io/hosting/configuration/supported-databases-settings/)

---

## The Critical Distinction — Two Types of "Database" in n8n

Before anything else, understand that "database" means two different things in n8n context:

| Type | What It Is | Who It's For |
|------|-----------|-------------|
| **n8n's backend database** | Where n8n stores its OWN data — workflows, credentials, execution logs, users | You (the person hosting n8n) configure this once |
| **Databases in workflows** | External databases your workflows READ FROM or WRITE TO as part of automation logic | You use n8n nodes (Postgres, MySQL, MongoDB, etc.) inside workflows |

This guide covers BOTH. Most documentation only covers the first.

---

## Part 1 — n8n Backend Database Options

### What n8n Officially Supports (April 2026)

| Database | Status | When to Use |
|----------|--------|-------------|
| **SQLite** | ✅ Default — built-in | Development, testing, personal single-user use |
| **PostgreSQL** | ✅ Recommended for production | Any serious, multi-user, or high-volume deployment |
| **MySQL / MariaDB** | ❌ **Deprecated since n8n v1.0 — not supported** | Do not use. Was removed in n8n 1.0 |

> ⚠️ If you are reading old tutorials that mention MySQL for n8n's backend database, those are outdated. MySQL support was officially deprecated and removed in n8n v1.0.

---

### SQLite — The Default

**What it is:** A file-based database. No server needed. Everything lives in a single file: `~/.n8n/database.sqlite`

**No configuration needed** — n8n creates it automatically on first start.

| Advantage | Limitation |
|-----------|------------|
| Zero setup | Single file-level write lock — concurrent writes can corrupt data |
| Works offline, no dependencies | Cannot run Queue Mode (multiple workers) |
| Easy to backup (just copy the file) | Unsafe to backup while n8n is running — must stop n8n first |
| Fine for < 50 executions/day | Degrades significantly with high execution volume |
| Already included in n8n | No hot backups |

**When SQLite is fine:**
- You are learning n8n
- Personal automation with 1–5 workflows
- Testing before deploying to production
- Running on a laptop that's not always on

**When you MUST switch to PostgreSQL:**
- Running n8n on a public server
- Multiple team members using the same instance
- High-frequency scheduled workflows (many per hour)
- Using Queue Mode with workers
- Need reliable hot backups
- Any workflow handling sensitive production data

---

### PostgreSQL — Production Standard

**What it is:** A full client-server relational database. Runs as a separate service. n8n connects to it over TCP.

**Minimum version required:** PostgreSQL 13  
**Recommended version:** PostgreSQL 15 or 16

| Advantage | Note |
|-----------|------|
| Row-level locking — parallel writes safe | Requires a separate running service |
| Hot backups with pg_dump while n8n runs | Uses slightly more RAM (~50–100 MB baseline) |
| Required for Queue Mode | Need to create DB + user before n8n starts |
| Handles thousands of executions/minute | |
| Proper ACID transactions | |
| Easy monitoring and maintenance | |

---

## Part 2 — PostgreSQL: Complete Setup Guide

### Option A — PostgreSQL via Docker (Simplest)

If you already use Docker for n8n, this is the easiest path.

#### Step 1 — Add PostgreSQL to your docker-compose.yml

```yaml
version: '3.8'

services:
  # ─────────────────────────────────────────
  # PostgreSQL — the database server
  # ─────────────────────────────────────────
  postgres:
    image: postgres:16
    # Use postgres:16 or postgres:15 — both are stable.
    # Avoid :latest — versions can change and break compatibility.

    container_name: n8n-postgres
    restart: unless-stopped

    environment:
      # These three variables are read by the postgres image on FIRST START ONLY.
      # They create the database, user, and password.
      # If you change them after the volume exists, they have NO effect.
      # To change them after first start, you must use psql commands (see Part 3).

      POSTGRES_DB: n8n
      # The name of the database to create.
      # n8n will use this database to store all its tables.

      POSTGRES_USER: n8n
      # The database user that n8n will connect with.
      # This user gets full ownership of the n8n database.

      POSTGRES_PASSWORD: ${DB_POSTGRESDB_PASSWORD}
      # NEVER hardcode passwords in docker-compose.yml.
      # Load from .env file using this syntax.
      # Generate a strong password: openssl rand -base64 24

    volumes:
      - postgres_data:/var/lib/postgresql/data
      # This volume stores the actual database files.
      # CRITICAL: If you delete this volume, you lose ALL data permanently.
      # Never run 'docker volume prune' without knowing what you're deleting.

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U n8n -d n8n"]
      # pg_isready checks if Postgres is ready to accept connections.
      # -U n8n: check for our specific user
      # -d n8n: on our specific database
      interval: 5s    # Check every 5 seconds
      timeout: 5s     # Give up if no response within 5 seconds
      retries: 5      # Mark as unhealthy after 5 consecutive failures
      start_period: 10s  # Don't start checking for 10s (gives Postgres time to init)

    # OPTIONAL: Expose postgres port to your HOST machine
    # Only uncomment if you want to connect with a tool like DBeaver or pgAdmin
    # from outside Docker. Leave commented for security in production.
    # ports:
    #   - "5432:5432"
    # ⚠️ WARNING: If you expose this port, ensure a firewall blocks external access.
    # Only localhost should be able to reach port 5432.

    networks:
      - n8n-network

  # ─────────────────────────────────────────
  # n8n — the automation engine
  # ─────────────────────────────────────────
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
        # This is critical.
        # Without 'condition: service_healthy', n8n starts before Postgres
        # is ready and crashes immediately with a connection error.
        # With it: n8n waits for Postgres to pass its healthcheck first.
    networks:
      - n8n-network

volumes:
  postgres_data:
    # Docker-managed named volume for PostgreSQL data
  n8n_data:
    # Docker-managed named volume for n8n config/encryption key

networks:
  n8n-network:
    driver: bridge
    # A private Docker network shared between n8n and postgres.
    # Containers on this network can reach each other by service name.
    # postgres container is reachable at hostname 'postgres' from n8n.
```

#### Step 2 — Add PostgreSQL variables to your .env file

```env
# ─── n8n Auth & Encryption ───
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=YourN8NPassword!
N8N_ENCRYPTION_KEY=generate-with-openssl-rand-hex-32

# ─── Timezone ───
GENERIC_TIMEZONE=Asia/Kolkata
TZ=Asia/Kolkata

# ─── Webhook ───
WEBHOOK_URL=https://n8n.yourdomain.com

# ─── Database: PostgreSQL ───
DB_TYPE=postgresdb
# Tells n8n to use PostgreSQL instead of SQLite.

DB_POSTGRESDB_HOST=postgres
# The hostname of the PostgreSQL server.
# When Postgres is in the same Docker Compose stack: use the service name 'postgres'
# When Postgres is on a different machine: use that machine's IP or hostname
# When using Supabase: use your Supabase project URL

DB_POSTGRESDB_PORT=5432
# Default PostgreSQL port. Only change if your Postgres runs on a non-standard port.

DB_POSTGRESDB_DATABASE=n8n
# The name of the database. Must match POSTGRES_DB in the postgres service.

DB_POSTGRESDB_USER=n8n
# The database user. Must match POSTGRES_USER in the postgres service.

DB_POSTGRESDB_PASSWORD=GenerateAStrongPasswordHere!
# MUST match POSTGRES_PASSWORD in the postgres service.
# Generate: openssl rand -base64 24

DB_POSTGRESDB_SCHEMA=public
# The PostgreSQL schema to use.
# 'public' is the default schema.
# If you run multiple n8n instances on one database:
# use different schemas: n8n_prod, n8n_staging

# ─── Optional PostgreSQL SSL (for cloud-managed databases) ───
# DB_POSTGRESDB_SSL_ENABLED=true
# DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=true
# DB_POSTGRESDB_SSL_CA=/path/to/ca-certificate.crt
# DB_POSTGRESDB_SSL_CERT=/path/to/client-cert.crt
# DB_POSTGRESDB_SSL_KEY=/path/to/client-key.key
```

#### Step 3 — Start the Stack

```bash
docker compose up -d

# Watch startup (important — check for errors)
docker compose logs -f

# What healthy startup looks like:
# postgres  | database system is ready to accept connections
# n8n       | n8n ready on 0.0.0.0, port 5678
```

**⚠️ Common startup errors:**

| Error | Cause | Fix |
|-------|-------|-----|
| `password authentication failed for user "n8n"` | DB_POSTGRESDB_PASSWORD doesn't match POSTGRES_PASSWORD | Make sure both values in .env are identical |
| `database "n8n" does not exist` | Volume exists but database wasn't created | Delete volume (`docker volume rm postgres_data`) and restart — it'll recreate. ONLY do this on first setup |
| `n8n exited before postgres was ready` | depends_on not set to `service_healthy` | Add the healthcheck condition shown above |
| `could not connect to server: Connection refused` | n8n and postgres on different networks | Ensure both services are on the same Docker network |

---

### Option B — PostgreSQL Installed Directly on Linux (No Docker)

Use this when you want PostgreSQL to run directly on your Linux server without Docker.

#### Step 1 — Install PostgreSQL

```bash
# Ubuntu 22.04 / 24.04
sudo apt update
sudo apt install -y postgresql postgresql-contrib

# RHEL / CentOS 8+
sudo dnf install -y postgresql-server postgresql-contrib
sudo postgresql-setup --initdb

# Start and enable PostgreSQL service
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql
# Should show: active (running)
```

#### Step 2 — Create the n8n Database and User

```bash
# Switch to the postgres system user (the default superuser)
sudo -u postgres psql
# You are now in the PostgreSQL interactive shell (prompt shows: postgres=#)
```

Run these SQL commands inside psql:

```sql
-- Create a dedicated user for n8n
-- Replace 'your-strong-password' with a real password
CREATE USER n8n WITH PASSWORD 'your-strong-password';

-- Create the database owned by this user
CREATE DATABASE n8n OWNER n8n;

-- Grant all privileges on the database to the n8n user
GRANT ALL PRIVILEGES ON DATABASE n8n TO n8n;

-- Connect to the n8n database to set schema permissions
\c n8n

-- Grant usage and create permissions on the public schema
-- n8n needs to create its own tables on first run
GRANT ALL ON SCHEMA public TO n8n;

-- Verify: list databases
\l

-- Should show 'n8n' in the list with owner 'n8n'

-- Exit psql
\q
```

#### Step 3 — Configure n8n to Use the New Database

Add to your `.env` or systemd service environment:

```env
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
# 'localhost' works when n8n and postgres are on the SAME machine
# Use '127.0.0.1' if 'localhost' causes issues (IPv4 vs IPv6 resolution)

DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=your-strong-password
DB_POSTGRESDB_SCHEMA=public
```

#### Step 4 — Configure PostgreSQL to Accept Connections (if needed)

If n8n is on a DIFFERENT machine from PostgreSQL:

```bash
# Edit postgresql.conf
sudo nano /etc/postgresql/16/main/postgresql.conf
# Find: #listen_addresses = 'localhost'
# Change to: listen_addresses = '*'  (or specific IP)

# Edit pg_hba.conf (host-based authentication)
sudo nano /etc/postgresql/16/main/pg_hba.conf
# Add this line at the end:
# host  n8n  n8n  10.0.0.0/8  scram-sha-256
# Meaning: allow user 'n8n' to connect to db 'n8n' from 10.x.x.x range using password auth

# Restart PostgreSQL
sudo systemctl restart postgresql
```

---

### Option C — Using Supabase as PostgreSQL Backend (Easiest Cloud Option)

Supabase is a hosted PostgreSQL service with a beautiful UI. Free tier is generous enough for n8n.

**Why Supabase for n8n backend:**
- Free tier: 500 MB database, pauses after 1 week inactive (upgrade to Pro to avoid)
- No server to maintain
- Built-in web UI to browse tables
- Automatic backups
- SSL enabled by default

**Setup:**
1. Sign up at [supabase.com](https://supabase.com)
2. Create a new project — choose a region close to your n8n server
3. Set a database password (save it)
4. Go to **Settings → Database** → copy the connection string
5. Extract the host, port, database, user, and password from the connection string

```env
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=db.xxxxxxxxxxxx.supabase.co
# Your project's database host from Supabase Settings → Database

DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=postgres
# Supabase always names the database 'postgres'

DB_POSTGRESDB_USER=postgres
# Default Supabase user

DB_POSTGRESDB_PASSWORD=your-supabase-db-password
DB_POSTGRESDB_SCHEMA=public

# SSL is required for Supabase connections
DB_POSTGRESDB_SSL_ENABLED=true
DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=true
```

> ⚠️ Supabase free tier pauses the database after 1 week of inactivity. Upgrade to Pro ($25/month) for always-on. For a hobby n8n instance, the free tier works if you use it regularly.

---

## Part 3 — PostgreSQL Maintenance

### Daily / Weekly Tasks

#### Check Database Size
```sql
-- Connect to the n8n database
\c n8n

-- Show database size
SELECT pg_size_pretty(pg_database_size('n8n'));

-- Show table sizes (find what's taking up space)
SELECT
  table_name,
  pg_size_pretty(pg_total_relation_size(quote_ident(table_name))) AS size
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY pg_total_relation_size(quote_ident(table_name)) DESC;
```

#### Check Which Tables Are Largest in n8n

n8n's biggest tables are usually:

| Table | Contains | Gets Large When |
|-------|---------|----------------|
| `execution_entity` | All workflow execution records | Many executions, no pruning |
| `execution_data` | Input/output data for each execution | Large payloads |
| `workflow_entity` | Your workflow definitions | Not usually large |
| `credentials_entity` | Encrypted credentials | Not usually large |

#### Prune Old Execution Data

Do this from inside n8n:
**Settings → n8n → Execution Data Pruning → Enable**

Or via SQL (last resort, only if n8n pruning isn't working):
```sql
-- Delete executions older than 30 days
-- ALWAYS TEST with a SELECT first before deleting
SELECT COUNT(*) FROM execution_entity
WHERE "startedAt" < NOW() - INTERVAL '30 days';

-- If count looks right, delete
DELETE FROM execution_data
WHERE "executionId" IN (
  SELECT id FROM execution_entity
  WHERE "startedAt" < NOW() - INTERVAL '30 days'
);

DELETE FROM execution_entity
WHERE "startedAt" < NOW() - INTERVAL '30 days';
```

#### Run VACUUM ANALYZE (Reclaim Space)

After deleting many rows, PostgreSQL doesn't immediately free disk space. VACUUM reclaims it.

```sql
-- Safe VACUUM (n8n can keep running)
VACUUM ANALYZE;

-- VACUUM FULL (reclaims more space but locks tables — do during low traffic)
-- Only needed if database size hasn't reduced after regular VACUUM
VACUUM FULL execution_entity;
VACUUM FULL execution_data;
```

---

### Backup & Restore

#### Backup with pg_dump (Hot Backup — n8n Can Keep Running)

```bash
# Docker setup
docker exec -t n8n-postgres pg_dump -U n8n -d n8n > backup-$(date +%Y%m%d-%H%M).sql

# Direct install
pg_dump -U n8n -h localhost -d n8n > backup-$(date +%Y%m%d-%H%M).sql

# Compressed backup (much smaller file)
pg_dump -U n8n -d n8n | gzip > backup-$(date +%Y%m%d-%H%M).sql.gz
```

#### Automated Daily Backup (Cron)

```bash
crontab -e

# Add:
0 2 * * * docker exec n8n-postgres pg_dump -U n8n -d n8n | gzip > ~/backups/n8n-$(date +\%Y\%m\%d).sql.gz && find ~/backups -name 'n8n-*.sql.gz' -mtime +14 -delete
# Runs at 2 AM daily, keeps 14 days of backups
```

#### Restore from Backup

```bash
# Stop n8n first
docker compose stop n8n

# Drop and recreate the database (clean slate restore)
docker exec -it n8n-postgres psql -U postgres
```
```sql
DROP DATABASE n8n;
CREATE DATABASE n8n OWNER n8n;
\q
```
```bash
# Restore
cat backup-20260430.sql | docker exec -i n8n-postgres psql -U n8n -d n8n

# Or from compressed:
gunzip -c backup-20260430.sql.gz | docker exec -i n8n-postgres psql -U n8n -d n8n

# Restart n8n
docker compose start n8n
```

---

### Migration: SQLite → PostgreSQL

When you're ready to move from SQLite to PostgreSQL:

**⚠️ Important:** SQLite and PostgreSQL schemas differ. Direct data migration is complex. The cleanest approach:

#### Method 1 — Export Workflows and Start Fresh (Recommended)

```bash
# While still on SQLite, export everything from n8n

# Export all workflows
docker exec -it n8n n8n export:workflow --all --output=/home/node/.n8n/workflows-backup

# Export all credentials (encrypted — you'll need your encryption key)
docker exec -it n8n n8n export:credentials --all --output=/home/node/.n8n/credentials-backup

# Copy exports to your host
docker cp n8n:/home/node/.n8n/workflows-backup ./n8n-exports/
docker cp n8n:/home/node/.n8n/credentials-backup ./n8n-exports/
```

Then:
1. Update your `.env` with PostgreSQL settings
2. `docker compose down && docker compose up -d`
3. n8n starts fresh on PostgreSQL
4. Import workflows: Settings → Import from file (or n8n import CLI)

#### Method 2 — n8n CLI Migration (preserves execution history)

```bash
# 1. Set up PostgreSQL DB first (empty)
# 2. Keep SQLite as current DB_TYPE
# 3. Run migration command:
docker exec -it n8n n8n db:revert
# Then update DB_TYPE and connection in .env, restart
```

> **Honest assessment:** For most people, Method 1 (export/import) is cleaner and more reliable than trying to migrate the raw data. Execution history is usually not worth migrating.

---

### Useful PostgreSQL Commands (Quick Reference)

```bash
# Connect to PostgreSQL (Docker)
docker exec -it n8n-postgres psql -U n8n -d n8n

# Connect to PostgreSQL (Direct install)
psql -U n8n -d n8n

# As postgres superuser
sudo -u postgres psql
```

```sql
-- List all databases
\l

-- Connect to a database
\c n8n

-- List all tables
\dt

-- Describe a table structure
\d execution_entity

-- Show active connections to n8n database
SELECT pid, usename, application_name, client_addr, state
FROM pg_stat_activity
WHERE datname = 'n8n';

-- Kill a stuck connection
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE pid = 12345;  -- replace with actual pid

-- Check PostgreSQL version
SELECT version();

-- Show database size
SELECT pg_size_pretty(pg_database_size('n8n'));

-- Exit psql
\q
```

---

## Part 4 — Databases for Use INSIDE n8n Workflows

These are databases your n8n workflows can read from and write to as part of automation logic. This is a completely separate concept from n8n's backend database.

### Open-Source Databases with n8n Nodes

| Database | n8n Node | Type | Free? | Ease of Setup | Best For |
|----------|----------|------|-------|---------------|----------|
| **PostgreSQL** | ✅ Built-in | Relational SQL | ✅ Open source | Medium | Structured data, reporting, complex queries |
| **MySQL / MariaDB** | ✅ Built-in | Relational SQL | ✅ Open source | Medium | Classic web apps, widely hosted |
| **MongoDB** | ✅ Built-in | Document (JSON) | ✅ Open source | Easy (Atlas free tier) | Flexible schema, nested data |
| **Redis** | ✅ Built-in | Key-Value / Cache | ✅ Open source | Easy | Fast lookups, pub/sub, session data |
| **Supabase** | ✅ Built-in | PostgreSQL + extras | ✅ Free tier | Very Easy | Modern apps, auth, realtime |
| **NocoDB** | ✅ Built-in | No-code DB (Airtable alt) | ✅ Open source | Easy | Spreadsheet-style DB without code |
| **Baserow** | ✅ Built-in | No-code DB | ✅ Open source | Easy | Airtable replacement |
| **SQLite** | Via Code node | File-based SQL | ✅ Open source | Very Easy | Local file processing, small datasets |

---

## Part 5 — Other Open-Source Databases: Comparison

### 1. SQLite

| Attribute | Detail |
|-----------|--------|
| **Type** | File-based relational database |
| **Install** | Zero — no server, no install, included in most languages |
| **Best for** | Prototyping, local apps, file-based data stores |
| **n8n backend?** | ✅ Yes — it's the default |
| **n8n workflow target?** | Via Code node (using better-sqlite3) |
| **Limitation** | No concurrent write support; not suitable for multi-user production |

---

### 2. MySQL / MariaDB

| Attribute | Detail |
|-----------|--------|
| **Type** | Relational SQL |
| **Install** | `sudo apt install mysql-server` or Docker: `mysql:8.0` |
| **Best for** | Web applications, WordPress, Drupal, well-documented |
| **n8n backend?** | ❌ Deprecated in n8n v1.0 — cannot be used as n8n's backend |
| **n8n workflow target?** | ✅ Yes — MySQL node is built-in |
| **Free?** | ✅ MySQL Community Edition is free; MariaDB is fully open source |
| **Limitation** | More strict than PostgreSQL on some SQL features; less feature-rich than Postgres |

**Docker quick start:**
```yaml
mysql:
  image: mysql:8.0
  environment:
    MYSQL_ROOT_PASSWORD: rootpassword
    MYSQL_DATABASE: mydb
    MYSQL_USER: myuser
    MYSQL_PASSWORD: mypassword
  volumes:
    - mysql_data:/var/lib/mysql
```

---

### 3. MongoDB

| Attribute | Detail |
|-----------|--------|
| **Type** | Document database (JSON/BSON) |
| **Install** | Docker: `mongo:7.0` or MongoDB Atlas (free cloud tier) |
| **Best for** | Flexible/nested data, event logs, content that doesn't fit rigid tables |
| **n8n backend?** | ❌ Not supported as n8n's internal database |
| **n8n workflow target?** | ✅ Yes — MongoDB node built-in |
| **Free?** | ✅ MongoDB Community is free; Atlas free tier: 512 MB |
| **Limitation** | No joins (use $lookup); eventual consistency in clusters; BSON storage overhead |

**Docker quick start:**
```yaml
mongodb:
  image: mongo:7.0
  environment:
    MONGO_INITDB_ROOT_USERNAME: admin
    MONGO_INITDB_ROOT_PASSWORD: password
    MONGO_INITDB_DATABASE: mydb
  volumes:
    - mongo_data:/data/db
```

**n8n MongoDB node example — find documents:**
```
MongoDB node:
  Operation: Find
  Collection: users
  Query: { "status": "active", "createdAt": { "$gte": "{{ $now.minus({days: 7}).toISO() }}" } }
  Limit: 100
```

---

### 4. Redis

| Attribute | Detail |
|-----------|--------|
| **Type** | In-memory key-value store |
| **Install** | Docker: `redis:7-alpine` |
| **Best for** | Caching, session storage, pub/sub messaging, rate limiting, n8n Queue Mode, chat memory |
| **n8n backend?** | ❌ Not a backend DB, but required for Queue Mode |
| **n8n workflow target?** | ✅ Yes — Redis node built-in |
| **Free?** | ✅ Open source |
| **Limitation** | Data is in memory (volatile by default); persistence needs configuration |

**Docker quick start:**
```yaml
redis:
  image: redis:7-alpine
  command: redis-server --appendonly yes
  # --appendonly yes: enables AOF persistence so data survives restarts
  volumes:
    - redis_data:/data
```

**n8n Redis node example — set a cache value:**
```
Redis node:
  Operation: Set
  Key: processed:{{ $json.id }}
  Value: {{ $now.toISO() }}
  Expire: 86400  (24 hours in seconds)
```

---

### 5. Supabase (Hosted PostgreSQL)

| Attribute | Detail |
|-----------|--------|
| **Type** | Managed PostgreSQL + REST API + Auth + Realtime |
| **Install** | No install — sign up at supabase.com. Self-host option also available |
| **Best for** | When you want PostgreSQL power without managing a server |
| **n8n backend?** | ✅ Yes — use with DB_TYPE=postgresdb (see Part 2, Option C) |
| **n8n workflow target?** | ✅ Yes — dedicated Supabase node + PostgreSQL node |
| **Free?** | ✅ Generous free tier (500 MB, 2 projects) |
| **Limitation** | Free tier pauses after 1 week inactivity; not open source in core (based on PG which is) |

**Why Supabase is great for n8n workflows:**
- Row-level security (fine-grained access control)
- Realtime subscriptions (n8n can trigger on DB changes)
- Auto-generated REST and GraphQL API
- Built-in file storage
- Auth (users, sessions)

---

### 6. NocoDB

| Attribute | Detail |
|-----------|--------|
| **Type** | No-code database / Airtable open-source alternative |
| **Install** | Docker: `nocodb/nocodb:latest` |
| **Best for** | Non-technical team members who need a spreadsheet-style database |
| **n8n backend?** | ❌ Not supported |
| **n8n workflow target?** | ✅ Yes — NocoDB node built-in |
| **Free?** | ✅ Open source |
| **Limitation** | Not suitable for complex queries; designed for human interaction, not raw SQL |

```yaml
nocodb:
  image: nocodb/nocodb:latest
  ports:
    - "8080:8080"
  volumes:
    - nocodb_data:/usr/app/data
  environment:
    NC_DB: "pg://postgres:5432?u=nocodb&p=password&d=nocodb"
    # NocoDB itself uses Postgres or SQLite as its backend
```

---

## Part 6 — Decision Guide: Which Database to Use?

### For n8n's Internal Database

```
Are you testing / learning?
└── YES → SQLite (zero setup, default)

Are you running in production?
└── YES
    ├── Do you want full control? → PostgreSQL (Docker or direct install)
    ├── Want managed/no maintenance? → Supabase free tier
    └── Using Queue Mode (multiple workers)? → PostgreSQL is REQUIRED
```

### For Databases Used Inside Workflows

```
What kind of data?
│
├── Structured, relational (tables, joins, reports)
│   └── PostgreSQL or MySQL
│
├── Flexible / nested / JSON documents
│   └── MongoDB or Supabase (jsonb columns)
│
├── Key-value pairs, caching, fast lookups
│   └── Redis
│
├── Spreadsheet-style, non-technical team
│   └── NocoDB, Baserow, or Airtable
│
├── Need a hosted DB with no server?
│   └── Supabase (PostgreSQL) or MongoDB Atlas (free tiers)
│
└── n8n Tracker / state storage (as discussed in GitHub guide)?
    ├── Low volume (< 100/day) → GitHub JSON file or n8n Static Data
    ├── Medium volume → Redis or Supabase
    └── High volume → PostgreSQL
```

---

## Part 7 — Running Multiple Databases Together

A common production n8n stack:

```yaml
# Full production stack:
# n8n + PostgreSQL (backend) + Redis (queue mode) + MongoDB (workflow data)

services:
  postgres:    # n8n's internal database
  redis:       # n8n Queue Mode + n8n AI chat memory
  mongodb:     # Workflow target — storing automation outputs
  n8n:         # The automation engine
```

---

## Part 8 — Tools for Managing Databases

| Tool | What It Is | Supports | Free? |
|------|-----------|----------|-------|
| **DBeaver** | Desktop GUI for SQL databases | PostgreSQL, MySQL, SQLite, MongoDB | ✅ Community edition free |
| **pgAdmin** | Web UI specifically for PostgreSQL | PostgreSQL only | ✅ Free |
| **MongoDB Compass** | Desktop GUI for MongoDB | MongoDB | ✅ Free |
| **RedisInsight** | Desktop GUI for Redis | Redis | ✅ Free |
| **TablePlus** | Clean desktop GUI | PostgreSQL, MySQL, Redis, SQLite | Free (limited) |
| **Supabase Studio** | Web UI (built into Supabase) | Supabase/PostgreSQL | ✅ Free with Supabase |
| **NocoDB** | Web-based spreadsheet UI | PostgreSQL, MySQL, SQLite | ✅ Free |

**Recommended for quick exploration of your n8n database:**  
DBeaver (free, connects to the Docker PostgreSQL container by exposing port 5432 temporarily).

```bash
# Temporarily expose port 5432 for DBeaver access:
# In docker-compose.yml postgres section, add:
ports:
  - "5432:5432"
# docker compose up -d
# Connect DBeaver to localhost:5432
# Remove the port mapping when done (security)
```
