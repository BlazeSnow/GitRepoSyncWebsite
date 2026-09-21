# 命令行

可执行文件自带命令行接口（不启动窗口），供 shell 脚本与 Agent 使用。

## 运行模式

| 命令 | 说明 |
| --- | --- |
| `git-repo-sync` | 启动图形界面 |
| `git-repo-sync mcp` | 以 stdio 模式运行 MCP 服务（供 Agent 客户端连接） |
| `git-repo-sync --help` | 查看完整帮助 |
| `git-repo-sync --version` | 查看版本号 |

## 选项与环境变量

| 项 | 说明 |
| --- | --- |
| `mcp --api-key <KEY>` | MCP APIKEY（优先级高于环境变量） |
| `GIT_REPO_SYNC_API_KEY=<KEY>` | MCP APIKEY 鉴权 |
| `GIT_REPO_SYNC_LANG=<zh\|en>` | MCP 消息语言（默认 zh） |

示例：

```bash
# 以 stdio 模式运行 MCP 服务
git-repo-sync mcp --api-key grs_xxx
```

## 行为说明

- Windows 下 release 构建为 GUI 子系统（无控制台）：被管道 / 重定向捕获 stdout 时直接输出；在交互 shell 中直接运行 `--help` / `--version` 时自动附加父控制台，输出可见
- MCP 客户端断开后，进程会等待在途同步完成再退出，不中断同步
