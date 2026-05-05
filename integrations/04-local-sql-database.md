# Local SQL Database Integration in n8n

> Author: Shrinivas Ramaprasad | May 2026
> MySQL + PostgreSQL: connecting, querying, cross-table joins, and automated management.

---

## Part 1 — MySQL Node: Every Field

**Credential:** MySQL (see `credentials/02-credentials-guide.md`)
**Requirement:** MySQL server accessible from n8n

### Operation: Execute Query

| Field | Options | Sample Value | What It Does | If Wrong |
|---|---|---|---|---|
| Credential | Dropdown | `MySQL - CRM Production` | Auth connection | Node fails |
| Operation | Execute Query, Insert, Update, Delete | `Execute Query` | Action | Wrong fields |
| Query | SQL string | `SELECT * FROM users WHERE status = :status` | The SQL to run | Syntax error |
| Query Parameters | Key-value | `status = active` | Binds `:status` safely | SQL injection risk |
| Options → Query Batching | Independently, Transaction | `Transaction` | Wrap in DB transaction | Partial updates on error |

Sample queries:

```sql
-- Read active users
SELECT id, name, email, created_at
FROM users
WHERE status = :status
  AND DATE(created_at) = CURDATE()
ORDER BY created_at DESC
LIMIT :limit

-- Insert new ticket
INSERT INTO tickets (title, email, priority, status, created_at)
VALUES (:title, :email, :priority, 'open', NOW())

-- Update status
UPDATE tickets
SET status = :status, updated_at = NOW()
WHERE id = :id
```

Access output: `{{ $json.id }}` `{{ $json.name }}` `{{ $json.email }}`

**⚠️ MySQL mistakes:**

| Mistake | Effect | Fix |
|---|---|---|
| Inline values: `WHERE id = {{ $json.id }}` | SQL injection risk | Use parameters: `WHERE id = :id` |
| No LIMIT on SELECT | Returns millions of rows | Always add `LIMIT :limit` |
| Wrong column name | Unknown column error | Check schema first |

---

## Part 2 — PostgreSQL Node: Every Field

**Note:** PostgreSQL uses `$1`, `$2`... placeholders (not `:name`)

```sql
-- Read with JOIN
SELECT
  u.id, u.name, u.email,
  COUNT(o.id) AS order_count,
  SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.status = $1
GROUP BY u.id, u.name, u.email
HAVING SUM(o.amount) > $2
ORDER BY total_spent DESC
LIMIT $3

-- Parameters: active, 1000, 50

-- Upsert (insert or update)
INSERT INTO contacts (email, name, updated_at)
VALUES ($1, $2, NOW())
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name, updated_at = NOW()

-- Get last inserted ID
INSERT INTO tickets (title, email) VALUES ($1, $2) RETURNING id
```

---

## Part 3 — Cross-Table Joins in n8n Workflows

### Option A — SQL JOIN in one query (best performance)

```sql
SELECT
  c.id AS customer_id, c.name, c.email,
  o.id AS order_id, o.amount, o.status AS order_status,
  p.name AS product_name
FROM customers c
INNER JOIN orders o ON o.customer_id = c.id
INNER JOIN order_items oi ON oi.order_id = o.id
INNER JOIN products p ON p.id = oi.product_id
WHERE c.status = $1
  AND o.created_at > NOW() - INTERVAL '30 days'
ORDER BY o.created_at DESC
```

### Option B — Multiple nodes with Merge (for data across different DBs)

```
PostgreSQL: Get users → { userId, name, email }
     +
MySQL: Get orders → { userId, orderId, amount }
     ↓
Merge node (Combine → By Matching Field)
  Input 1 match field: userId
  Input 2 match field: userId
     ↓
Combined: { userId, name, email, orderId, amount }
```

---

## Part 4 — Automatic DB Management Patterns

### Deduplication Before Insert

```
Get data from API
    → MySQL: SELECT id FROM leads WHERE email = :email
    → If: {{ $json.length > 0 }}
          [Exists]     → Skip or update
          [Not exists] → MySQL: INSERT INTO leads
```

### Scheduled Sync

```
Schedule Trigger (every hour)
    → HTTP Request: GET API records updated in last 2 hours
    → Split Out: individual records
    → Loop Over Items (batch: 10)
          → MySQL: CHECK if exists
          → If: exists? YES → UPDATE | NO → INSERT
          → Wait: 1 second
    [Done] → Telegram: "Sync complete"
```

### Archive Old Records

```sql
-- Archive and clean in one go
INSERT INTO orders_archive
SELECT * FROM orders
WHERE created_at < NOW() - INTERVAL 90 DAY
  AND status = 'completed';

DELETE FROM orders
WHERE created_at < NOW() - INTERVAL 90 DAY
  AND status = 'completed';
```

### Database Health Monitor

```sql
-- One query checks everything
SELECT
  (SELECT COUNT(*) FROM orders WHERE status = 'pending'
   AND created_at < NOW() - INTERVAL 1 HOUR) AS stuck_orders,
  (SELECT COUNT(*) FROM errors
   WHERE created_at > NOW() - INTERVAL 5 MINUTE) AS recent_errors
```

```
If: stuck_orders > 10 OR recent_errors > 5
    → Telegram: "⚠️ DB Alert: {{ $json.stuck_orders }} stuck orders"
```

---

## Part 5 — Docker Networking for Local DBs

### n8n and DB in same Docker Compose

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: crm_db
      MYSQL_USER: n8n_user
      MYSQL_PASSWORD: SecurePass123!

# In n8n MySQL credential:
# Host: mysql   ← use service name, NOT localhost
```

### n8n in Docker, DB on host (Windows/Mac)

```
Host in credential: host.docker.internal
Port: 3306
```

### n8n in Docker, DB on host (Linux)

```yaml
# Add to n8n service in docker-compose.yml:
extra_hosts:
  - "host.docker.internal:host-gateway"
# Then use: host.docker.internal as host
```

### DB on separate server/NAS

```
Host: 192.168.1.100  (server's LAN IP)
Port: 3306
# Ensure DB allows connections from n8n's IP:
# MySQL: GRANT ... TO 'n8n_user'@'192.168.1.%'
# Firewall: open port 3306 from n8n IP
```
