# How Syncing Works

Every sync calls the system git and runs a three-step pipeline for each repository:

```text
Source (origin) ──fetch──▶ Relay directory ──push──▶ Backup targets 1..N
                           (~/repo/<name>)
                           ├─ update LFS
                           └─ update submodules
```

## 1. Fetch

- If the relay directory (`{base directory}/{repo name}`) does not exist, the repository is cloned from the source first
- Then `git fetch origin --prune --tags` updates only the origin tracking refs
- **Fetch-only mode**: the working tree is never merged, so uncommitted local changes and merge conflicts cannot interfere — safe to run unattended
- If the source URL changes, it is updated via `remote set-url` automatically

## 2. Update the Relay

- **LFS**: `git lfs fetch --all origin` downloads the LFS objects referenced by any ref; skipped with a note when git-lfs is not installed (see the [FAQ](/en/faq) for a fix)
- **Submodules**: `git submodule update --init --recursive`
- Failures in both steps **degrade to warnings**: refs are already backed up, content may be incomplete, pushing is not blocked

## 3. Push (1-to-Many)

For each backup target:

1. First `git lfs push --all` **pre-uploads every LFS object** (failure degrades to a warning for that target and does not block the push)
2. Then local branches + origin tracking branches (covering branches not checked out locally) + tags are pushed, with `--prune` to force alignment with the source (removing branches / tags that no longer exist there)

- Targets are pushed one after another; **a failing target does not affect the others**, and status is recorded per target
- When a push is rejected with "LFS objects are missing" (a GitLab indexing race), the app waits 5 seconds, re-uploads LFS and retries once
- Mirror repositories get `lfs.locksverify=false`: unattended backups do not fail because someone else locked a file

## Reliability Guarantees

- Every network git command has a timeout (default 1800s, LFS 3600s) and an HTTP low-speed abort (120s stall kills it); timeouts and "Stop Sync" kill the child process
- Repositories sync **serially** (an internal queue) so parallel fetches do not fight over the network
- `GIT_TERMINAL_PROMPT=0` during syncs prevents private repositories from blocking on interactive prompts
- GUI-launched processes often inherit a minimal PATH (especially macOS launched from Finder); the app appends common install locations (Homebrew / MacPorts / Linuxbrew) so git-lfs is not falsely reported as missing
- LFS objects always travel over the HTTPS LFS protocol: even SSH targets negotiate to their HTTPS LFS endpoint; a pre-upload auth failure is recorded as a warning for that target (the push is still attempted) — configure a credential helper or switch to an HTTPS URL
