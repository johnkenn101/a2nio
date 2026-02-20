# Horizontal Scaling — Run Multiple a2n Instances

Horizontal scaling means running **multiple a2n containers** that share the same database and Redis. Each instance processes workflows independently, multiplying your total capacity.

> **When to use:** You've already maxed out vertical scaling on a single server, or you want high availability across multiple machines.

---

## Single Instance vs Multi-Instance

```
 Single Instance:              Multi-Instance (Horizontal):

 ┌──────────────┐              ┌──────────┐
 │     a2n      │              │  main    │ ← UI + workflows
 │  (embedded   │              └────┬─────┘
 │   DB + Redis │                   │
 │   all-in-one)│            ┌──────┼──────┐
 └──────────────┘            │      │      │
                        ┌────┴─┐ ┌──┴───┐ ┌┴─────┐
                        │worker│ │worker│ │worker│ ← workflows only
                        └──────┘ └──────┘ └──────┘
                             │      │      │
                        ┌────┴──────┴──────┴────┐
                        │  Shared PostgreSQL     │
                        │  Shared Redis          │
                        └───────────────────────┘
```

| | Single Instance | Multi-Instance |
|---|---|---|
| **Containers** | 1 | 1 main + N workers |
| **Database** | Embedded (inside container) | External PostgreSQL (shared) |
| **Redis** | Embedded (inside container) | External Redis (shared) |
| **Capacity** | Limited by one machine | Scales across machines |
| **Setup** | `docker run` one-liner | `docker compose` with shared services |

---

## Migrating from Single Instance to Multi-Instance

If you're currently running a2n with `docker run` (single container), follow these steps to move to horizontal scaling.

### Step 1 — Back up your data

Before changing anything, back up your workflows and credentials. Go to **Database Tools** in the sidebar and click **Backup**.

### Step 2 — Stop your current container

```bash
docker stop a2n && docker rm a2n
```

### Step 3 — Download the horizontal scaling template

```bash
curl -O https://raw.githubusercontent.com/johnkenn101/a2nio/main/docker-compose.horizontal.yml
```

### Step 4 — Create a `.env` file

Create a `.env` file in the same folder as the compose file. This keeps secrets out of your docker-compose:

```env
# REQUIRED — Generate with: openssl rand -hex 32
ENCRYPTION_KEY=your-64-char-hex-key-here
JWT_SECRET=your-jwt-secret-here

# Database credentials
POSTGRES_USER=a2n
POSTGRES_PASSWORD=your-secure-db-password
POSTGRES_DB=a2n

# Optional — Redis password (recommended for production)
# REDIS_PASSWORD=your-redis-password

# Optional — Worker count (default: 2)
# WORKER_COUNT=3

# Optional — Scaling limits per instance
# A2N_MAX_CONCURRENT_EXECUTIONS=10
# DATABASE_POOL_SIZE=10
# A2N_QUEUE_CONCURRENCY=5
# NODE_MEMORY_MB=1024

# Optional — AI provider API keys
# OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...
```

> **Important:** If you had an `ENCRYPTION_KEY` in your previous single-instance setup, **use the same key here**. Otherwise your saved credentials won't be readable.

### Step 5 — Start the cluster

```bash
docker compose -f docker-compose.horizontal.yml up -d
```

This starts: 1 main instance + 2 workers + PostgreSQL + Redis.

### Step 6 — Restore your data

Open **http://localhost:8080**, go to **Database Tools**, and restore the backup from Step 1.

### Step 7 — Verify everything works

Go to **System Monitor** — you should see all instances in the cluster table with green health indicators.

### Step 8 — Scale up (whenever you need more capacity)

```bash
# Add more workers
docker compose -f docker-compose.horizontal.yml up -d --scale a2n-worker=5

# Or reduce
docker compose -f docker-compose.horizontal.yml up -d --scale a2n-worker=1
```

---

## How It Works

```
                    ┌───────────────┐
                    │  Load Balancer│
                    │  (Traefik /   │
                    │   nginx)      │
                    └───────┬───────┘
               ┌────────────┼────────────┐
               ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ a2n-main │ │a2n-worker│ │a2n-worker│
        │ (port    │ │ (no port)│ │ (no port)│
        │  8080)   │ │          │ │          │
        └────┬─────┘ └────┬─────┘ └────┬─────┘
             │             │             │
     ────────┴─────────────┴─────────────┴────────
     │                                            │
     │   Shared PostgreSQL    Shared Redis         │
     │                                            │
     ──────────────────────────────────────────────
```

- **All instances connect to the same PostgreSQL and Redis**
- BullMQ (the job queue) automatically distributes scheduled workflow jobs across all workers
- Each instance tracks its own capacity and registers in Redis for cluster monitoring
- The System Monitor page shows all instances and their health in real time

---

## Prerequisites

Horizontal scaling requires:

1. **External PostgreSQL** — A shared database all instances connect to
2. **External Redis** — A shared Redis all instances connect to
3. **A load balancer** (optional) — If you want to distribute web traffic across instances (Traefik, nginx, Caddy, etc.)

> You **cannot** use the embedded PostgreSQL/Redis for multi-instance setups.

---

## Quick Start

### 1. Create `docker-compose.yml`

```yaml
version: '3.8'

services:
  # Main instance — serves UI & API, processes jobs
  a2n-main:
    image: sudoku1016705/a2n:latest
    container_name: a2n-main
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://a2n:your-secure-password@postgres:5432/a2n
      - REDIS_HOST=redis
      - SELF_HOSTED=true
      - A2N_INSTANCE_ROLE=main
      - ENCRYPTION_KEY=your-64-char-hex-key
      - JWT_SECRET=your-jwt-secret
      - A2N_MAX_CONCURRENT_EXECUTIONS=10
      - DATABASE_POOL_SIZE=10
      - A2N_QUEUE_CONCURRENCY=5
      - NODE_OPTIONS=--max-old-space-size=1024
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  # Worker instances — process queue jobs
  a2n-worker:
    image: sudoku1016705/a2n:latest
    environment:
      - DATABASE_URL=postgresql://a2n:your-secure-password@postgres:5432/a2n
      - REDIS_HOST=redis
      - SELF_HOSTED=true
      - A2N_INSTANCE_ROLE=worker
      - ENCRYPTION_KEY=your-64-char-hex-key    # MUST match main
      - JWT_SECRET=your-jwt-secret             # MUST match main
      - A2N_MAX_CONCURRENT_EXECUTIONS=10
      - DATABASE_POOL_SIZE=10
      - A2N_QUEUE_CONCURRENCY=5
      - NODE_OPTIONS=--max-old-space-size=1024
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    deploy:
      replicas: 2    # Add more workers here
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    container_name: a2n-postgres
    environment:
      - POSTGRES_DB=a2n
      - POSTGRES_USER=a2n
      - POSTGRES_PASSWORD=your-secure-password
    volumes:
      - pg-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U a2n -d a2n"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: a2n-redis
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    restart: unless-stopped

volumes:
  pg-data:
  redis-data:
```

### 2. Start the Cluster

```bash
docker compose up -d
```

### 3. Check Status

Open **http://localhost:8080** and go to **System Monitor**. You'll see all instances in the cluster table.

### 4. Scale Up or Down

```bash
# Add more workers
docker compose up -d --scale a2n-worker=5

# Reduce workers
docker compose up -d --scale a2n-worker=1
```

---

## Environment Variables for Horizontal Scaling

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | Shared PostgreSQL connection string | Yes |
| `REDIS_HOST` | Shared Redis hostname | Yes |
| `REDIS_PORT` | Redis port (default: 6379) | No |
| `REDIS_PASSWORD` | Redis password | No |
| `A2N_INSTANCE_ROLE` | Instance label: `main` or `worker` | No (default: `main`) |
| `A2N_INSTANCE_ID` | Custom instance identifier | No (auto-generated) |
| `ENCRYPTION_KEY` | Must be **identical** across all instances | Yes |
| `JWT_SECRET` | Must be **identical** across all instances | Yes |

---

## Main vs Worker Instances

| Feature | Main | Worker |
|---------|------|--------|
| Serves UI & API | Yes | Yes |
| Processes queue jobs | Yes | Yes |
| Port exposed | 8080 (for users) | None (internal) |
| Suggested count | 1 | 1-10+ |

Both instance types run the same code. The `A2N_INSTANCE_ROLE` is a label used in the cluster monitoring view. The main instance simply has its port exposed for user access.

> **Tip:** You can put a load balancer (Traefik, nginx) in front of all instances if you want to distribute API/webhook traffic across them.

---

## Cluster Monitoring

The **System Monitor** page shows:

- **Cluster summary** — Total instances online, aggregate CPU/memory/execution stats
- **Instance table** — Per-instance health, CPU, memory, active executions, uptime
- **Health indicators** — Green (healthy), yellow (slow heartbeat), red (stale/offline)

Each instance reports its status to Redis every 10 seconds. If an instance doesn't report for 30 seconds, it's automatically removed from the cluster view.

---

## Important Notes

### Database Connections

Each instance opens its own database connection pool. With N instances and `DATABASE_POOL_SIZE=10`, you'll have N × 10 connections to PostgreSQL. Make sure your PostgreSQL `max_connections` setting can handle this:

```
Total connections needed = (1 main + N workers) × DATABASE_POOL_SIZE
```

For example, 1 main + 4 workers with pool size 10 = 50 connections. PostgreSQL default is 100, which is fine.

### Startup Sequence

When multiple instances start simultaneously, each runs database migrations independently. PostgreSQL handles this safely with advisory locks, but it's cleaner to:

1. Start `postgres` and `redis` first
2. Start `a2n-main` and wait for it to be healthy
3. Then start workers

### Encryption Key & JWT Secret

**All instances MUST use the same `ENCRYPTION_KEY` and `JWT_SECRET`**. If they differ:
- Credentials encrypted by one instance can't be decrypted by another
- JWT tokens from one instance won't be valid on another

### Webhooks

Webhooks arrive at whichever instance handles the HTTP request (the one with port exposed, or one chosen by the load balancer). The webhook workflow runs on that specific instance, not through the queue.

---

## FAQ

**Q: Do I need a load balancer?**
A: Not necessarily. The simplest setup exposes only the main instance's port. Workers process queue jobs automatically. A load balancer is only needed if you want to distribute webhook or API traffic.

**Q: Can I run instances on different servers?**
A: Yes! As long as all instances can reach the same PostgreSQL and Redis, they can run on different machines. This is ideal for true high availability.

**Q: How many workers should I run?**
A: Start with 1-2 workers and monitor the cluster view. If CPU and memory are consistently high across instances, add more. Each worker multiplies your queue processing capacity.

**Q: Can I mix vertical and horizontal scaling?**
A: Yes. Give each instance more resources (vertical) AND run more instances (horizontal). The scaling sliders apply per-instance.

**Q: What happens if a worker crashes?**
A: BullMQ automatically retries failed jobs on another available instance. The crashed instance disappears from the cluster view after 30 seconds. Data is never lost.

**Q: Will this cost more?**
A: Each additional instance uses more CPU and memory. On cloud providers, you pay for each container's resources. Start small and scale only when needed.
