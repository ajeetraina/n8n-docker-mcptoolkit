# n8n + Docker MCP Toolkit Bridge

> **Making n8n work with Docker MCP Toolkit through a simple HTTP bridge**

This repository provides a solution for connecting [n8n](https://n8n.io) workflows with [Docker MCP Toolkit](https://docs.docker.com/desktop/features/mcp/), solving the [community integration challenge](https://community.n8n.io/t/n8n-interact-with-docker-mcp-toolkit/135733).

## Problem Solved

**n8n Community Question**: *"How can n8n interact with Docker MCP Toolkit?"*

**Our Solution**: HTTP bridge that translates n8n's HTTP requests into `docker mcp` CLI commands.

```
[n8n Workflow] → HTTP → [Bridge:3001] → docker mcp → [100+ MCP Tools]
```

## Getting Started

### Prerequisite

- Docker Desktop 4.42+ installed
- Enable Docker MCP Toolkit
- GitHub MCP added as Server


### Step 1. Clone and setup


```
git clone https://github.com/ajeetraina/n8n-docker-mcptoolkit
cd n8n-docker-mcptoolkit
```

### Step 2. Start n8n stack

```
docker compose up -d --build
```


### Step 3. Start the bridge

```
npm install
node mcp-http-bridge.js
```

Keep it running.


### Step 4. Test the connection

```
curl http://localhost:3001/health
```

**That's it!** n8n can now access 100+ Docker MCP tools via HTTP requests.


## 🏗️ Architecture: The Missing Bridge

**Why n8n can't directly connect to Docker MCP Toolkit:**

```
❌ What Doesn't Work:
┌─────────┐                    ┌──────────────────┐
│   n8n   │ ──── ❌ ────────▶ │ Docker Desktop   │
│(container)                   │ (host CLI)       │
└─────────┘                    └──────────────────┘
```

**The Solution - HTTP Bridge:**

```
✅ Complete Architecture:
┌─────────┐    HTTP     ┌─────────────┐    CLI      ┌──────────────────┐    API     ┌─────────┐
│   n8n   │ ──────────▶ │HTTP Bridge  │ ──────────▶ │ Docker Desktop   │ ─────────▶ │ GitHub  │
│Workflow │             │   :3001     │             │                  │            │   API   │
└─────────┘             └─────────────┘             │ ┌──────────────┐ │            └─────────┘
                                                    │ │ GitHub MCP   │ │
                                                    │ │ Server       │ │
                                                    │ └──────────────┘ │
                                                    │ ┌──────────────┐ │
                                                    │ │ MCP Toolkit  │ │
                                                    │ │ CLI          │ │
                                                    │ └──────────────┘ │
                                                    └──────────────────┘
```

**Data Flow Example:**
```bash
# 1. n8n makes HTTP request
POST http://localhost:3001/tools/create_repository
{"arguments": {"name": "my-repo", "description": "Created via n8n"}}

# 2. Bridge translates to Docker MCP CLI
docker mcp tools call create_repository '{"name": "my-repo", "description": "Created via n8n"}'

# 3. GitHub MCP Server executes
GitHub API: POST /user/repos {"name": "my-repo", "description": "Created via n8n"}

# 4. Response flows back through bridge to n8n
{"success": true, "data": {"html_url": "https://github.com/user/my-repo"}}
```

