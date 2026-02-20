# License Key Activation

Self-hosted Docker features require a **free lifetime license key** from [a2n.io](https://a2n.io). This lets us track Docker adoption and ensure quality support for self-hosted users.

---

## What Gets Unlocked

| Feature | Description |
|---------|-------------|
| **Create Custom Node** | Build your own nodes and use them instantly in workflows |
| **Database Tools** | Backup, restore, migrate, schema upgrades, purge logs |
| **System Monitor** | Live metrics, vertical scaling sliders, quick presets |

These features are **completely free** — the key is only used for tracking, not billing.

---

## How It Works

```
┌──────────────────────┐         ┌──────────────────────┐
│   Your Docker        │         │    a2n.io            │
│   Instance           │         │    (Managed)         │
│                      │         │                      │
│  1. Enter key ───────┼────────►│  Validate key        │
│                      │         │  Bind to instance    │
│  2. Key activated ◄──┼─────── │  Return success      │
│                      │         │                      │
│  3. Periodic check ──┼────────►│  Verify still valid  │
│     (every 7 days)   │         │                      │
└──────────────────────┘         └──────────────────────┘
```

- Each key is bound to **one Docker instance** (identified by a unique instance ID)
- Validation runs every **7 days** in the background
- If a2n.io is temporarily unreachable, a **grace period** keeps features working
- If a key is revoked from a2n.io, features are disabled on the next validation

---

## Step-by-Step Setup

### 1. Create an Account on a2n.io

Go to [https://a2n.io/signup](https://a2n.io/signup) and create a free account.

### 2. Generate a License Key

1. Log in to [a2n.io](https://a2n.io)
2. Go to **Settings → Docker License**
3. Click **"Generate Key"**
4. Copy your key (format: `a2n_dk_...`)

> You can generate up to **3 keys** per account (one per Docker instance).

### 3. Activate in Your Docker Instance

1. Open your self-hosted a2n.io at `http://localhost:8080`
2. Click any gated feature in the sidebar (Create Custom Node, Database Tools, or System Monitor)
3. You'll see the **Activation Page** — paste your key and click **"Activate License"**
4. Done! All 3 features are now unlocked.

You can also navigate directly to:
```
http://localhost:8080/dashboard/activation
```

---

## Managing Your Keys

### View Keys

On a2n.io, go to **Settings → Docker License** to see all your keys with:
- Key (masked for security)
- Status: Active, Not Activated, or Revoked
- Bound instance ID (once activated)
- Last seen timestamp

### Revoke a Key

If you decommission a Docker instance or want to free up a key:
1. Go to **Settings → Docker License** on a2n.io
2. Click **"Revoke"** next to the key
3. The Docker instance will lose access on the next validation cycle (within 7 days)

### Remove License from Docker

To deactivate your Docker instance:
1. Go to `http://localhost:8080/dashboard/activation`
2. Click **"Remove License"**
3. Features will be gated again until a new key is entered

---

## Key Rules

| Rule | Details |
|------|---------|
| **Free forever** | Keys are free — no payment, no trial, no expiration |
| **One key = one instance** | Each key binds to the first Docker instance that activates it |
| **Max 3 keys per account** | Create up to 3 keys for 3 separate Docker instances |
| **Revocable** | You can revoke keys from a2n.io at any time |
| **Grace period** | If a2n.io is temporarily unreachable, features remain active |
| **Auto-disable** | If a key is revoked, features disable within 7 days |

---

## Troubleshooting

### "Invalid license key"
- Double-check you copied the full key (starts with `a2n_dk_`)
- Make sure the key hasn't been revoked on a2n.io

### "Key already activated on another instance"
- Each key can only be used on one Docker instance
- Generate a new key from a2n.io, or revoke the old one first

### Features disabled after working
- Your key may have been revoked — check a2n.io Settings → Docker License
- If a2n.io was unreachable for an extended period, the grace period may have expired
- Re-activate with the same or a new key

### Can't reach a2n.io for validation
- The Docker instance has a built-in grace period
- Features continue working while a2n.io is temporarily down
- Once connectivity is restored, validation resumes automatically

---

## FAQ

**Q: Is the license key really free?**
A: Yes, 100% free forever. We use it only to track how many Docker instances are running in the wild.

**Q: What happens if a2n.io goes down?**
A: Your Docker instance keeps working. The grace period ensures features stay unlocked even if a2n.io is temporarily unreachable.

**Q: Can I move my key to a different Docker instance?**
A: Revoke the key on a2n.io, then activate it on the new instance. Or generate a new key.

**Q: Do I need internet access for my Docker instance?**
A: Only briefly — during initial activation and for periodic validation (every 7 days). The actual features work entirely offline.

**Q: What data does the validation send?**
A: Only the license key and instance ID. No workflow data, credentials, or usage details are ever sent.
