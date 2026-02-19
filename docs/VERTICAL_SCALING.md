# Vertical Scaling — Make a2n Handle More Work

Vertical scaling means giving your a2n instance more power so it can run more workflows at the same time. You can do this directly from the UI — no command line needed.

> **Note:** The System Monitor page is only available for self-hosted Docker deployments.

---

## Quick Overview

| What you can control | What it does | Applies |
|---------------------|-------------|---------|
| **Max Concurrent Executions** | How many workflows run at the same time | Instantly |
| **Database Pool Size** | How many database connections are open | After restart |
| **Queue Concurrency** | How many scheduled workflows fire at once | After restart |
| **Node.js Memory** | How much RAM the app can use | After restart |

---

## How to Scale Up from the UI

### Step 1: Open System Monitor

1. Log in to your a2n.io instance
2. Open the sidebar
3. Click **System Monitor**

### Step 2: Check Your Current Usage

At the top of the page, you'll see live metrics:

- **CPU** — How busy your server's processor is
- **Memory** — How much RAM is being used
- **Node.js Heap** — How much memory the app itself is using
- **Active Executions** — How many workflows are running right now

If any of these are consistently high (above 80%), it's time to scale up.

### Step 3: Adjust the Sliders

Scroll down to the **Vertical Scaling** section. You'll see 4 sliders:

1. **Max Concurrent Executions** — Drag right to allow more workflows to run at the same time
2. **Database Pool Size** — Increase if you have many concurrent workflows
3. **Queue Concurrency** — Increase if you have many scheduled/cron workflows
4. **Node.js Memory** — Increase if you work with large data or many workflows

### Step 4: Save

Click the **Save Changes** button. You'll see one of two messages:

- ✅ **"Applied successfully"** — Done! The change is already active.
- 🔄 **"Restart required"** — Run `docker compose restart` to apply the remaining changes.

---

## Quick Presets

Don't want to fiddle with individual sliders? Use a preset:

| Preset | Best For | Memory | Concurrent Executions |
|--------|----------|--------|-----------------------|
| **Light** | Personal use, small server (1-2 GB RAM) | 512 MB | 5 |
| **Standard** | Small team, 10-20 workflows (4 GB RAM) | 1024 MB | 10 |
| **Performance** | Growing team, 50+ workflows (8 GB RAM) | 2048 MB | 25 |
| **Heavy** | Large teams, heavy usage (16 GB RAM) | 4096 MB | 50 |

Just click a preset and then click **Save Changes**.

> **Not sure which to pick?** Start with **Standard** — you can always change it later.

---

## Recommended Settings by Server Size

| Your Server | Executions | DB Pool | Queue | Memory |
|-------------|-----------|---------|-------|--------|
| 1 GB RAM / 1 vCPU | 3 | 5 | 2 | 256 MB |
| 2 GB RAM / 1 vCPU | 5 | 5 | 3 | 512 MB |
| 4 GB RAM / 2 vCPU | 10 | 10 | 5 | 1024 MB |
| 8 GB RAM / 4 vCPU | 25 | 20 | 10 | 2048 MB |
| 16 GB RAM / 8 vCPU | 50 | 40 | 20 | 4096 MB |

---

## Alternative: Scale via Docker Compose

You don't need the UI — you can also set these values as environment variables in your `docker-compose.yml`:

```yaml
services:
  a2n:
    image: sudoku1016705/a2n:latest
    environment:
      # Scaling settings
      - A2N_MAX_CONCURRENT_EXECUTIONS=25
      - DATABASE_POOL_SIZE=20
      - A2N_QUEUE_CONCURRENCY=10
      - NODE_OPTIONS=--max-old-space-size=2048
      
      # ... your other settings
```

Then restart:

```bash
docker compose down && docker compose up -d
```

Or with `docker run`:

```bash
docker run -d \
  --name a2n \
  -p 8080:8080 \
  -v a2n-data:/data \
  -e A2N_MAX_CONCURRENT_EXECUTIONS=25 \
  -e DATABASE_POOL_SIZE=20 \
  -e A2N_QUEUE_CONCURRENCY=10 \
  -e NODE_OPTIONS="--max-old-space-size=2048" \
  sudoku1016705/a2n:latest
```

### Priority Order

If you set values in **both** docker-compose and the UI:

1. Docker compose environment variables always win
2. UI-saved settings are second
3. Built-in defaults are last

To use the UI sliders, remove the env vars from docker-compose.

---

## Environment Variables Reference

| Variable | What it controls | Default | Range |
|----------|-----------------|---------|-------|
| `A2N_MAX_CONCURRENT_EXECUTIONS` | Max parallel workflow runs | `10` | 1 – 100 |
| `DATABASE_POOL_SIZE` | PostgreSQL connection pool | `10` | 2 – 100 |
| `A2N_QUEUE_CONCURRENCY` | Scheduled workflow parallelism | `5` | 1 – 50 |
| `NODE_OPTIONS` | Node.js flags (set `--max-old-space-size`) | `512` MB | 256 – 16384 MB |

---

## How Do I Know I Need to Scale Up?

Watch for these signs on the System Monitor page:

| Sign | What to do |
|------|-----------|
| Workflows fail with **"maximum capacity"** | Increase **Max Concurrent Executions** |
| **CPU** stays above 80% | Upgrade to a server with more cores |
| **Memory** stays above 80% | Increase **Node.js Memory** or upgrade server |
| **Scheduled workflows run late** or skip | Increase **Queue Concurrency** |
| **Yellow warning banner** appears | Follow the suggestion in the banner |

A yellow warning banner will automatically appear on the System Monitor page when your system is running low on resources.

---

## What Applies Instantly vs. Needs a Restart?

| Setting | When it takes effect | What to do |
|---------|---------------------|-----------|
| Max Concurrent Executions | ⚡ **Instantly** | Just save |
| Database Pool Size | 🔄 After restart | `docker compose restart` |
| Queue Concurrency | 🔄 After restart | `docker compose restart` |
| Node.js Memory | 🔄 After restart | `docker compose restart` |

> **Tip:** Restarting only takes a few seconds and won't lose any data. Your workflows and credentials are safely stored.

---

## FAQ

**Q: Will I lose my data if I restart?**
A: No. All workflows, credentials, and execution history are stored in the database. A restart only briefly stops the app.

**Q: Can I break something by changing these settings?**
A: It's very hard to break anything. If values are too high for your server, things may slow down — just lower them. The Quick Presets are always safe.

**Q: What if I set Node.js Memory higher than my server's RAM?**
A: The app may crash or become very slow. Always keep Node.js Memory well below your total server RAM (leave at least 1 GB for the OS and database).

**Q: Can I scale back down?**
A: Yes. Move the sliders down and save, or pick a lower preset. You can change these as often as you want.

**Q: How do I check my server's RAM?**
A: Look at the **Memory** card at the top of the System Monitor page — it shows total and used memory. Or check your hosting provider's dashboard.
