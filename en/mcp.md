# Agent Access (MCP)

Git Repo Sync exposes repository management to agents as an MCP (Model Context Protocol) server over stdio. Agents run the app's `mcp` mode as a child process; it talks newline-delimited JSON-RPC 2.0 and shares the same SQLite database as the GUI.

## Client Configuration

Copy the API Key from the app's Settings (or MCP) page first, then configure your MCP client (adjust the path per platform):

```json
{
  "mcpServers": {
    "git-repo-sync": {
      "command": "C:\\Program Files\\GitRepoSync\\git-repo-sync.exe",
      "args": ["mcp"],
      "env": { "GIT_REPO_SYNC_API_KEY": "<your API key>" }
    }
  }
}
```

## Authentication

- Provide the API key via the `GIT_REPO_SYNC_API_KEY` environment variable or the `--api-key` argument (the argument wins); it must match the key shown on the Settings page
- You can regenerate the key on the Settings page at any time — update your client configuration afterwards

## Available Tools

| Tool | Description |
| --- | --- |
| `list_repos` | List all repositories with their latest sync status |
| `discover_repos` | Scan the base directory and register existing local repositories (same discovery as the GUI); returns the full list |
| `add_repo` | Add or update a repository (idempotent by name: an existing entry gets its source updated and targets merged, response includes a `created` flag) |
| `update_repo` | Update a repository (rename / source URL; `targets`, when provided, replaces the whole target list) |
| `remove_repo` | Remove a repository (soft delete: hides it and clears its targets) |
| `sync_repo` | Sync one repository immediately (asynchronous) |
| `sync_repos` | Trigger syncs in bulk: an `ids` list or a `days` range (0 = all, N = stale within N days, including never-synced — same semantics as the UI); unconfigured repositories are skipped and reported |
| `get_sync_status` | Query the latest sync status of all repositories |
| `list_logs` | List operation logs newest-first (`limit` optional, default 200, max 1000; read-only, writes no log) |
| `get_base_dir` | Get the local base directory |
| `set_base_dir` | Change the local base directory |

## Tool Semantics

The repository `name` doubles as the relay directory name and the discovery identity, and is unique:

- `add_repo` **merges**: when a repository with the same name (including hidden ones) exists, its source is updated and targets are merged — no duplicates
- `update_repo` **replaces**: `targets`, when provided, replaces the whole list (same as editing in the GUI); use `add_repo` to append targets
- `remove_repo` is a **soft delete**: the entry is hidden and its targets cleared; the directory in the base directory is kept and will not be rediscovered — same semantics as the GUI

## Localization

- Tool descriptions, parameter descriptions and runtime messages follow the `locale` field of the client's `initialize` request
- The `GIT_REPO_SYNC_LANG` (`zh` / `en`) environment variable overrides it
- Chinese is the default when neither is given; tool names are protocol contracts and never localized

## Stability

- The GUI and the MCP subprocess share one SQLite database (WAL + 5s busy_timeout: write conflicts wait instead of erroring)
- A per-request exception is answered as a JSON-RPC internal error; the process keeps running
- After a client disconnects, the process waits for in-flight syncs to finish before exiting
- Each request's method and duration are written to stderr, which helps diagnose disconnects
