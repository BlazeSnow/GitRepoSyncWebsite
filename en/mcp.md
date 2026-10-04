# Agent Access (MCP)

Git Repo Sync exposes repository inspection and sync triggering to agents as an MCP (Model Context Protocol) server over stdio. Repository configuration is **read-only**: the source and backup targets are derived from each local repository's `.git/config`, and configuration tools answer with git guidance instead of modifying anything. Agents run the app's `mcp` mode as a child process; it talks newline-delimited JSON-RPC 2.0 and shares the same SQLite database as the GUI.

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
| `add_repo` | Repository configuration is read-only: returns git guidance for configuring a repository yourself (changes nothing) |
| `update_repo` | Repository configuration is read-only: returns git guidance for changing a repository's configuration (changes nothing) |
| `remove_repo` | Repository configuration is read-only: returns git guidance for removing a registration (changes nothing) |
| `sync_repo` | Sync one repository immediately (asynchronous) |
| `sync_repos` | Trigger syncs in bulk: an `ids` list or a `days` range (0 = all, N = stale within N days, including never-synced — same semantics as the UI); unconfigured repositories are skipped and reported |
| `stop_syncs` | Stop syncing: terminate queued and running tasks (`ids` optional, defaults to all active tasks), effective across processes; returns the dequeued and stop-flagged repositories |
| `get_sync_status` | Query the latest sync status of all repositories |
| `list_logs` | List operation logs newest-first (`limit` optional, default 200, max 1000; read-only, writes no log) |
| `get_base_dir` | Get the local base directory |
| `set_base_dir` | Change the local base directory |

## Tool Semantics

The repository `name` doubles as the relay directory name and the discovery identity, and is unique:

- **Repository configuration is read-only**: `add_repo` / `update_repo` / `remove_repo` are kept for compatibility and answer with git guidance (clone into the `{base directory}/{repo name}` relay directory yourself, add backup targets with `git remote add <remote> <url>`, etc.) without changing anything; the sync tools (`sync_repo` / `sync_repos` / `stop_syncs`) operate on registered repositories
- `upstream` is a reserved remote name: auto discovery excludes it (a fork's upstream never participates in syncing, so a fork is never pushed back to its origin)
- Git repositories inside the base directory are registered automatically; the source URL and the remote set always align with `.git/config` — new remotes become targets, removed ones are dropped

## Localization

- Tool descriptions, parameter descriptions and runtime messages follow the `locale` field of the client's `initialize` request
- The `GIT_REPO_SYNC_LANG` (`zh` / `en`) environment variable overrides it
- Chinese is the default when neither is given; tool names are protocol contracts and never localized

## Stability

- The GUI and the MCP subprocess share one SQLite database (WAL + 5s busy_timeout: write conflicts wait instead of erroring)
- **Single sync executor (daemon role)**: both the GUI and MCP can start syncs; tasks are delivered through a SQLite queue and consumed serially by the one executor (elected via a `sync-daemon.lock` file in the data directory), so two pipelines never operate on the same relay directory concurrently; stop requests travel through SQLite too and take effect across processes
- A per-request exception is answered as a JSON-RPC internal error; the process keeps running
- After a client disconnects, the process waits for in-flight syncs to finish before exiting
- Each request's method and duration are written to stderr, which helps diagnose disconnects
