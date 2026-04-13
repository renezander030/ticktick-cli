# @reneza/ticktick-cli

> **Disclaimer:** This is an unofficial, community-maintained project and is not affiliated with, endorsed by, or connected to TickTick or Appest. "TickTick" is a trademark of Appest. This tool uses the official [TickTick Open API](https://developer.ticktick.com/).

TickTick task management for Claude Code, Claude Desktop, and the terminal. 17 MCP tools for creating, searching, completing, and organizing tasks — plus optional semantic vector search via Qdrant + Ollama.

**Fork of [kvanland/ticktick-cli](https://github.com/kvanland/ticktick-cli)** with semantic search, completed tasks filtering, and Claude Code plugin marketplace support.

## Install

```bash
npm install -g @reneza/ticktick-cli
ticktick setup
```

The setup wizard walks you through creating an app at [developer.ticktick.com](https://developer.ticktick.com/) and completing OAuth.

## MCP Server — Claude Code

```bash
claude mcp add ticktick ticktick-mcp
```

Or add to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "ticktick": {
      "command": "ticktick-mcp"
    }
  }
}
```

### Claude Code Plugin Marketplace

```bash
/plugin marketplace add ticktick-cli https://github.com/renezander030/ticktick-cli
/plugin install ticktick@ticktick-cli
```

Registers the MCP server automatically — no manual config needed.

## MCP Server — Claude Desktop

Add to your Claude Desktop config:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
**Linux:** `~/.config/claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "ticktick": {
      "command": "ticktick-mcp"
    }
  }
}
```

## MCP Tools

| Tool | Description |
|------|-------------|
| `ticktick_auth_status` | Check authentication status |
| `ticktick_projects_list` | List all projects |
| `ticktick_projects_get` | Get project with tasks |
| `ticktick_tasks_list` | List tasks in project |
| `ticktick_tasks_get` | Get task details |
| `ticktick_tasks_create` | Create a new task |
| `ticktick_tasks_update` | Update an existing task |
| `ticktick_tasks_complete` | Mark task as complete |
| `ticktick_tasks_delete` | Delete a task |
| `ticktick_tasks_search` | Search by keyword, tags, or priority |
| `ticktick_tasks_semantic_search` | Semantic search via vector similarity |
| `ticktick_tasks_similar` | Find semantically similar tasks |
| `ticktick_tasks_due` | Get tasks due within N days |
| `ticktick_tasks_priority` | Get high priority tasks |
| `ticktick_tasks_completed` | List completed tasks within a date range |
| `ticktick_vector_sync` | Sync tasks into vector index |
| `ticktick_vector_status` | Check vector index health |

**Example prompts:**
- "What tasks do I have due this week?"
- "Create a task to buy groceries tomorrow with high priority"
- "Find tasks related to deployment"
- "Show me what I completed last week"

## CLI Usage

```bash
# Tasks
ticktick tasks create "Buy groceries" --due 2026-01-30 --priority high --tags "shopping"
ticktick tasks due 3                    # Due in 3 days
ticktick tasks search "meeting"         # Keyword search
ticktick tasks search --tags "work"
ticktick tasks completed                # Last 7 days
ticktick tasks completed --start 2026-01-01 --end 2026-01-31

# Projects
ticktick projects list
ticktick projects create "Name" --color "#ff6b6b"

# Auth
ticktick auth status
ticktick auth login
```

All IDs are displayed as 8-character short IDs. Use `--format json` for machine-readable output.

## Semantic Vector Search (Optional)

Keyword search fires N+1 API calls per query and only does substring matching. Vector search queries a local Qdrant index in <100ms and finds semantically related results ("deployment" matches "push release to prod").

Entirely optional — if Qdrant/Ollama aren't running, semantic search falls back to keyword search automatically.

### Setup

```bash
# Start Qdrant and Ollama (Docker)
docker run -d --name qdrant -p 6333:6333 qdrant/qdrant
docker run -d --name ollama -p 11434:11434 ollama/ollama
docker exec ollama ollama pull nomic-embed-text

# Build the index
ticktick tasks vector-sync

# Search
ticktick tasks semantic "anything about deployments"
ticktick tasks similar TASK_ID
```

Sync is incremental — only re-embeds tasks whose content changed. Set up a cron job for automated updates:

```bash
0 */4 * * * ticktick tasks vector-sync --format json >> /var/log/ticktick-vector-sync.log 2>&1
```

## Configuration

| File | Purpose |
|------|---------|
| `~/.config/ticktick/config.json` | Client credentials |
| `~/.config/ticktick/tokens.json` | OAuth tokens (auto-managed) |

Environment variables: `TICKTICK_CLIENT_ID`, `TICKTICK_CLIENT_SECRET`, `QDRANT_URL`, `OLLAMA_URL`, `EMBEDDING_MODEL`

## License

MIT — see [LICENSE](LICENSE). Originally created by [kvanland](https://github.com/kvanland/ticktick-cli).
