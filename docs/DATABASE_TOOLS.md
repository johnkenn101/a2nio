# Database Tools — Backup, Restore & Migration

a2n.io includes built-in database tools accessible from the **Dashboard → Database Tools** page. These tools let you manage your data without ever touching the command line.

> **Note:** Database Tools is only available for self-hosted Docker deployments.

---

## Quick Overview

| Feature | What it does |
|---------|-------------|
| **Backup** | Creates a downloadable `.sql` backup of your entire database |
| **Restore** | Uploads a `.sql` backup file and restores your database from it |
| **Migrate** | Moves your data from the embedded database to an external PostgreSQL |
| **Schema Status** | Shows the current database schema version and applies pending upgrades |
| **Purge** | Deletes old workflow execution logs to free up disk space |

---

## How to Access

1. Log in to your a2n.io instance
2. Open the sidebar
3. Click **Database Tools** (under the System section)

---

## Backup

Creates a complete snapshot of your database that you can download as a `.sql` file.

### From the UI

1. Go to **Database Tools**
2. Click **Create Backup**
3. Wait for the backup to generate
4. A `.sql` file will automatically download to your computer

### From the Command Line

If you prefer the terminal:

```bash
# Using the embedded PostgreSQL
docker exec a2n pg_dump -U a2n -d a2n > backup-$(date +%Y%m%d).sql

# Or with docker compose
docker compose exec a2n pg_dump -U a2n -d a2n > backup-$(date +%Y%m%d).sql
```

### Best Practices

- **Back up before upgrading** — Always create a backup before pulling a new Docker image
- **Back up regularly** — Set a reminder to back up weekly, or more often if you change workflows frequently
- **Store backups safely** — Copy your backup files to a separate location (cloud storage, USB drive, etc.)

---

## Restore

Restores your database from a previously downloaded `.sql` backup file.

### From the UI

1. Go to **Database Tools**
2. Click **Restore from Backup**
3. Select your `.sql` backup file
4. Confirm the restore
5. Wait for the process to complete — your container may restart automatically

### From the Command Line

```bash
# Stop the app first
docker stop a2n

# Restore the backup
docker exec -i a2n psql -U a2n -d a2n < backup-20260219.sql

# Start again
docker start a2n
```

### Important Notes

- **Restore replaces all current data** — Make a fresh backup before restoring if you want to keep your current state
- **Same version recommended** — Restore backups into the same (or newer) version of a2n that created them
- **Large backups** — Very large databases may take several minutes to restore

---

## Migration (Embedded → External Database)

If you started with the default embedded PostgreSQL and want to move to an external database (recommended for production), the migration tool handles it for you.

### Why Migrate?

- **Better performance** — Dedicated database servers handle more load
- **Easier backups** — Use your cloud provider's automated backup features
- **High availability** — Cloud databases (RDS, Cloud SQL, etc.) offer replication and failover
- **Separate concerns** — Database and app can scale independently

### From the UI

1. Go to **Database Tools**
2. Click **Migrate to External DB**
3. Enter your external PostgreSQL connection string:
   ```
   postgresql://user:password@host:5432/a2n
   ```
4. Click **Start Migration**
5. Wait for the data transfer to complete
6. The tool will verify the migration was successful
7. Update your `docker-compose.yml` to point to the external database

### After Migration

Update your `docker-compose.yml` environment:

```yaml
environment:
  DATABASE_URL: "postgresql://user:password@your-db-host:5432/a2n"
  DIRECT_URL: "postgresql://user:password@your-db-host:5432/a2n"
```

Then restart:

```bash
docker compose down && docker compose up -d
```

---

## Schema Status & Upgrades

When you update to a newer version of a2n, the database schema may need to be upgraded. This usually happens automatically on startup, but you can also check and apply upgrades manually.

### From the UI

1. Go to **Database Tools**
2. Look at the **Schema Status** card
3. If upgrades are available, click **Apply Upgrades**

### What Happens During an Upgrade

- New tables or columns are added as needed
- Existing data is preserved
- The process is safe and non-destructive
- A backup is recommended before applying upgrades (just in case)

---

## Purge Execution Logs

Over time, workflow execution logs can use significant disk space. The purge feature lets you clean up old logs.

### From the UI

1. Go to **Database Tools**
2. Click **Purge Executions**
3. Choose how old the logs should be before deletion (e.g., older than 30 days)
4. Confirm the purge

### What Gets Deleted

- Workflow execution records older than your selected threshold
- Input/output data from those executions
- Error logs from those executions

### What Is NOT Deleted

- Your workflows (they are never deleted by purge)
- Your credentials
- Your node configurations
- Recent execution logs (within your threshold)

---

## Troubleshooting

### Backup download is empty or fails

```bash
# Check if PostgreSQL is running
docker exec a2n pg_isready

# Check available disk space
docker exec a2n df -h /data
```

### Restore fails

- Make sure the backup file is a valid `.sql` file (not compressed)
- Check that the file isn't corrupted (try opening it in a text editor — it should contain SQL statements)
- Ensure you have enough disk space

### Migration fails

- Verify the external database connection string is correct
- Make sure the external database is reachable from the Docker container
- Check firewall rules — the container needs to reach your database port (usually 5432)
- Ensure the database user has CREATE and INSERT permissions

---

## FAQ

**Q: Do I need to stop a2n before backing up?**
A: No. The UI backup uses `pg_dump` which creates a consistent snapshot without stopping the app.

**Q: Can I restore a backup from a different version?**
A: You can restore to the same or a newer version. Restoring to an older version may fail if schema changes were made.

**Q: How much disk space do execution logs use?**
A: It depends on how many workflows you run and how much data they process. As a rough guide, 1000 executions with medium-sized data use about 50-100 MB.

**Q: Can I schedule automatic backups?**
A: Not from the UI yet. You can set up a cron job on your server:
```bash
# Add to crontab: daily backup at 2 AM
0 2 * * * docker exec a2n pg_dump -U a2n -d a2n > /backups/a2n-$(date +\%Y\%m\%d).sql
```
