# a2n.io Supported Nodes

This document provides detailed information about all nodes available in a2n.io.

## Table of Contents

- [Triggers](#triggers)
- [AI & LLM Nodes](#ai--llm-nodes)
- [Communication](#communication)
- [Data & Databases](#data--databases)
- [Social Media](#social-media)
- [Productivity](#productivity)
- [Utilities](#utilities)
- [Storage & Files](#storage--files)

---

## Triggers

### Webhook Trigger
Start workflows from external HTTP requests.

**Features:**
- Supports GET, POST, PUT, PATCH, DELETE methods
- Custom authentication (Header, Basic, Bearer Token)
- Automatic URL generation
- Response customization

**Use Cases:**
- Receive form submissions
- GitHub/GitLab webhooks
- Stripe payment notifications
- Custom API endpoints

---

### Schedule Trigger
Run workflows on a schedule using cron expressions.

**Features:**
- Cron expression support
- Timezone configuration
- Multiple schedules per workflow
- Visual cron builder

**Use Cases:**
- Daily reports
- Hourly data sync
- Weekly cleanup tasks
- Monthly billing

---

### Manual Trigger
Start workflows manually with optional input data.

**Features:**
- Custom input schema
- Test with sample data
- Form-based input UI

**Use Cases:**
- On-demand reports
- Manual data processing
- Testing and debugging

---

## AI & LLM Nodes

### OpenAI
Connect to OpenAI's GPT models.

**Supported Models:**
- GPT-4 Turbo
- GPT-4
- GPT-3.5 Turbo
- Text Embeddings

**Features:**
- Chat completions
- Text generation
- Function calling
- JSON mode
- Vision (image analysis)

---

### Anthropic Claude
Access Anthropic's Claude models.

**Supported Models:**
- Claude 3.5 Sonnet
- Claude 3 Opus
- Claude 3 Haiku

**Features:**
- Long context windows (200k tokens)
- System prompts
- Multi-turn conversations
- Tool use

---

### Google Gemini
Use Google's Gemini AI models.

**Supported Models:**
- Gemini Pro
- Gemini Pro Vision

**Features:**
- Text generation
- Multimodal (text + images)
- Embeddings

---

### Grok
Ultra-fast LLM inference with Grok.

**Supported Models:**
- Llama 3
- Mixtral
- Gemma

**Features:**
- Sub-second response times
- OpenAI-compatible API
- Cost-effective

---

### AI Agent
Autonomous AI agents that can use tools.

**Features:**
- Multi-step reasoning
- Tool/function calling
- Memory persistence
- Context management

**Use Cases:**
- Research assistants
- Data analysis
- Customer support automation

---

## Communication

### Email (SMTP)
Send emails via SMTP servers.

**Features:**
- Custom SMTP configuration
- HTML and plain text
- Attachments
- Multiple recipients (To, CC, BCC)
- Templates with variables

---

### Slack
Send messages to Slack workspaces.

**Features:**
- Channel messages
- Direct messages
- Rich formatting (blocks)
- File uploads
- Thread replies

---

### Discord
Post messages to Discord.

**Features:**
- Webhook integration
- Embeds
- File attachments
- Username/avatar customization

---

### Telegram
Send messages via Telegram bots.

**Features:**
- Text messages
- Photos and documents
- Inline keyboards
- Markdown formatting

---

### Twilio
SMS and WhatsApp messaging.

**Features:**
- SMS worldwide
- WhatsApp Business
- Message status tracking
- Media messages

---

## Data & Databases

### HTTP Request
Make HTTP requests to any API.

**Features:**
- All HTTP methods
- Custom headers
- Query parameters
- Request body (JSON, form, raw)
- Authentication (Basic, Bearer, API Key)
- Response parsing

---

### GraphQL
Execute GraphQL operations.

**Features:**
- Queries and mutations
- Variables
- Custom headers
- Introspection

---

### PostgreSQL
Query PostgreSQL databases.

**Features:**
- Raw SQL queries
- Parameterized queries
- Transaction support
- Connection pooling

---

### MySQL
Connect to MySQL databases.

**Features:**
- SQL queries
- Prepared statements
- Multiple result sets

---

### MongoDB
Work with MongoDB databases.

**Features:**
- CRUD operations
- Aggregation pipelines
- Index management
- GridFS support

---

### Redis
Cache and data operations.

**Features:**
- Get/Set/Delete
- Lists, Sets, Hashes
- Pub/Sub
- TTL management

---

## Social Media

### Twitter/X
Post and interact on Twitter.

**Features:**
- Post tweets
- Upload media
- Read timeline
- Search tweets

---

### Facebook
Manage Facebook pages.

**Features:**
- Post to pages
- Read page insights
- Manage comments

---

### Reddit
Interact with Reddit.

**Features:**
- Create posts
- Read subreddits
- Comment on posts
- Vote

---

### LinkedIn
Professional network integration.

**Features:**
- Share posts
- Company updates
- Profile data

---

## Productivity

### Google Sheets
Spreadsheet operations.

**Features:**
- Read/Write cells
- Append rows
- Create sheets
- Formatting

---

### Google Drive
File management.

**Features:**
- Upload/Download files
- Create folders
- Share permissions
- Search files

---

### Notion
Workspace management.

**Features:**
- Create pages
- Update databases
- Query databases
- Block operations

---

### Airtable
Database operations.

**Features:**
- CRUD records
- Filter and sort
- View management

---

## Utilities

### Code (JavaScript)
Execute custom JavaScript.

**Features:**
- Full ES6+ support
- Access to input data
- Return custom output
- npm packages (limited)

```javascript
// Example
const data = $input.all();
return data.map(item => ({
  ...item,
  processed: true,
  timestamp: new Date().toISOString()
}));
```

---

### Code (Python)
Execute Python scripts.

**Features:**
- Python 3.x
- Common libraries (json, datetime, etc.)
- Input/output handling

```python
# Example
import json
data = json.loads(input_data)
data['processed'] = True
return json.dumps(data)
```

---

### Filter
Filter items based on conditions.

**Features:**
- Multiple conditions (AND/OR)
- Comparison operators
- Regular expressions
- Null checks

---

### Transform
Map and transform data.

**Features:**
- Field mapping
- Data type conversion
- Expression evaluation
- Array operations

---

### Merge
Combine data from multiple branches.

**Features:**
- Merge by index
- Merge by key
- Keep all items
- Custom merge logic

---

### Split
Split data into batches.

**Features:**
- Split by count
- Split by field
- Parallel processing

---

### Wait
Add delays between operations.

**Features:**
- Fixed delay
- Random delay
- Wait until time
- Rate limiting

---

### If/Else
Conditional branching.

**Features:**
- Multiple conditions
- Nested conditions
- Default (else) branch

---

### Switch
Multi-way branching.

**Features:**
- Multiple cases
- Default case
- Expression matching
- Regex matching

---

### Loop
Iterate over arrays.

**Features:**
- For each item
- Loop with index
- Break conditions
- Nested loops

---

## Storage & Files

### AWS S3
Amazon S3 operations.

**Features:**
- Upload/Download
- List buckets and objects
- Presigned URLs
- Multipart upload

---

### FTP/SFTP
File transfer operations.

**Features:**
- Upload/Download
- List directories
- Create/Delete folders
- Rename files

---

### Read/Write File
Local file operations.

**Features:**
- Read text/binary
- Write/Append
- Delete files
- File info

---

### PDF
PDF operations.

**Features:**
- Generate from HTML
- Parse text
- Merge PDFs
- Extract pages

---

### CSV
CSV file handling.

**Features:**
- Parse CSV to JSON
- Generate CSV from data
- Custom delimiters
- Header handling

---

## Node Categories Summary

| Category | Node Count |
|----------|-----------|
| Triggers | 3 |
| AI & LLM | 5 |
| Communication | 5 |
| Data & Databases | 6 |
| Social Media | 4 |
| Productivity | 4 |
| Utilities | 10 |
| Storage & Files | 5 |
| **Total** | **42** |

---

## Adding Custom Nodes

Custom node development guide coming soon. In the meantime, you can use the **Code** nodes (JavaScript/Python) for custom logic.

---

## Feature Requests

Have a node request? Open an issue on GitHub or join our Discord community!
