# Changelog

All notable changes to a2n.io will be documented in this file.

## [1.0.0] - 2026-02-04

### 🎉 Initial Release

**Core Features:**
- Visual workflow builder with drag-and-drop interface
- 42+ nodes across 8 categories
- AI Agent support with OpenAI, Anthropic, Google, and Groq
- Real-time workflow execution with live logs
- Self-hosted deployment with Docker

**Deployment:**
- Unified Docker image with embedded PostgreSQL and Redis
- Support for external database connections
- Zero-config startup for quick testing
- Production-ready with external DB support

**Integrations:**
- Triggers: Webhook, Schedule, Manual
- AI: OpenAI, Anthropic Claude, Google Gemini, Groq
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
