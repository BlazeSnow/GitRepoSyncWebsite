# Agent 接入（MCP）

Git Repo Sync 以 MCP（Model Context Protocol）Server 的形式向 Agent 开放仓库查询与同步触发能力，传输方式为 stdio。仓库配置**只读**：源与备份目标由本地仓库的 `.git/config` 派生，配置类工具的调用返回对应的 git 操作指引而非直接修改。Agent 以子进程运行本程序的 `mcp` 模式，与图形界面访问同一个 SQLite 数据库，协议为换行分隔的 JSON-RPC 2.0。

## 客户端配置

先在软件设置页（或 MCP 页）复制 API Key，然后在支持 MCP 的 Agent 客户端中配置（路径按平台调整）：

```json
{
  "mcpServers": {
    "git-repo-sync": {
      "command": "C:\\Program Files\\GitRepoSync\\git-repo-sync.exe",
      "args": ["mcp"],
      "env": { "GIT_REPO_SYNC_API_KEY": "<你的 API Key>" }
    }
  }
}
```

## 鉴权说明

- APIKEY 通过环境变量 `GIT_REPO_SYNC_API_KEY` 或 `--api-key` 参数提供（参数优先级更高），与软件设置页的 API Key 一致方可访问
- 可在设置页随时重新生成 API Key，重新生成后需同步更新客户端配置

## 可用工具

| 工具 | 说明 |
| --- | --- |
| `list_repos` | 列出所有仓库及最近同步状态 |
| `discover_repos` | 扫描基地址内已有本地仓库并登记（与界面自动发现一致），返回登记后的完整列表 |
| `add_repo` | 仓库配置只读：返回自行配置仓库的 git 操作指引（不改任何数据） |
| `update_repo` | 仓库配置只读：返回修改仓库配置的操作指引（不改任何数据） |
| `remove_repo` | 仓库配置只读：返回移除登记的操作指引（不改任何数据） |
| `sync_repo` | 立即同步指定仓库（异步） |
| `sync_repos` | 批量触发同步：`ids` 列表或 `days` 范围（0=全部，N=最近 N 天未同步，含从未同步，与界面范围一致）；未配置仓库自动跳过，按仓库报告是否启动 |
| `stop_syncs` | 停止同步：终止排队与运行中的任务（`ids` 可选，缺省全部活动任务），跨进程生效；返回出队与标记停止的仓库 |
| `get_sync_status` | 查询所有仓库最近同步状态 |
| `list_logs` | 按时间倒序列出操作日志（`limit` 可选，默认 200、上限 1000；只读不写日志） |
| `get_base_dir` | 查询本地仓库基地址 |
| `set_base_dir` | 修改本地仓库基地址 |

## 工具语义

仓库名 `name` 同时是中转目录名与自动发现的身份，全库唯一：

- **仓库配置只读**：`add_repo` / `update_repo` / `remove_repo` 为兼容保留的入口，调用返回对应的 git 操作指引（自行 clone 到 `{基地址}/{仓库名}` 中转目录、用 `git remote add <远端名> <地址>` 添加备份目标等），不改任何数据；同步类工具（`sync_repo` / `sync_repos` / `stop_syncs`）作用于发现登记的仓库
- `upstream` 为保留远端名：自动发现排除（fork 上游不参与同步，避免把 fork 反向推送回原仓库）
- 基地址内的 git 仓库自动登记入列；源地址与远端集合恒与 `.git/config` 对齐，新增远端补为目标、删除的远端移除

## 多语言

- MCP 工具描述、参数描述与运行时消息按 Agent 客户端 `initialize` 请求的 `locale` 字段返回
- 可用环境变量 `GIT_REPO_SYNC_LANG`（`zh` / `en`）强制指定
- 均未提供时默认中文；工具名称为协议契约，不随语言变化

## 稳定性

- GUI 与 MCP 子命令共用同一 SQLite（WAL + 5 秒 busy_timeout，多进程写冲突等待而非报错）
- **同步执行者单例（守护角色）**：GUI 与 MCP 均可发起同步，任务经 SQLite 队列投递、由唯一执行者（数据目录 `sync-daemon.lock` 竞选产生）串行消费，杜绝两条流水线并发操作同一中转目录；停止请求同样经 SQLite 传递，跨进程生效
- 单请求异常被捕获并以 JSON-RPC 内部错误应答，进程不退出
- 客户端断开后进程会等待在途同步完成再退出，不会中断同步
- 每个请求的方法与耗时输出到 stderr，便于排查断连类问题
