# Changelog

All notable changes to a2n.io will be documented in this file.

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
