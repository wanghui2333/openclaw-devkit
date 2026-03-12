# OpenClaw DevKit 用户安装手册

本文档描述了新用户从克隆仓库到完成配置的完整执行流程。

## 1. 极速起步 (Fast Path - 仅需 2 步)

对于大多数用户，直接使用预构建镜像即可：

```bash
# 第 1 步：安装并启动 (自动下载镜像、初始化配置)
make install [flavor]      # flavor 可选: go, java, office. 默认为标准版

# 第 2 步：交互式配置 (设置 LLM、Slack、飞书等)
make onboard
```

## 2. 安装执行全流程 (Detailed Execution Flow)

```text
       开始 (Start)
          │
          ▼
    ┌───────────────────────────┐
    │  1. 下载源码 (Clone)      │
    │  $ git clone ...          │
    └──────────┬────────────────┘
               │
               ▼
    ┌───────────────────────────┐
    │  2. 初始化并拉起 (Install) │
    │  $ make install [flavor]  │
    └──────────┬────────────────┘
               │
               ├─> 🔍 检查 Docker/Compose
               ├─> 📄 生成 .env (基于模板)
               ├─> 🔑 生成安全 Token
               ├─> 📁 初始化存储目录
               ├─> 🏗️  拉取对应镜像 (Pull/Build)
               └─> 🚀 启动 Gateway 服务
               │
               ▼
    ┌───────────────────────────┐
    │  3. 交互引导 (Onboard)    │
    │  $ make onboard           │
    └──────────┬────────────────┘
               │
               └─> 🎙️  设置 LLM、配对频道
               │
               ▼
             完成 (Done)

## 3. 老手进阶操作 (Advanced Operations)

对于熟悉 Docker 的“老手”，可以通过以下方式快速维护和切换环境：

### A. 切换镜像变体 (Switching Flavors)
如果您已经安装了标准版，现在想切换到 Go 开发版：
```bash
# 方式 1：一键切换（推荐，会自动同步 .env 并重启）
make install go

# 方式 2：强制拉取/重建并重启
make rebuild go
```

### B. 系统升级 (Updating)
当仓库有新的逻辑更新时：
```bash
make update    # 同步最新代码并重启容器
```

### C. 手动调优 (Manual Tuning)
您可以直接编辑 `.env` 文件进行深度定制：
- 修改 `OPENCLAW_IMAGE` 为特定版本。
- 调整代理 `HTTP_PROXY` 或端口。
- 增加 `OPENCLAW_EXTRA_MOUNTS` 挂载本地代码。
- **改动后执行**: `make restart` 即可生效。

### D. 状态诊断
```bash
make status    # 实时查看分层架构状态、镜像大小和运行情况
make verify    # 检查容器内工具链版本是否合规
```

## 4. 常见疑问 (FAQ)

### Q: 已经安装过了，每天启动还需要执行 `make install` 吗？
**A: 不需要。**
- **日常启动**: 仅需 `make up`。
- **切换版本**: 建议使用 `make install <flavor>`，因为它会同步修改 `.env` 并确保新版本的镜像已就绪。
- **强制更新**: 如果你想拉取最新的远程镜像并覆盖本地，使用 `make rebuild`。

### Q: `make install` 会删掉我的数据吗？
**A: 不会。**
配置目录 (`~/.openclaw`) 和工作区 (`workspace`) 是持久化的。`make install` 仅负责环境适配（如更新 `.env`、检查 Docker 权限等），是幂等（Idempotent）的，可以放心多次执行。
```

## 2. 关键阶段说明

### A. 初始化 (`make install`)
该步骤由 `docker-setup.sh` 驱动，会自动识别您的宿主机环境，并确保所有挂载点（Volumes）和权限（Permissions）已就绪。

### B. 按需构建 (`make build`)
由于采用了分层架构，如果您选择了 `go` 版本，系统会智能拉取/构建 Go 运行时，然后仅在顶层安装 OpenClaw。

### C. 启动与引导 (`make up` & `make onboard`)
启动容器后，`make onboard` 会进入容器内部，通过交互式命令行引导您完成 API Key 的填报和通讯渠道的配对。

## 5. 容器运行时架构 (Runtime Architecture)

### 启动流程图

```text
make install / make up
       │
       ▼
┌──────────────────────────────┐
│  1. openclaw-init           │ ◄── 一次性容器
│  • 执行 openclaw doctor     │     仅在首次或配置
│    --fix 修复过时配置        │     问题时运行
│  • 清理 contextPruning 等   │
│    废弃字段                 │
└──────────┬───────────────────┘
           │ 完成后自动删除
           ▼
┌──────────────────────────────┐
│  2. openclaw-gateway        │ ◄── 主服务 (常驻)
│  • 健康检查通过后运行        │
│  • 提供 Web UI (18789)      │
│  • 提供 Bridge (18790)      │
└──────────┬───────────────────┘
           │ depends_on:
           │   condition: service_healthy
           ▼
┌──────────────────────────────┐
│  3. openclaw-cli           │ ◄── 按需启动
│  • 仅在 make onboard /     │
│    make cli 时运行          │
│  • 交互式命令行界面         │
└──────────────────────────────┘
```

### 配置迁移机制 (Config Migration)

当用户从旧版本 OpenClaw 升级时，可能遇到配置文件 schema 不兼容问题（如 `contextPruning.windowSize` 字段已废弃）。

**自动修复流程：**

1. **Pre-start Init**: `openclaw-init` 容器在 gateway 启动前运行
2. **Doctor Fix**: 执行 `openclaw doctor --fix` 自动修复已知问题
3. **深度净化**: Node.js 脚本清理废弃字段（`contextPruning`, `compaction` 等）
4. **Gateway 启动**: 配置修复完成后，gateway 才开始健康检查

**故障排查：**
- 如果启动失败，查看 init 日志：`docker logs openclaw-init`
- 手动修复配置：`docker exec openclaw-gateway openclaw doctor --fix`

### 数据持久化

| 数据类型 | 存储位置 | 说明 |
| :--- | :--- | :--- |
| **配置文件** | `~/.openclaw/openclaw.json` | 宿主机目录，容器只读挂载 |
| **会话数据** | Docker 卷 `openclaw-state` | 容器内持久化 |
| **工作区** | `~/.openclaw/workspace` | 宿主机目录，双向同步 |
