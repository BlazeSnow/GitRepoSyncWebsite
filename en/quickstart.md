# Quick Start

## Install

Download the installer for your platform from [GitHub Releases](https://github.com/BlazeSnow/GitRepoSync/releases/latest):

| Platform            | Package                                      |
| ------------------- | -------------------------------------------- |
| Windows x64         | `GitRepoSync_<version>_x64-setup.exe` / `.msi` |
| macOS Apple Silicon | `GitRepoSync_<version>_aarch64.dmg`          |
| macOS Intel         | `GitRepoSync_<version>_x64.dmg`              |
| Linux x64           | `.deb` / `.rpm` / `.AppImage`                |

Versions in the form `vX.Y.Z-beta.N` are betas: more frequent updates, but potentially less stable.

## Sign In

The app ships with a built-in account:

- Username: `admin`
- Password: `admin123`

It is created automatically on first launch. Tick "Keep me signed in for 30 days" to stay logged in across restarts, and change the password on the Settings page as soon as possible.

## Set the Base Directory

The base directory is the parent of the local relay directories (defaults to a `repo` folder in your user home):

1. Open the **Settings** page
2. Click **Choose Directory…** to pick a folder with the system directory picker

Every repository lives at `{base directory}/{repo name}` and acts as the relay for syncing.

## Add Repositories

Clone or move the repositories you want to back up into the base directory — the app **discovers and registers them automatically** on launch:

- The `origin` remote becomes the **source**
- **Every other remote is registered as a backup target** (gitee / gitlab / custom names, 1-to-many; the reserved name `upstream` is excluded, so a fork's upstream remote is never pushed back to)
- A repository without any extra remote is marked "unconfigured" until you add one

Repository configuration is **read-only**: the source and backup targets are derived entirely from `.git/config`, and remotes are managed by you with git — run `git remote add <name> <url>` / `git remote remove <name>` inside the repository and the backup list follows automatically; the app never manages repository configuration for you.

## Start Syncing

On the **Sync Repositories** page pick a range (all / configured / stale within 1 / 3 / 7 / 30 days) and click **Start Sync**:

- Each sync pushes to every backup target in turn, each with its own status
- One failing target does not affect the others
- While syncing, click **Stop Sync** to terminate running and queued syncs (effective across processes)

Double-click or right-click a table row to open a **read-only detail dialog** (the source and backup-target list — to change them, run git commands yourself); the context menu also offers "Open Folder" and "Sync Now".

## Next Steps

- Learn [how syncing works](/en/sync): the three-step pipeline and how LFS and submodules are handled
- Let an AI agent inspect repositories and trigger or stop syncs via [Agent Access (MCP)](/en/mcp)
