---
titleTemplate: 'Git Repo Sync'
description: 'Git Repo Sync: a cross-platform Git repository sync desktop tool — sync once, back up everywhere'
layout: home

hero:
    name: "Git Repo Sync"
    tagline: "A cross-platform Git repository sync desktop tool: use a local base directory as a relay, and automatically back up repositories from the source remote to every target remote you configure — sync once, back up everywhere"
    actions:
    - theme: brand
      text: Quick Start
      link: /en/quickstart
    - theme: alt
      text: Download
      link: https://github.com/BlazeSnow/GitRepoSync/releases

features:
  - icon: 🔄
    title: 1-to-Many Backup
    details: origin is the source; every other remote (gitee / gitlab / custom names) becomes a backup target, all pushed in one sync
  - icon: 🔍
    title: Auto Discovery
    details: Repositories in the base directory are registered automatically; remotes you add or remove stay in sync with the backup list
  - icon: 🕒
    title: Stale-Reminder Sync
    details: Pick a range by last sync time (1 / 3 / 7 / 30 days stale) and catch up on long-unbacked repositories with one click
  - icon: 📦
    title: LFS & Submodules
    details: Fully back up large files and submodules; degrades gracefully with a hint when git-lfs is not installed
  - icon: 📜
    title: Operation Logs
    details: Logins, syncs and configuration changes are all stored in SQLite and browsable in reverse chronological order
  - icon: 🌐
    title: Bilingual / Dark Mode
    details: UI language switchable (简体中文 / English); dark mode follows the system or can be set manually
  - icon: 🤖
    title: Agent Access
    details: Built-in MCP server lets AI agents manage repositories and trigger syncs directly
  - icon: 🖥️
    title: Cross-Platform
    details: Available on Windows, macOS (Apple Silicon / Intel) and Linux
---

## License

This software is licensed under the GNU AGPL v3.0.
