# a2n.io - Workflow Automation & AI Agents Platform

<p align="center">
  <img src="a2n-logo.png" alt="a2n.io Logo" width="200"/>
</p>

<p align="center">
  <strong>Build powerful automations and AI agents with a visual workflow builder</strong>
</p>

<p align="center">
  <a href="https://hub.docker.com/r/sudoku1016705/a2n">
    <img src="https://img.shields.io/docker/pulls/sudoku1016705/a2n" alt="Docker Pulls"/>
  </a>
  <a href="https://hub.docker.com/r/sudoku1016705/a2n">
    <img src="https://img.shields.io/docker/v/sudoku1016705/a2n/latest" alt="Docker Version"/>
  </a>
  <a href="https://github.com/a2nio/a2nio-open/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"/>
  </a>
</p>

---

## 🚀 What is a2n.io?

**a2n.io** is a powerful, self-hosted workflow automation platform that combines:

- 🔧 **Visual Workflow Builder** - Drag-and-drop interface to create complex automations
- 🤖 **AI Agents** - Built-in AI capabilities powered by OpenAI, Anthropic, and more
- 🔗 **110+ Integrations** - Connect to popular services and APIs
- ⚡ **Real-time Execution** - Watch your workflows run with live logs
- 🏠 **Self-Hosted** - Your data stays on your infrastructure

---

## 📦 Quick Start with Docker

### Option 1: Single Command (Recommended)

The easiest way to run a2n.io is with our all-in-one Docker image that includes everything:

```bash
docker run -d \
  --name a2n \
  -p 8080:8080 \
  -v a2n-data:/data \
  sudoku1016705/a2n:latest
```

Then open **http://localhost:8080** in your browser.

### Option 2: Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  a2n:
    image: sudoku1016705/a2n:latest
    container_name: a2n
    ports:
      - "8080:8080"
    volumes:
      - a2n-data:/data
    environment:
      # Optional: Set encryption key for credential security
      # ENCRYPTION_KEY: your-32-character-secret-key-here
      
      # Optional: Set JWT secret for session management
      # JWT_SECRET: your-jwt-secret-key
    restart: unless-stopped

volumes:
  a2n-data:
```

Then run:

```bash
docker-compose up -d
```

---

## 🔧 Configuration Options

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PORT` | Backend server port | `5000` |
| `ENCRYPTION_KEY` | Key for encrypting credentials (32+ chars recommended) | Auto-disabled if not set |
| `JWT_SECRET` | Secret for JWT tokens | Auto-generated |
| `DATABASE_URL` | External PostgreSQL connection string | Embedded PostgreSQL |
| `REDIS_URL` | External Redis connection string | Embedded Redis |
| `SELF_HOSTED` | Enable unlimited self-hosted mode | `true` |

### Using External Database (Production)

For production deployments, we recommend using an external PostgreSQL database:

```bash
docker run -d \
  --name a2n \
  -p 8080:8080 \
  -e DATABASE_URL="postgresql://user:password@host:5432/a2n" \
  -e REDIS_URL="redis://host:6379" \
  -e ENCRYPTION_KEY="your-32-character-secret-key-here" \
  sudoku1016705/a2n:latest
```

---

## 🔌 Supported Nodes (110+)

### Triggers
| Node | Description |
|------|-------------|
| **Webhook Trigger** | Trigger workflows via HTTP requests with authentication, custom responses, and path routing |
| **Schedule Trigger** | Run workflows every day, hour, or custom interval |
| **Manual Trigger** | Start workflows manually with a button click |
| **Chat Message Trigger** | Trigger workflows from user chat messages — for use with AI nodes |
| **Form Submission Trigger** | Generate webforms in a2n and pass their responses to the workflow |

### AI & LLM
| Node | Description |
|------|-------------|
| **OpenAI** | GPT-4, GPT-4o, GPT-3.5 for text generation, chat, images, audio, and video analysis |
| **Grok AI** | xAI Grok models for text generation, chat, vision understanding, and image generation |
| **AI Agent** | Autonomous AI agents with tool use — supports OpenAI, Anthropic (Claude), and Google Gemini providers |
| **AI Prompt** | Send prompts to any LLM — supports OpenAI, Gemini, Claude, Grok, and custom endpoints |
| **LLM Chain** | Send a prompt to an LLM — lighter than a full AI agent, ideal for simple prompt-to-output pipelines |
| **MCP Protocol Tool** | Connect to MCP servers to access tools, resources, and prompts for AI agents (HTTP, SSE, STDIO) |
| **MCP Client Tool** | Connect to an external MCP server and invoke its tools (HTTP Streamable and SSE) |
| **Tavily Search** | AI-optimised web search via Tavily API with citation-ready results and content extraction |
| **Chat Memory** | Buffer-window memory for AI agents — maintains sliding window of recent conversation messages |
| **Sentiment Analyzer** | Classify text sentiment using an LLM with customizable categories and confidence scores |
| **AI Summarizer** | Summarize long text using stuff, map-reduce, or refine strategies via LLM |
| **Output Corrector** | Automatically fix AI-generated output that does not match the expected JSON schema |
| **Schema Parser** | Enforce a JSON schema on AI agent output — validates and extracts structured data |

### Communication
| Node | Description |
|------|-------------|
| **Send Email (SMTP)** | Send emails via any SMTP server with HTML/text support and attachments |
| **Slack** | Send messages, manage channels, files, reactions, and user groups |
| **Discord** | Send messages, manage channels, members, and webhooks |
| **Telegram** | Send messages, photos, documents, media groups, and manage chats and bots |
| **Twilio** | Send SMS, make calls, and manage Twilio communication services |

### Email Marketing & Transactional
| Node | Description |
|------|-------------|
| **Brevo** | Send transactional emails and manage contacts (formerly Sendinblue) |
| **Mailgun** | Send transactional and bulk emails with template support |
| **SendGrid** | Send transactional emails and manage contacts |
| **Mandrill** | Send transactional emails through Mandrill (Mailchimp Transactional) |
| **Mailchimp** | Manage audiences, members, campaigns, and tags |
| **MailerLite** | Manage subscribers and groups |
| **Postmark** | Send transactional emails through Postmark |
| **ConvertKit** | Manage subscribers, tags, forms, and sequences |
| **Mailjet** | Send transactional emails and SMS |
| **ActiveCampaign** | Manage contacts, deals, tags, and lists |

### CRM
| Node | Description |
|------|-------------|
| **HubSpot** | Manage contacts, companies, deals, tickets, and engagements with upsert and SOQL search |
| **Salesforce** | Manage leads, contacts, accounts, opportunities, cases, tasks, and custom objects |
| **Pipedrive** | Manage deals, persons, organizations, activities, leads, notes, and products |
| **Agile CRM** | Manage contacts, companies, and deals with tag and lead-score support |
| **Copper** | Manage persons, companies, leads, opportunities, tasks, and projects |
| **Freshworks CRM** | Manage contacts, accounts, deals, tasks, notes, and appointments (Freshsales) |
| **Salesmate** | Manage companies, deals, and activities via session-token authenticated API |
| **Keap** | Manage contacts, companies, notes, tags, emails, and e-commerce in Keap (Infusionsoft) |
| **Affinity** | Manage persons, organizations, lists, and list entries for relationship intelligence |
| **HighLevel** | Manage contacts, opportunities, tasks, and calendars in GoHighLevel |

### Data & APIs
| Node | Description |
|------|-------------|
| **HTTP Request** | Make REST API calls to any service with full authentication support |
| **Webhook Response** | Send a custom response back to the HTTP caller that triggered the workflow |
| **Web Scraper** | Scrape web pages with geo-targeting, CAPTCHA bypass, and structured output |

### Databases
| Node | Description |
|------|-------------|
| **PostgreSQL** | Execute queries, insert, update, and delete rows in PostgreSQL |
| **MySQL** | Execute queries, insert, update, and delete rows in MySQL |
| **MongoDB** | Find, insert, update, and delete documents in MongoDB |
| **Redis** | Get, set, delete keys, manage lists, and publish messages |
| **Supabase** | Read, create, update, and delete rows in Supabase tables |
| **CrateDB** | Run SQL queries against CrateDB distributed database |
| **QuestDB** | Run queries against QuestDB time-series database |
| **TimescaleDB** | Run SQL queries against TimescaleDB time-series database |
| **Snowflake** | Run SQL queries against Snowflake Data Cloud |

### Spreadsheets & No-Code Databases
| Node | Description |
|------|-------------|
| **Google Sheets** | Read, write, append, update, and manage Google Sheets data |
| **Airtable** | Interact with Airtable bases and records |
| **NocoDB** | CRUD operations on NocoDB open-source Airtable alternative |
| **Baserow** | CRUD operations on Baserow open-source database |
| **SeaTable** | CRUD operations on SeaTable spreadsheet database |
| **Stackby** | List, read, append, and delete records in Stackby |
| **QuickBase** | Query, create, update, and delete records in QuickBase |
| **Grist** | CRUD operations on Grist spreadsheet database |
| **Coda** | CRUD operations on Coda document tables, list tables, and get formulas |
| **Metabase** | Run questions, native queries, list databases and collections in Metabase |
| **Data Table** | Permanently save and query data across workflow executions in structured tables |

### Social Media
| Node | Description |
|------|-------------|
| **Twitter/X** | Create, delete, search tweets, send DMs, and manage users and lists |
| **Reddit** | Create posts, comments, and interact with Reddit communities |
| **Facebook** | Create, read, update posts, comments, events, photos, and videos |

### Productivity & Docs
| Node | Description |
|------|-------------|
| **Google Docs** | Create, read, and update Google Docs documents |
| **Notion** | Create, read, update, and delete Notion pages and databases |
| **Hacker News** | Fetch articles, search stories, and look up users |

### CMS & Content
| Node | Description |
|------|-------------|
| **WordPress** | Create, update, and manage posts, pages, and media via REST API |
| **Ghost** | Create, update, and manage posts and members using Admin or Content API |
| **Medium** | Create and publish stories using Integration Tokens |
| **Contentful** | Manage entries, assets, and content types in Contentful headless CMS |
| **Storyblok** | Create, update, and publish stories in Storyblok visual CMS |
| **Strapi** | CRUD operations on Strapi headless CMS content types via REST API |
| **Webflow** | Manage CMS items, sites, and collections in Webflow |

### E-Commerce & Payments
| Node | Description |
|------|-------------|
| **Shopify** | Manage products, orders, customers, and inventory |
| **WooCommerce** | Manage products, orders, and customers on WooCommerce WordPress stores |
| **Magento** | Manage products, orders, and customers on Magento 2 stores |
| **Stripe** | Create charges, manage customers, subscriptions, and invoices |
| **PayPal** | Process payments, manage orders, and handle payouts |
| **Gumroad** | Manage products, sales, and subscribers on Gumroad |
| **Tapfiliate** | Track affiliate conversions, manage affiliates, and report on commissions |

### Scheduling & Events
| Node | Description |
|------|-------------|
| **Calendly** | Manage events, invitees, and scheduling links |
| **Cal.com** | Manage bookings, event types, and availability (open-source scheduling) |
| **Acuity Scheduling** | Manage appointments, clients, and calendars |
| **Eventbrite** | Create events, manage attendees, and track ticket sales |
| **Demio** | Manage webinars, registrants, and sessions |
| **GoToWebinar** | Create webinars, manage registrants, and track attendance |

### URL Shorteners
| Node | Description |
|------|-------------|
| **Bitly** | Shorten URLs, manage links, and track click analytics |
| **YOURLS** | Shorten URLs and manage links with your self-hosted YOURLS instance |

### Flow Control
| Node | Description |
|------|-------------|
| **IF** | Branch workflow based on conditions (string, number, date, boolean, array, object) |
| **Merge** | Merge data from multiple branches with matching, appending, or combining |
| **Loop Over Items** | Process items one by one or in batches |
| **Wait** | Pause workflow execution for a specified time or until a specific date |
| **Filter** | Filter data based on flexible conditions |
| **Approval** | Wait for manual approval before continuing — supports multi-approver workflows |
| **No Operation** | Pass input data through unchanged — useful as a placeholder in conditional branches |

### Data Transformation
| Node | Description |
|------|-------------|
| **Code (JavaScript)** | Run custom JavaScript code to transform data |
| **Set** | Set, map, and reshape data fields with manual or JSON modes |
| **Combine Items** | Combine field values from many items into arrays, or merge all items into a single list |
| **Cap Results** | Restrict the number of items passed through — keep first N or last N items |
| **Deduplicate** | Remove duplicate items based on field comparison or full item comparison |
| **Reorder** | Sort items by field values, randomly shuffle, or use custom comparator code |
| **Unwind** | Split an array field inside items into separate items (like MongoDB $unwind) |
| **Rollup** | Aggregate data with sum, count, average, min, max, append, concatenate across items |

### AI Agent Tools
| Node | Description |
|------|-------------|
| **HTTP Tool** | Make HTTP requests as a tool for AI agents |
| **Simple Vector Store** | Search or add documents using keyword matching for AI agents |

---

## 🏗️ Architecture

The a2n.io Docker image is a unified, all-in-one solution:

```
┌─────────────────────────────────────────┐
│           a2n.io Container              │
├─────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────────┐  │
│  │   Frontend  │  │    Backend      │  │
│  │  (React UI) │  │   (NestJS)      │  │
│  └─────────────┘  └─────────────────┘  │
│  ┌─────────────┐  ┌─────────────────┐  │
│  │ PostgreSQL  │  │     Redis       │  │
│  │ (Embedded)  │  │   (Embedded)    │  │
│  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────┘
```

**Embedded Mode (Default):**
- PostgreSQL 16 and Redis run inside the container
- Data persisted via Docker volume
- Zero external dependencies

**External Database Mode:**
- Provide `DATABASE_URL` and `REDIS_URL`
- Container connects to your existing infrastructure
- Recommended for production

---

## 📊 System Requirements

### Minimum
- **CPU:** 1 core
- **RAM:** 2 GB
- **Storage:** 10 GB
- **Docker:** 20.10+

### Recommended
- **CPU:** 2+ cores
- **RAM:** 4+ GB
- **Storage:** 50+ GB SSD
- **Docker:** 24.0+

---

## 🔄 Updating

a2n.io supports **upgrades** - your workflows, credentials, and data are preserved automatically via the Docker volume. The app will also **notify you** of new versions with a toast message when you log in.

### Upgrade to Latest Version

```bash
# 1. Pull the latest image (safe to run while container is running)
docker pull sudoku1016705/a2n:latest

# 2. Stop and remove the old container (volume is preserved)
docker stop a2n && docker rm a2n

# 3. Start new container with the same volume
docker run -d \
  --name a2n \
  -p 8080:8080 \
  -v a2n-data:/data \
  sudoku1016705/a2n:latest
```

> **💡 Zero data loss:** The `a2n-data` volume holds all your data (PostgreSQL, Redis, credentials, secrets). As long as you **don't** run `docker volume rm a2n-data`, everything is preserved across upgrades.

### Upgrade to a Specific Version

```bash
docker stop a2n && docker rm a2n

docker run -d \
  --name a2n \
  -p 8080:8080 \
  -v a2n-data:/data \
  sudoku1016705/a2n:1.0.3
```

Available version tags can be found on [Docker Hub](https://hub.docker.com/r/sudoku1016705/a2n/tags).

### ⬇️ Downgrade to a Previous Version

If you need to roll back to an earlier version:

```bash
# 1. Stop and remove the current container
docker stop a2n && docker rm a2n

# 2. Run the older version with the same volume
docker run -d \
  --name a2n \
  -p 8080:8080 \
  -v a2n-data:/data \
  sudoku1016705/a2n:1.0.2
```

> **⚠️ Downgrade Notes:**
> - Downgrading is safe for patch versions (e.g., 1.0.3 → 1.0.2).
> - If a newer version introduced database schema changes (migrations), downgrading may cause issues. Check the release notes before downgrading across major or minor versions.
> - As a precaution, you can **backup your volume** before upgrading:
>   ```bash
>   docker run --rm -v a2n-data:/data -v $(pwd):/backup alpine tar czf /backup/a2n-backup.tar.gz /data
>   ```
>   To restore from backup:
>   ```bash
>   docker run --rm -v a2n-data:/data -v $(pwd):/backup alpine sh -c "cd / && tar xzf /backup/a2n-backup.tar.gz"
>   ```

### 🔔 Automatic Update Notifications

a2n.io checks Docker Hub for newer versions and notifies you with a toast message on login/signup when an update is available. This check runs every 6 hours and requires no configuration.

---

## 🔒 Security Notes

1. **HTTPS:** Use a reverse proxy (nginx, Traefik, Caddy) for HTTPS in production
2. **Encryption:** Set `ENCRYPTION_KEY` to encrypt stored credentials
3. **Firewall:** Only expose port 5000 to trusted networks
4. **Updates:** Regularly pull the latest image for security patches

---

## 🆘 Troubleshooting

### Container won't start
```bash
# Check logs
docker logs a2n
```

### Database connection issues
```bash
# Verify PostgreSQL is running inside container
docker exec a2n pg_isready
```

### Reset to clean state
```bash
# Warning: This deletes all data
docker stop a2n && docker rm a2n
docker volume rm a2n-data
docker run -d --name a2n -p 8080:8080 -v a2n-data:/data sudoku1016705/a2n:latest
```

---

## 📚 Documentation

- **Website:** [https://a2n.io](https://a2n.io)
- **Documentation:** [https://docs.a2n.io](https://www.a2n.io/tutorials)
- **Discord Community:** [Join Discord](https://discord.gg/mYH3ynfvhr)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

Built with ❤️ using:
- [NestJS](https://nestjs.com/) - Backend framework
- [React](https://reactjs.org/) - Frontend framework
- [React Flow](https://reactflow.dev/) - Workflow canvas
- [PostgreSQL](https://postgresql.org/) - Database
- [Redis](https://redis.io/) - Caching and queues

---

<p align="center">
  <strong>⭐ Star this repo if you find it useful!</strong>
</p>

