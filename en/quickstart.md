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
- **Every other remote is registered as a backup target** (gitee / gitlab / custom names, 1-to-many)
- A repository without any extra remote is marked "unconfigured" until you add one

Afterwards, remotes you add or remove inside the repository follow automatically — no manual list maintenance.

You can also register repositories manually via "Add Repository" without touching the base directory.

## Start Syncing

On the **Sync Repositories** page pick a range (all / stale within 1 / 3 / 7 / 30 days) and click **Start Sync**:

- Each sync pushes to every backup target in turn, each with its own status
- One failing target does not affect the others
- While syncing, click **Stop Sync** to terminate running and queued syncs

Double-click or right-click a table row to edit a repository; the context menu also offers "Sync Now" and "Delete".

## Next Steps

- Learn [how syncing works](/en/sync): the three-step pipeline and how LFS and submodules are handled
- Let an AI agent manage repositories via [Agent Access (MCP)](/en/mcp)
