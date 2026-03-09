# OpenClaw 开发工具箱套件 (OpenClaw DevKit)

[![OpenClaw](https://img.shields.io/badge/Powered%20By-OpenClaw-blue)](https://github.com/openclaw/openclaw)
[![Docker](https://img.shields.io/badge/Environment-Docker-blue?logo=docker)](https://www.docker.com/)
[![Claude Code](https://img.shields.io/badge/Built%20With-Claude%20Code-purple)](https://claude.ai/code)

**OpenClaw 开发工具箱套件** 为 [OpenClaw](https://github.com/openclaw/openclaw) 多通道 AI 生产力工具提供完整的容器化开发、调试与运行环境。

集成了一套开箱即用的工具链，旨在帮助开发者快速构建基于 OpenClaw 的 AI 工作流，支持自动源码更新、一键环境搭建及内置的地理位置代理优化。

---

## ✨ 核心特性

- 🚀 **一键式环境搭建**：基于 Docker Compose，秒级启动完整的开发运行环境。
- 🛠️ **全能开发栈**：镜像内置 Go 1.25, Node.js 22, pnpm, Bun, Python 以及 `gh` CLI。
- 🤖 **Claude Code 集成**：原生支持 Claude Code CLI，提供极致的 AI 辅助编程体验。
- 🌐 **网络优化**：内置针对 Google API 和 Claude API 的代理转发逻辑，解决国内访问难题。
- 🎥 **自动化能力**：预装 Playwright 及所有浏览器依赖，支持复杂的网页自动化任务。
- 📝 **文档处理**：集成 Pandoc 和 LaTeX，支持高质量的文档格式转换与生成。
- 💾 **数据持久化**：精心设计的 Named Volumes，确保 node_modules、Go 缓存及会话数据在容器重启后依然存在。

---

## 🏗️ 项目架构

![项目架构图](docs/assets/architecture.svg)

<details>
<summary>查看 Mermaid 源码</summary>

```mermaid
graph TD
    User([开发者]) -->|make| Makefile[Makefile 运维入口]
    Makefile -->|control| DockerCompose[Docker Compose]
    
    subgraph "Docker Container (openclaw-gateway)"
        Gateway[OpenClaw Gateway]
        CLI[OpenClaw CLI]
        Claude[Claude Code CLI]
        Playwright[Playwright Browsers]
    end

    subgraph "Host System"
        Src[.openclaw_src - 源码]
        Config[~/.openclaw - 配置文件]
        Env[.env - 环境变量]
    end

    DockerCompose -->|Mount| Src
    DockerCompose -->|Mount| Config
    DockerCompose -->|Load| Env
    
    Gateway -->|Proxy| ProxyServer((外部代理服务:7897/15721))
```
</details>

---

## 📂 目录结构

| 路径                     | 说明                                         |
| :----------------------- | :------------------------------------------- |
| `Makefile`               | 项目运维命令的统一入口                       |
| `docker-compose.dev.yml` | Docker Compose 服务定义                      |
| `Dockerfile.dev`         | 开发镜像构建脚本                             |
| `.openclaw_src/`         | OpenClaw 核心源码 (git submodule 或本地目录) |
| `docker-dev-setup.sh`    | 首次运行的初始化脚本                         |
| `update-source.sh`       | 从 GitHub 发布页面自动更新源码的工具         |
| `.env`                   | 环境变量配置 (git-ignored)                   |

---

## 🚥 快速开始

### 1. 准备工作
确保宿主机已安装：
- Docker & Docker Compose (V2)
- 可选：已配置好的本地 HTTP 代理 (默认 7897 端口)

### 2. 初始化安装
```bash
# 自动执行权限设置、网络检查及镜像构建
make install
```

### 3. 启动服务
```bash
make up
```

### 4. 访问界面
- **Web 控制台**: [http://127.0.0.1:18789](http://127.0.0.1:18789)
- **API 接口**: `ws://127.0.0.1:18789`

---

## ⚙️ 配置说明

编辑项目根目录下的 `.env` 文件进行个性化配置：

| 变量名                  | 说明                            | 示例值                             |
| :---------------------- | :------------------------------ | :--------------------------------- |
| `OPENCLAW_CONFIG_DIR`   | 宿主机配置存储路径              | `~/.openclaw`                      |
| `OPENCLAW_GATEWAY_PORT` | Gateway 访问端口                | `18789`                            |
| `HTTP_PROXY`            | 容器访问外网用的代理            | `http://host.docker.internal:7897` |
| `GITHUB_TOKEN`          | 用于 `make update` 自动拉取源码 | `your_github_token`                |

---

## 🛠️ 运维命令手册

| 命令分类     | 命令                  | 说明                                              |
| :----------- | :-------------------- | :------------------------------------------------ |
| **生命周期** | `make up / down`      | 启动 / 停止服务                                   |
|              | `make restart`        | 重启所有服务                                      |
|              | `make status`         | 查看容器状态及访问地址                            |
| **构建更新** | `make build`          | 重新构建开发镜像                                  |
|              | `make rebuild`        | 重建镜像并重启服务 (更新代码后必用)               |
|              | `make update`         | 从 GitHub Release 获取最新 OpenClaw 源码          |
| **调试诊断** | `make logs`           | 追踪 Gateway 主服务日志                           |
|              | `make shell`          | 进入容器内部交互环境 (bash)                       |
|              | `make test-proxy`     | **一键测试** Google/Claude API 连通性             |
|              | `make gateway-health` | 检查网关响应状态                                  |
| **备份恢复** | `make backup-config`  | 备份所有 Agent 及全局配置到 `~/.openclaw-backups` |
|              | `make restore-config` | 交互式恢复指定的配置文件                          |
| **清理**     | `make clean`          | 清理孤儿容器与悬空镜像                            |
|              | `make clean-volumes`  | **危险**：清空所有缓存与持久化数据卷              |

---

## 🔄 开发流程

1. **修改代码**：直接编辑 `.openclaw_src/` 目录下的代码。
2. **应用更改**：运行 `make rebuild`。由于使用了 Named Volumes 存储 `node_modules`，构建速度非常快。
3. **查看效果**：访问 Web UI 或查看 `make logs`。
4. **运行测试**：`make exec CMD="pnpm test"`。

---

## ❓ 常见问题 (FAQ)

**Q: 容器内无法访问外网或 Claude API？**
A: 请确保宿主机上的代理服务 (如 Clash/V2Ray) 已开启「允许局域网连接」，且端口与 `.env` 中一致。使用 `make test-proxy` 可快速定位问题。

**Q: 如何更新到 OpenClaw 的最新正式版？**
A: 运行 `make update` 即可，脚本会自动处理解压与目录替换。

**Q: 更改了镜像配置但没生效？**
A: 使用 `make build` 而不是 `make up`，或者直接 `make rebuild`。

---

## 📄 许可证

基于 [OpenClaw](https://github.com/openclaw/openclaw) 的原始许可协议。建议详细阅读核心源码中的 LICENSE 文件。
