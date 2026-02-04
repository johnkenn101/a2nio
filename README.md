# a2n.io - Workflow Automation & AI Agents Platform

<p align="center">
  <img src="https://a2n.io/logo.png" alt="a2n.io Logo" width="200"/>
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
- 🔗 **30+ Integrations** - Connect to popular services and APIs
- ⚡ **Real-time Execution** - Watch your workflows run with live logs
- 🏠 **Self-Hosted** - Your data stays on your infrastructure

---

## 📦 Quick Start with Docker

### Option 1: Single Command (Recommended)

The easiest way to run a2n.io is with our all-in-one Docker image that includes everything:

```bash
docker run -d \
  --name a2n \
  -p 5000:5000 \
  -v a2n-data:/data \
  sudoku1016705/a2n:latest
```

Then open **http://localhost:5000** in your browser.

### Option 2: Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  a2n:
    image: sudoku1016705/a2n:latest
    container_name: a2n
    ports:
      - "5000:5000"
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
  -p 5000:5000 \
  -e DATABASE_URL="postgresql://user:password@host:5432/a2n" \
  -e REDIS_URL="redis://host:6379" \
  -e ENCRYPTION_KEY="your-32-character-secret-key-here" \
  sudoku1016705/a2n:latest
```

---

## 🔌 Supported Nodes (30+)

### Triggers
| Node | Description |
|------|-------------|
| **Webhook Trigger** | Trigger workflows via HTTP requests |
| **Schedule Trigger** | Run workflows on a cron schedule |
| **Manual Trigger** | Start workflows manually |

### AI & LLM
| Node | Description |
|------|-------------|
| **OpenAI** | GPT-4, GPT-3.5 for text generation, chat, embeddings |
| **Anthropic Claude** | Claude 3 models for advanced AI tasks |
| **Google Gemini** | Gemini Pro for multimodal AI |
| **Groq** | Ultra-fast LLM inference |
| **AI Agent** | Autonomous AI agents with tool use |

### Communication
| Node | Description |
|------|-------------|
| **Email (SMTP)** | Send emails via any SMTP server |
| **Slack** | Send messages to Slack channels |
| **Discord** | Post messages to Discord webhooks |
| **Telegram** | Send messages via Telegram bots |
| **Twilio** | SMS and WhatsApp messaging |

### Data & APIs
| Node | Description |
|------|-------------|
| **HTTP Request** | Make REST API calls to any service |
| **GraphQL** | Execute GraphQL queries and mutations |
| **PostgreSQL** | Query and update PostgreSQL databases |
| **MySQL** | Connect to MySQL databases |
| **MongoDB** | Work with MongoDB collections |
| **Redis** | Cache and retrieve data |

### Social Media
| Node | Description |
|------|-------------|
| **Twitter/X** | Post tweets, read timeline |
| **Facebook** | Post to pages, read data |
| **Reddit** | Create posts, read subreddits |
| **LinkedIn** | Post updates, manage connections |

### Productivity
| Node | Description |
|------|-------------|
| **Google Sheets** | Read/write spreadsheet data |
| **Google Drive** | Upload, download, manage files |
| **Notion** | Create pages, update databases |
| **Airtable** | Work with Airtable bases |

### Utilities
| Node | Description |
|------|-------------|
| **Code (JavaScript)** | Run custom JavaScript code |
| **Code (Python)** | Execute Python scripts |
| **Filter** | Filter data with conditions |
| **Transform** | Map and transform data |
| **Merge** | Combine data from multiple sources |
| **Split** | Split data into batches |
| **Wait** | Add delays between steps |
| **If/Else** | Conditional branching |
| **Switch** | Multi-way branching |
| **Loop** | Iterate over arrays |

### Storage & Files
| Node | Description |
|------|-------------|
| **AWS S3** | Upload/download from S3 buckets |
| **FTP/SFTP** | File transfer protocol operations |
| **Read/Write File** | Local file operations |
| **PDF** | Generate and parse PDFs |
| **CSV** | Parse and generate CSV files |

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

To update to the latest version:

```bash
# Pull latest image
docker pull sudoku1016705/a2n:latest

# Stop and remove old container
docker stop a2n && docker rm a2n

# Start new container (data persists in volume)
docker run -d \
  --name a2n \
  -p 5000:5000 \
  -v a2n-data:/data \
  sudoku1016705/a2n:latest
```

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
docker run -d --name a2n -p 5000:5000 -v a2n-data:/data sudoku1016705/a2n:latest
```

---

## 📚 Documentation

- **Website:** [https://a2n.io](https://a2n.io)
- **Documentation:** [https://docs.a2n.io](https://docs.a2n.io)
- **Discord Community:** [Join Discord](https://discord.gg/a2n)

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
