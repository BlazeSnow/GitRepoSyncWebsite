# 同步原理

每次同步调用系统 git，对每个仓库执行三步流水线：

```text
源远端(origin) ──拉取──▶ 基地址中转目录 ──推送──▶ 备份目标 1..N
                         (~/repo/仓库名)
                        ├─ 更新 LFS
                        └─ 更新 submodule
```

## 1. 拉取

- 中转目录（`{基地址}/{仓库名}`）不存在时先 `git clone <源地址>`
- 随后 `git fetch origin --prune --tags` 只更新 origin 跟踪引用
- **fetch-only 模式**：不合并工作区，不受本地未提交改动或合并冲突影响，无人值守安全
- 源地址变更时自动 `remote set-url`

## 2. 更新中转站

- **LFS**：`git lfs fetch --all origin` 下载所有引用指向的 LFS 对象；未安装 git-lfs 时跳过并注明（可在[常见问题](/faq)查看解决办法）
- **submodule**：`git submodule update --init --recursive`
- 两步失败**降级为警告**：引用已备份、内容可能不完整，不阻断推送

## 3. 推送（1 对多）

对每个备份目标：

1. 先 `git lfs push --all` **预上传全部 LFS 对象**（失败降级为该目标的警告，不阻断推送）
2. 再依次推送本地分支 + origin 跟踪分支（补全本地未 checkout 的分支）+ 标签，`--prune` 强制与来源对齐（删除来源已不存在的分支 / 标签）

- 多个目标依次推送，**单个目标失败不影响其余目标**，状态按目标独立记录
- 推送被 "LFS objects are missing" 拒绝时（GitLab 索引滞后竞态）等待 5 秒、重传 LFS 后自动重试一次
- 镜像仓库写入 `lfs.locksverify=false`：无人值守备份不因文件被他人加锁而失败

## 可靠性保障

- 每个网络 git 命令带超时（默认 1800s，LFS 3600s）与 HTTP 低速中断（停滞 120s 判死），超时或「停止同步」时强杀子进程
- 多个仓库**串行**同步（内部队列依次执行），避免并发拉取抢占网络
- 同步期间 `GIT_TERMINAL_PROMPT=0`，避免私有仓库卡在交互式输入
- 图形界面启动时继承的 PATH 可能缺少 Homebrew 等用户级安装目录（尤其 macOS 从 Finder 启动），软件会自动补充常见安装目录，避免 git-lfs 误报未安装
- LFS 对象统一走 HTTPS LFS 协议上传：目标地址即使为 SSH，git-lfs 也会自动协商到对应 HTTPS LFS 端点；预上传认证失败会记录为该目标的警告（推送仍尝试），此时需配置凭据助手或改用 HTTPS 目标地址
