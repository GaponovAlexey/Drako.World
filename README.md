<div align="center">

# 🐉 Drako World

**AI-native habit, task, note & mood tracker — powered by Model Context Protocol**

[![MCP Compatible](https://img.shields.io/badge/MCP-Compatible-6366f1?style=flat-square&logo=anthropic)](https://modelcontextprotocol.io)
[![Rust Backend](https://img.shields.io/badge/Backend-Rust%20%2B%20Axum-orange?style=flat-square&logo=rust)](https://www.rust-lang.org)
[![Python MCP](https://img.shields.io/badge/MCP%20Server-Python%20%2B%20FastMCP-3776AB?style=flat-square&logo=python)](https://github.com/jlowin/fastmcp)
[![Docker](https://img.shields.io/badge/Deploy-Docker-2496ED?style=flat-square&logo=docker)](https://hub.docker.com/r/drakoworld/drako-mcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

[What is Drako World?](#what-is-drako-world) · [MCP Tools](#-mcp-tools) · [Quick Start](#-quick-start) · [Architecture](#-architecture) · [API Keys](#-api-keys) · [Self-hosting](#-self-hosting)

</div>

---

<!-- SCREENSHOT: app overview / hero image -->
<!-- ![Drako World App](assets/screenshot-hero.png) -->

---

## What is Drako World?

**Drako World** is a productivity ecosystem that lets AI agents — Claude, GPT, Cursor, or any MCP-compatible agent — directly manage your habits, tasks, notes, and daily mood tracking through the **Model Context Protocol**.

You stay in control. Each integration uses a scoped API key you generate and can revoke at any time. The AI can create tasks, log your mood, or search your notes — but only within the boundaries you define.

```
You talk to your AI agent  →  Agent calls Drako MCP tools  →  Your data is updated
```

**What the AI can do on your behalf:**
- ✅ Create tasks and break them into subtasks
- ✅ Create and manage habits (daily / weekly / monthly)
- ✅ Log your daily mood with a 1–10 star rating and notes
- ✅ Create and search through your notes
- ✅ Read your current tasks, habits, and mood history

**What the AI cannot do:**
- ❌ Access other users' data
- ❌ Use a revoked or expired API key
- ❌ Perform any action outside the granted scope

---

## 🛠 MCP Tools

Drako World exposes **8 tools** to any MCP-compatible agent:

| Tool | Description |
|------|-------------|
| `drako_create_task` | Create a task with optional subtasks and deadline |
| `drako_get_tasks` | Retrieve all your tasks |
| `drako_create_habit` | Create a habit (daily / weekly / monthly / custom days) |
| `drako_get_habits` | Retrieve all your habits with current progress |
| `drako_set_mood` | Set a star rating and note for any day |
| `drako_get_moods` | Retrieve your mood history |
| `drako_create_note` | Create a note with tag and color |
| `drako_search_notes` | Full-text search across your notes |

### What agents can do with these tools

**Daily planning:** Your agent reads today's habits and tasks, helps you prioritize, creates new tasks from your conversation, and logs a mood entry at the end of the day — all without leaving your chat.

**Weekly review:** Ask your agent to summarize your habit completion for the week, identify patterns in your mood history, and create next week's tasks from your notes.

**Voice-to-task:** Dictate ideas to your AI assistant — it creates structured tasks with subtasks, saves key decisions as notes, and sets up recurring habits you mentioned.

```python
# Example: agent-driven morning planning session

drako_get_habits()
# → returns all active habits with today's completion status

drako_get_tasks()
# → returns open tasks sorted by deadline

drako_create_task(
    task_name="Prepare Q1 report",
    subtasks=["Gather data", "Write draft", "Review with team"],
    deadline="2026-03-01"
)

drako_set_mood(
    day_rate="2026-02-19",
    star=8,
    message="Good focus session, completed 3 of 4 habits"
)
```

### Tool input/output

Each tool accepts a JSON object and returns a structured JSON response. All fields are documented in the [MCP server source](mcp/).

**`drako_create_task`**
```json
// Input
{
  "task_name": "string",
  "description": "string (optional)",
  "subtasks": ["string", "..."],
  "deadline": "YYYY-MM-DD (optional)"
}

// Output
{
  "task_id": 42,
  "created": true
}
```

**`drako_set_mood`**
```json
// Input
{
  "day_rate": "YYYY-MM-DD",
  "star": 8,
  "message": "string (optional)",
  "category": "mood (default) | work | relationships | ..."
}

// Output
{
  "rate_day_id": 17,
  "updated": true
}
```

**`drako_search_notes`**
```json
// Input
{
  "query": "string",
  "limit": 10
}

// Output
{
  "notes": [
    { "note_id": 5, "title": "...", "content": "...", "tag": "...", "created_at": "..." }
  ]
}
```

---

## ⚡ Quick Start

### 1. Get your API key

1. Open the Drako World app
2. Go to **Settings → API Keys**
3. Click **Generate new key** — give it a name like `"DW_api"`
4. Copy the key — **it is shown only once**

### 2. Connect to Claude Code

Add to `~/.claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "drako": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "DRAKO_AUTH_TOKEN=drk_your_key_here",
        "-e", "BACKEND_RPC_URL=https://api.drako.world/api/rpc",
        "drakoworld/drako-mcp:latest"
      ]
    }
  }
}
```

Or with Python (no Docker):

```json
{
  "mcpServers": {
    "drako": {
      "command": "python",
      "args": ["main.py"],
      "cwd": "/path/to/drako-mcp",
      "env": {
        "DRAKO_AUTH_TOKEN": "drk_your_key_here",
        "BACKEND_RPC_URL": "https://api.drako.world/api/rpc"
      }
    }
  }
}
```

### 3. Connect to Cursor / Windsurf / Zed

For editors that support MCP via HTTP/SSE:

```
URL:    http://localhost:8080/sse
Header: Authorization: drk_your_key_here
```

Start the MCP server in SSE mode first:

```bash
docker run -p 8080:8080 \
  -e DRAKO_AUTH_TOKEN=drk_your_key_here \
  -e BACKEND_RPC_URL=https://api.drako.world/api/rpc \
  -e MCP_MODE=sse \
  drakoworld/drako-mcp:latest
```

### 4. Verify the connection

Ask your agent:

```
"What habits do I have in Drako?"
```

If the agent lists your habits — you're connected.

---

## 🏗 Architecture

<!-- SCREENSHOT: architecture diagram -->
<!-- ![Architecture](assets/architecture.png) -->

```
┌─────────────────────────────────────────────────────────┐
│                   AI Agent Layer                        │
│         Claude Code · Cursor · GPT Actions              │
│         Any MCP-compatible client                       │
└────────────────────────┬────────────────────────────────┘
                         │  stdio (local) or HTTP/SSE (remote)
                         ▼
┌─────────────────────────────────────────────────────────┐
│                Drako MCP Server                         │
│              Python · FastMCP Framework                 │
│                                                         │
│   drako_create_task    drako_get_tasks                  │
│   drako_create_habit   drako_get_habits                 │
│   drako_set_mood       drako_get_moods                  │
│   drako_create_note    drako_search_notes               │
└────────────────────────┬────────────────────────────────┘
                         │  HTTP POST + drk_ token
                         ▼
┌─────────────────────────────────────────────────────────┐
│               Drako Backend (Rust · Axum)               │
│                    /api/rpc endpoint                    │
│                                                         │
│   • Validates drk_ API key (HMAC-SHA256)                │
│   • Resolves user_id from key                           │
│   • Executes business logic on PostgreSQL               │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
              PostgreSQL · habits · tasks
              notes · rate_days · drako_api_keys
```

### Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Rust + Dioxus + WebAssembly |
| Backend | Rust + Axum + SQLx |
| Database | PostgreSQL |
| MCP Server | Python 3.12 + FastMCP |
| Auth | HMAC-SHA256 hashed `drk_` tokens |
| Deploy | Docker |

### MCP transport modes

**stdio** — runs as a subprocess of the agent. Best for local use with Claude Code, Cursor, Zed. The agent starts and stops the server automatically.

**HTTP/SSE** — runs as a persistent server on a port. Best for remote agents, GPT Actions, or when you want one server shared across multiple tools.

### Request flow

1. Agent calls a Drako tool via MCP protocol
2. MCP server reads `DRAKO_AUTH_TOKEN` from environment
3. Server sends HTTP POST to `/api/rpc` with the method and params
4. Backend extracts the `drk_` key from the request header
5. Backend looks up `key_prefix` (first 8 chars) in the `drako_api_keys` table
6. Computes `HMAC-SHA256(key)` and compares with the stored hash
7. Checks `is_active = true` and `expires_at > now()`
8. Resolves `user_id` → creates authenticated request context
9. Executes the method → returns JSON
10. MCP server forwards the result back to the agent

---

## 🔑 API Keys

API keys give AI agents scoped, revocable access to your Drako data.

### Key format

```
drk_a7f3k9x2m1p8q4r6t0w5y
└──┘ └─────────────────────┘
prefix   cryptographically random token
```

### Key lifecycle

```
Generate  →  Copy once  →  Add to agent config  →  In use  →  Revoke anytime
```

The full key is shown **only at creation time**. The backend stores only:

| Field | What it is |
|-------|-----------|
| `key_prefix` | First 8 characters — used for fast DB lookup |
| `key_hash` | HMAC-SHA256 of the full key — never reversible |
| `name` | Human label you assigned (`"DW_api"`, `"cursor"`) |
| `expires_at` | Optional TTL — key stops working automatically |
| `last_used_at` | Updated on every successful request |

Your raw key never touches the database. Even a full DB leak cannot recover it.

### Managing keys

| Action | How |
|--------|-----|
| Generate | Settings → API Keys → Generate |
| List active keys | Settings → API Keys |
| Revoke | Settings → API Keys → Revoke |

You can have multiple keys — one per agent, one per device. Revoke individually without affecting others.

---

## 🐳 Self-hosting

> Self-hosting documentation is coming soon.
> The MCP server Docker image is available at `drakoworld/drako-mcp`.

### MCP server (Docker)

```bash
docker run -p 8080:8080 \
  -e DRAKO_AUTH_TOKEN=drk_your_key_here \
  -e BACKEND_RPC_URL=https://api.drako.world/api/rpc \
  drakoworld/drako-mcp:latest
```

### Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DRAKO_AUTH_TOKEN` | ✅ | Your `drk_` API key |
| `BACKEND_RPC_URL` | ✅ | Drako backend RPC endpoint URL |
| `MCP_PORT` | — | Port for HTTP/SSE mode (default: `8080`) |
| `MCP_MODE` | — | `stdio` (default) or `sse` |

---

## 📄 License

MIT — see [LICENSE](LICENSE)

---

<div align="center">

Built with 🦀 Rust · 🐍 Python · 🐉 Drako

[drako.world](https://drako.world)

</div>
