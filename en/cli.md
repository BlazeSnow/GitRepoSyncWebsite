# Command Line

The executable ships with a command-line interface (no window is opened), for shell scripts and agents.

## Run Modes

| Command | Description |
| --- | --- |
| `git-repo-sync` | Launch the graphical interface |
| `git-repo-sync mcp` | Run the MCP server over stdio (for agent clients) |
| `git-repo-sync --help` | Show the full help |
| `git-repo-sync --version` | Show the version |

## Options & Environment

| Item | Description |
| --- | --- |
| `mcp --api-key <KEY>` | MCP API key (takes precedence over the environment variable) |
| `GIT_REPO_SYNC_API_KEY=<KEY>` | MCP API key authentication |
| `GIT_REPO_SYNC_LANG=<zh\|en>` | MCP message language (default: zh) |

Example:

```bash
# Run the MCP server over stdio
git-repo-sync mcp --api-key grs_xxx
```

## Behavior Notes

- On Windows, release builds are GUI-subsystem (no console): output is written directly when stdout is captured via a pipe or redirection; running `--help` / `--version` from an interactive shell attaches the parent console so the output is visible
- After an MCP client disconnects, the process waits for in-flight syncs to finish before exiting
