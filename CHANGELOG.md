# Changelog

All notable changes to a2n.io will be documented in this file.

## [1.0.17] - 2026-02-20

### Added
- **License Key Activation** — Self-hosted Docker features now require a free license key from a2n.io
- **Activation Page** — In-app UI to enter, view, and remove your license key
- **License Gate** — Create Custom Node, Database Tools, and System Monitor are gated until activated
- **Periodic Validation** — Docker instances validate the key with a2n.io every 6 hours with a built-in grace period
- **License Activation Guide** — [docs/LICENSE_ACTIVATION.md](docs/LICENSE_ACTIVATION.md)

---

## [1.0.16] - 2026-02-20

### Added
- **Create Custom Node** — Self-hosted users can build and install custom nodes directly (auto-approved, no review needed)
- Sidebar item "Create Custom Node" for self-hosted Docker deployments

---

## [1.0.15] - 2025-07-03

### Added
- **Expanded horizontal scaling documentation** — 4 new beginner-friendly help topics in System Monitor UI
- **docker-compose.horizontal.yml** — Ready-to-use multi-instance deployment template with main + worker services
- Horizontal scaling env var references added to docker-compose.yml and docker-compose.production.yml
- Decision guide: vertical vs. horizontal scaling
- Common mistakes to avoid when scaling
- Horizontal scaling FAQ (6 questions)

---

## [1.0.14] - 2026-02-19

### Added
- **Horizontal Scaling** — Run multiple a2n instances sharing the same database & Redis
- **Cluster Monitoring** — Live cluster status showing all instances, CPU, memory, and active executions
- **Instance Registry** — Automatic instance discovery via Redis heartbeat (10s interval, 30s TTL)
- **Docker Compose Generator** — Generate multi-instance docker-compose from the System Monitor UI
- **Environment variables** — `A2N_INSTANCE_ROLE`, `A2N_INSTANCE_ID` for cluster configuration
- Horizontal scaling documentation and inline help guides

---

## [1.0.13] - 2025-07-02

### Changed
- Hidden **Subscription** sidebar item for self-hosted Docker users (not applicable)

---

## [1.0.12] - 2025-07-01

### Changed
- Hidden **Community** section (Marketplace, Submit Node, My Submissions) for self-hosted Docker users

---

## [1.0.11] - 2025-06-30

### Added
- **System Monitor** — Live CPU, memory, heap, and active-execution metrics (5-second polling)
- **Vertical Scaling** — UI sliders to tune Max Concurrent Executions, Database Pool Size, Queue Concurrency, and Node.js Memory
- **Quick Presets** — One-click Light / Standard / Performance / Heavy capacity profiles
- **Scaling environment variables** — `A2N_MAX_CONCURRENT_EXECUTIONS`, `DATABASE_POOL_SIZE`, `A2N_QUEUE_CONCURRENCY`, `NODE_OPTIONS`
- **Database Tools** — Backup, restore, migrate to external DB, schema status & upgrades, purge execution logs (all from the UI)
- Beginner-friendly documentation inside the System Monitor page

---

## [1.0.0] - 2026-02-04

### 🎉 Initial Release

**Core Features:**
- Visual workflow builder with drag-and-drop interface
- 42+ nodes across 8 categories
- AI Agent support with OpenAI, Anthropic, Google, and Grok
- Real-time workflow execution with live logs
- Self-hosted deployment with Docker

**Deployment:**
- Unified Docker image with embedded PostgreSQL and Redis
- Support for external database connections
- Zero-config startup for quick testing
- Production-ready with external DB support

**Integrations:**
- Triggers: Webhook, Schedule, Manual
- AI: OpenAI, Anthropic Claude, Google Gemini, Grok
- Communication: Email, Slack, Discord, Telegram, Twilio
- Databases: PostgreSQL, MySQL, MongoDB, Redis
- Social: Twitter, Facebook, Reddit, LinkedIn
- Productivity: Google Sheets, Drive, Notion, Airtable
- And many more...

**Security:**
- Optional AES-256-GCM encryption for credentials
- JWT-based authentication
- Role-based access control
- Secure environment variable handling

---

## Versioning

We use [SemVer](http://semver.org/) for versioning:
- **MAJOR** version for incompatible API changes
- **MINOR** version for new functionality (backwards compatible)
- **PATCH** version for bug fixes (backwards compatible)
