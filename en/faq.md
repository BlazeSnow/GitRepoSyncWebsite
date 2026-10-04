# FAQ

## Git commands fail?

Make sure [Git](https://git-scm.com) is installed and on your PATH; private repositories need credentials configured in advance (an HTTPS credential helper or SSH keys).

Syncs disable git's interactive credential prompts (`GIT_TERMINAL_PROMPT=0`) so unattended runs never hang — credentials must be ready before syncing.

## LFS files are not backed up?

Install [git-lfs](https://git-lfs.com) and sync again. LFS objects are pre-uploaded to every target (`git lfs push --all`); if a target row shows an "LFS pre-upload failed" warning, it is usually a target-side LFS authentication problem (SSH targets go through the HTTPS LFS endpoint) — configure a credential helper or switch to an HTTPS URL.

See [how syncing works](/en/sync).

## A repository shows "unconfigured"?

The repository is missing its `origin` source URL, or has no backup-target remote at all. Add one and it will join syncs:

- Unconfigured repositories do not participate in syncing (skipped by "Start Sync" and rejected by the MCP `sync_repo` tool), avoiding guaranteed "failed" records
- Add a remote inside the repository (`git remote add <name> <url>`) and the app registers it as a backup target automatically

## How do I stop registering a repository?

Move the repository directory out of the base directory — registration simply follows the directories inside it, so auto discovery no longer registers it, and the repository itself is untouched. The app has no delete entry: repository configuration is read-only (since v1.0.0-beta.6), the source and backup targets are derived from `.git/config`, and remotes are managed with git commands.

## How does "Keep me signed in for 30 days" work?

The session token is stored in the database and restored automatically when the app restarts; without it, signing out happens when the app closes.
