# n8n + Docker MCP Toolkit Bridge

> **Making n8n work with Docker MCP Toolkit through a simple HTTP bridge**

This repository provides a solution for connecting [n8n](https://n8n.io) workflows with [Docker MCP Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/toolkit/), solving the [community integration challenge](https://community.n8n.io/t/n8n-interact-with-docker-mcp-toolkit/135733).

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

### Step 1: Download AI Model

Pull your preferred models:

```bash
# Lightweight model (fast, good for testing)
docker model pull ai/llama3.2:1B-Q8_0

# Balanced model (recommended)
docker model pull ai/llama3.2:3B

# More capable model
docker model pull ai/gemma3:2B

# List downloaded models
docker model ls
```


### Step 2. Clone and setup


```
git clone https://github.com/ajeetraina/n8n-docker-mcptoolkit
cd n8n-docker-mcptoolkit
```

### Step 3. Start n8n stack

```
docker compose up -d --build
```


### Step 4. Start the bridge

```
npm install
node mcp-http-bridge.js
```

Keep it running.


### Step 5. Test the connection

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


## Testing the setup

### Test 1: Bridge health

```
curl http://localhost:3001/health
{"status":"MCP HTTP Bridge Ready","port":3001}%
```

### Test 2. Docker MCP Toolkit Integration Test

```
curl http://localhost:3001/test
```

### Results

```
curl http://localhost:3001/test
{"success":true,"data":"Tool call took: 604.375ms\n{\"login\":\"ajeetraina\",\"id\":313480,\"node_id\":\"MDQ6VXNlcjMxMzQ4MA==\",\"avatar_url\":\"https://avatars.githubusercontent.com/u/313480?v=4\",\"html_url\":\"https://github.com/ajeetraina\",\"gravatar_id\":\"\",\"name\":\"Ajeet Singh Raina, Docker Captain, ARM Innovator,\",\"company\":\"Docker Inc\",\"blog\":\"http://www.collabnix.com\",\"location\":\"Bengaluru\",\"email\":\"ajeetraina@gmail.com\",\"hireable\":true,\"bio\":\"Docker Captain, ARM Innovator, Work for Docker Inc, Docker Community Leader, Tip of the Captain's Hat Award Winner, Docker Community Winner @ Dockercon\",\"twitter_username\":\"ajeetsraina\",\"public_repos\":617,\"public_gists\":186,\"followers\":906,\"following\":10,\"created_at\":\"2010-06-24T09:10:23Z\",\"updated_at\":\"2025-06-26T01:55:05Z\",\"type\":\"User\",\"site_admin\"
```

### Test 3: Generic Tool Call

```
curl -X POST http://localhost:3001/tools/get_me \
  -H "Content-Type: application/json" \
  -d '{"arguments": {}}'
```

### Test 4: File Creation Tool

```
curl -X POST http://localhost:3001/tools/create_or_update_file \
  -H "Content-Type: application/json" \
  -d '{
    "arguments": {
      "owner": "your-username",
      "repo": "test-repo", 
      "path": "test.txt",
      "content": "Hello from n8n bridge!",
      "message": "Test from bridge"
    }
  }'
```

