# OpenClaw DevKit 详细参考手册 (Reference Manual)

本手册提供了 OpenClaw 开发工具箱套件的深度技术细节，补充了 `README.md` 中的精简信息。

---

## 🛠️ 运维命令手册 (Maintenance Manual)

| 命令分类       | 命令                  | 说明                                              |
| :------------- | :-------------------- | :------------------------------------------------ |
| **生命周期**   | `make up`             | 启动所有开发容器 (后台运行)                       |
|                | `make down`           | 停止并移除所有容器                                |
|                | `make install`        | **标准版**初始化 (检查环境、修复权限、构建)       |
|                | `make install office` | **Office 办公版**初始化                           |
|                | `make install java`   | **Java 增强版**初始化                             |
|                | `make restart`        | 重新启动所有服务                                  |
|                | `make status`         | 查看容器健康状态、镜像版本与访问地址              |
| **构建与更新** | `make build`          | 手动构建标准版镜像                                |
|                | `make build-java`     | 手动构建 Java 增强版镜像                          |
|                | `make build-office`   | 手动构建 Office 办公版镜像                        |
|                | `make rebuild`        | 重建标准镜像 + 重启服务                           |
|                | `make rebuild-java`   | 重建 Java 镜像 + 重启服务                         |
|                | `make rebuild-office` | 重建 Office 镜像 + 重启服务                       |
|                | `make update`         | 从 GitHub Release 自动化拉取最新源码              |
| **调试与诊断** | `make logs`           | 追踪 Gateway 主服务日志                           |
|                | `make logs-all`       | 追踪所有容器的日志                                |
|                | `make shell`          | 进入 Gateway 容器交互环境 (bash)                  |
|                | `make pairing`        | **频道配对** (如 `make pairing CMD="list slack"`) |
|                | `make test-proxy`     | **一键连通性测试** (Google/Claude API)            |
|                | `make gateway-health` | 深度检查网关 API 响应状态                         |
| **配置与备份** | `make backup-config`  | 备份所有 Agent 与全局配置 (`~/.openclaw-backups`) |
|                | `make restore-config` | 交互式恢复备份文件                                |
| **清理**       | `make clean`          | 清理孤儿容器与悬空镜像 (释放磁盘)                 |
|                | `make clean-volumes`  | **警告**: 清理所有持久化数据卷 (删除缓存数据)     |

---

## ⚙️ 环境变量详细说明 (Configuration Details)

在项目根目录的 `.env` 文件中定义：

| 变量名                  | 默认值         | 说明                                      |
| :---------------------- | :------------- | :---------------------------------------- |
| `OPENCLAW_CONFIG_DIR`   | `~/.openclaw`  | 宿主机配置映射路径                        |
| `OPENCLAW_IMAGE`        | `openclaw:dev` | 运行镜像版本                              |
| `HTTP_PROXY`            | -              | 容器内部使用的 HTTP 代理地址              |
| `HTTPS_PROXY`           | -              | 容器内部使用的 HTTPS 代理地址             |
| `SLACK_BOT_TOKEN`       | -              | Slack Bot 令牌 (xoxb格式)                 |
| `SLACK_APP_TOKEN`       | -              | Slack App 令牌 (xapp/Socket Mode)         |
| `SLACK_PRIMARY_OWNER`   | -              | 控制高权限指令的主要管理员 ID             |
| `OPENCLAW_GATEWAY_PORT` | `18789`        | Gateway Web 端访问端口                    |
| `GITHUB_TOKEN`          | -              | 提高构建/更新时访问 GitHub API 的速率限制 |

---

## 📂 目录结构详解 (Directory Structure)

| 路径                  | 详细用途                                         |
| :-------------------- | :----------------------------------------------- |
| `Makefile`            | 核心运维入口，封装了所有复杂指令                 |
| `docker-compose.yml`  | 定义容器编排、网络、数据卷挂载逻辑               |
| `Dockerfile`          | 标准版环境：集成 Go, Node, Python, Playwright 等 |
| `Dockerfile.java`     | Java 增强版：追加 JDK 25, Gradle, Maven 等       |
| `.openclaw_src/`      | 存储 OpenClaw 核心源码，由 `make update` 管理    |
| `docker-dev-setup.sh` | 初始化脚本，处理文件夹预建及权限纠正             |
| `update-source.sh`    | 增量版本同步工具                                 |
| `.env.example`        | 配置模板文件                                     |
| `docs/`               | 存放架构图等项目资产                             |
| `CLAUDE.md`           | 给 AI 助手的项目规范指引                         |
| `slack-manifest.json` | Slack App 快速导入配置单                         |

---
## 🤖 高级多智能体配置 (Advanced Multi-Agent Patterns)

OpenClaw 支持基于 **Commander-Worker (指挥官-执行者)** 模型的高级协作模式，适用于复杂的软件研发全生命周期。

### 1. 指挥官-执行者架构
*   **指挥官 (Maintainer)**: 唯一与用户交互的入口。负责分解目标、调度 Worker、审查成果并进行 PR 合并。
*   **专用执行者 (Specialized Workers)**: 
    *   `scout`: 负责项目环境勘察与上游同步验证。
    *   `developer`: 负责功能开发与原子化提交 (Commits)。
    *   `reviewer`: 负责代码规范审查与 PR 描述校验。
    *   `tester`: 负责本地测试执行。
    *   `github-ops`: 负责 `gh` CLI 操作与 PR 创建。

### 2. 安全隔离与权限控制
建议为各 Agent 实施 **最小权限原则 (Least Privilege)**：
*   **工具隔离**: 通过 `tools.allow` 限制行为（例如：禁止 `scout` 进行 `write` 操作）。
*   **空间隔离**: 为每个 Agent 设置独立的 `workspace` 路径，避免 Git 指针冲突。
*   **运行隔离**: 为开发者角色开启 `sandbox.mode: "all"`，在 Docker 容器内进行代码实验。

### 3. 项目感知与模板遵循 (Adherence)
配置 Agent 规则以提升专业度：
*   **项目规则**: 要求 `scout` 发现并让团队遵循项目根目录的 `AGENTS.md` 或 `CLAUDE.md`。
*   **GitHub 模板**: 要求 `main` 与 `github-ops` 在创建 Issue/PR 前检索 `.github/` 下的模板。

### 4. 长期记忆系统 (Memory System)
通过在 `identity.rules` 中注入指令，让 Agent 定期更新工作区下的 `memory.md`，确保长周期任务的上下文连续性。

---

## 🔁 核心协作逻辑 (Core Collaboration)

1. **Makefile (入口)** -> **docker-dev-setup.sh (初始化)** -> **Dockerfile (环境构建)**。
2. **缓存优化**: `node_modules` 与 Go 缓存通过 `Named Volumes` 管理，即使删除容器，二次构建依然极速。
3. **权限安全**: 镜像内部使用 `node` 用户 (UID 1000)运行，所有卷权限在启动前由 setup 脚本通过 root 容器自动修正。

---
<p align="center">
  <a href="../README.md">← 返回主 README</a>
</p>
