---
titleTemplate: 'Git Repo Sync'
description: 'Git Repo Sync：跨平台 Git 仓库同步桌面工具，一次同步，多端备份'
layout: home

hero:
    name: "Git Repo Sync"
    tagline: "跨平台 Git 仓库同步桌面工具：把本地基地址作为中转站，自动将仓库从源远端备份到你配置的多个目标远端——一次同步，多端备份"
    actions:
    - theme: brand
      text: 快速开始
      link: /quickstart
    - theme: alt
      text: 前往下载
      link: https://github.com/BlazeSnow/GitRepoSync/releases

features:
  - icon: 🔄
    title: 1 对多备份
    details: origin 为主源，其余远端（gitee / gitlab / 自命名均可）全部作为备份目标，一次同步依次推送全部远端
  - icon: 🔍
    title: 自动发现
    details: 基地址内的仓库自动登记，新增 / 删除远端自动同步到备份列表，无需手工维护
  - icon: 🕒
    title: 定时提醒式同步
    details: 按「最近同步时间」圈定范围（1 / 3 / 7 / 30 天未同步），一键补齐长期未备份的仓库
  - icon: 📦
    title: LFS 与 submodule 支持
    details: 完整备份大文件与子模块，未安装 git-lfs 时自动降级并提示
  - icon: 📜
    title: 操作日志
    details: 登录、同步、配置变更等全部操作入库 SQLite，按时间倒序可查
  - icon: 🌐
    title: 中英双语 / 深色模式
    details: 界面语言可切换（简体中文 / English），深浅色跟随系统或手动指定
  - icon: 🤖
    title: Agent 接入
    details: 内置 MCP 服务，Agent 可直接管理仓库并触发同步
  - icon: 🖥️
    title: 跨平台
    details: 支持 Windows、macOS（Apple Silicon / Intel）与 Linux
---

## 许可证

软件使用 GNU AGPL v3.0 协议。
