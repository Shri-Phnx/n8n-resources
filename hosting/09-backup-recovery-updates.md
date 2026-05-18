# n8n Backup, Recovery & Updates

> **Author:** Shrinivas Ramaprasad | **Updated:** May 2026
> How to back up your n8n instance, restore from backup, and safely update to new versions.

---

## What to Back Up

| What | Why | How Often |
|------|-----|-----------|
| `N8N_ENCRYPTION_KEY` | Losing this = losing all credentials permanently | Once (store in password manager) |
| n8n_data Docker volume | Workflows, credentials, executions | Daily |
| `.env` file | All configuration | After every change |
| postgres_data volume (if using PostgreSQL) | All workflow data | Daily |

> **Rule:** If you haven't tested your restore, you don't have a backup.

---

## Backup Commands

### Backup n8n Data Volume

```bash
# Back up n8n data volume to a .tar.gz file
mkdir -p ~/n8n-backups

docker run --rm \
  -v n8n-production_n8n_data:/source \
  -v ~/n8n-backups:/backup \
  alpine tar czf /backup/n8n-$(date +%Y%m%d-%H%M).tar.gz -C /source .

# Result: ~/n8n-backups/n8n-20260501-0200.tar.gz
```

### Backup PostgreSQL Database

```bash
# Dump the n8n database to a SQL file
mkdir -p ~/n8n-backups

docker exec postgres pg_dump -U n8n n8n > ~/n8n-backups/n8n-db-$(date +%Y%m%d).sql

# Result: ~/n8n-backups/n8n-db-20260501.sql
```

### Backup npm Install (non-Docker)

```bash
# Back up the n8n data folder
cp -r ~/.n8n ~/.n8n-backup-$(date +%Y%m%d)

# Back up your .env file
cp .env .env-backup-$(date +%Y%m%d)
```

### Automated Daily Backup via Cron

```bash
# Edit crontab
crontab -e

# Add this line — runs at 2 AM daily, backs up volume, deletes backups older than 7 days
0 2 * * * cd ~/n8n-production && docker run --rm \
  -v n8n-production_n8n_data:/source \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/n8n-$(date +\%Y\%m\%d).tar.gz -C /source . && \
  find $(pwd)/backups -name 'n8n-*.tar.gz' -mtime +7 -delete
```

---

## Restore from Backup

### Restore n8n Data Volume

```bash
# Step 1: Stop n8n
docker compose stop n8n

# Step 2: Restore from backup file
docker run --rm \
  -v n8n-production_n8n_data:/target \
  -v ~/n8n-backups:/backup \
  alpine tar xzf /backup/n8n-20260501-0200.tar.gz -C /target

# Step 3: Start n8n
docker compose start n8n

# Step 4: Verify workflows and credentials are present
# Open n8n UI → check Workflows list
```

### Restore PostgreSQL Database

```bash
# Step 1: Stop n8n (not postgres)
docker compose stop n8n

# Step 2: Drop and recreate the database
docker exec -it postgres psql -U n8n -c "DROP DATABASE n8n;"
docker exec -it postgres psql -U n8n -c "CREATE DATABASE n8n;"

# Step 3: Restore from SQL dump
cat ~/n8n-backups/n8n-db-20260501.sql | docker exec -i postgres psql -U n8n -d n8n

# Step 4: Start n8n
docker compose start n8n
```

### Restore npm Install (non-Docker)

```bash
# Replace current data folder with backup
rm -rf ~/.n8n
cp -r ~/.n8n-backup-20260501 ~/.n8n

# Restart n8n
n8n start
```

---

## Updating n8n

### Update: Docker

```bash
cd ~/n8n-production

# Step 1: Pull new image
docker compose pull n8n

# Step 2: Back up before updating (always)
docker run --rm \
  -v n8n-production_n8n_data:/source \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/pre-update-$(date +%Y%m%d).tar.gz -C /source .

# Step 3: Recreate n8n container with new image (data volume is preserved)
docker compose up -d n8n

# Step 4: Verify new version is running
docker exec -it n8n n8n --version

# Step 5: Check logs for migration issues
docker compose logs -f n8n
```

### Update: npm Install

```bash
# Step 1: Back up first
cp -r ~/.n8n ~/.n8n-backup-pre-update-$(date +%Y%m%d)

# Step 2: Update n8n
npm update n8n -g

# Step 3: Verify version
n8n --version

# Step 4: Start n8n
n8n start
```

---

## Common Restore Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Credentials show "cannot decrypt" after restore | `N8N_ENCRYPTION_KEY` doesn't match what was used when credentials were saved | Restore original `N8N_ENCRYPTION_KEY` from password manager |
| Workflows missing after restore | Restored to wrong volume or wrong folder | Verify you restored to the correct volume: `docker volume ls` |
| Database connection error after restore | PostgreSQL restore created tables in wrong schema | Check schema: `GRANT USAGE ON SCHEMA public TO n8n_user;` |
| n8n fails to start after update | Breaking change in new version | Check n8n release notes: https://github.com/n8n-io/n8n/releases |

---

## Backup Checklist (Run Monthly)

```
✅ N8N_ENCRYPTION_KEY stored in password manager
✅ Latest .env file saved to secure storage
✅ n8n_data volume backup exists and is recent
✅ PostgreSQL dump exists and is recent (if applicable)
✅ Restore procedure tested on a clean instance
✅ Cron job for automated daily backup is active
✅ Old backups pruned (> 7 days) to save disk space
```
