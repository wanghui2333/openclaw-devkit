# Makefile UI 现代化改进计划

> **For Claude:** 使用 superpowers:executing-plans 技能逐任务执行此计划。

**目标:** 将 Makefile 帮助界面改为现代分组列表形式，命令+简述展示，底部快捷操作提示

**方案:** 重新设计 help 和 help-full 目标，使用 emoji 分组、固定宽度对齐、底部提示栏

**技术:** GNU Makefile 原生语法 (无外部依赖)

---

## 任务总览

| 任务 | 内容 | 文件 |
|:---:|:---|:---|
| 1 | 重写 help 目标 (主帮助) | Makefile:56-84 |
| 2 | 重写 help-full 目标 (完整帮助) | Makefile:85-135 |
| 3 | 测试输出效果 | 终端执行 `make help` |
| 4 | 提交变更 | git commit |

---

### Task 1: 重写 help 目标 (主帮助)

**文件:**
- 修改: `Makefile:56-84`

**步骤 1: 替换 help 目标代码**

将现有的 help 目标替换为新的分组式输出:

```makefile
help: ## 显示帮助 (现代分组版)
	@echo ""
	@echo "  🦷  OpenClaw DevKit  v2.0"
	@echo "  ─────────────────────────────────────────────"
	@echo ""
	@echo "  ⚡  快速开始"
	@printf "    %-20s %s\n" "install" "安装环境并启动"
	@printf "    %-20s %s\n" "up" "启动服务"
	@printf "    %-20s %s\n" "logs" "查看日志"
	@echo ""
	@echo "  🔄  生命周期"
	@printf "    %-20s %s\n" "down" "停止服务"
	@printf "    %-20s %s\n" "restart" "重启服务"
	@printf "    %-20s %s\n" "status" "查看状态"
	@echo ""
	@echo "  🔧  构建"
	@printf "    %-20s %s\n" "build [version]" "构建镜像"
	@printf "    %-20s %s\n" "rebuild [version]" "重建并重启"
	@echo ""
	@echo "  🐛  调试"
	@printf "    %-20s %s\n" "shell" "进入容器"
	@printf "    %-20s %s\n" "logs-all" "查看所有日志"
	@printf "    %-20s %s\n" "exec CMD='...'" "执行命令"
	@printf "    %-20s %s\n" "test-proxy" "测试代理"
	@echo ""
	@echo "  💾  备份"
	@printf "    %-20s %s\n" "backup-config" "备份配置"
	@printf "    %-20s %s\n" "restore-config FILE=..." "恢复配置"
	@echo ""
	@echo "  🧹  维护"
	@printf "    %-20s %s\n" "update" "更新源码"
	@printf "    %-20s %s\n" "check-deps" "检查依赖"
	@printf "    %-20s %s\n" "clean" "清理镜像"
	@printf "    %-20s %s\n" "clean-volumes" "清理数据卷"
	@echo ""
	@echo "───────────────────────────────────────────────"
	@printf "  版本: %s\n" "make <cmd> java|office|dev"
	@printf "  完整: %s\n" "make help-full"
	@echo "───────────────────────────────────────────────"
	@echo ""
```

---

### Task 2: 重写 help-full 目标 (完整帮助)

**文件:**
- 修改: `Makefile:85-135`

**步骤 1: 替换 help-full 目标代码**

将现有的 help-full 目标替换为更详细的分组式输出:

```makefile
help-full: ## 显示完整帮助
	@echo ""
	@echo "  🦷  OpenClaw DevKit  v2.0  |  完整命令列表"
	@echo "  ═════════════════════════════════════════════"
	@echo ""
	@echo "  ⚡  快速开始 (首次使用)"
	@printf "    %-22s %s\n" "make install" "安装并初始化环境"
	@printf "    %-22s %s\n" "make up" "启动服务"
	@printf "    %-22s %s\n" "make down" "停止服务"
	@echo ""
	@echo "  🔄  生命周期管理"
	@printf "    %-22s %s\n" "make install" "首次安装/初始化"
	@printf "    %-22s %s\n" "make up" "启动服务"
	@printf "    %-22s %s\n" "make down" "停止服务"
	@printf "    %-22s %s\n" "make restart" "重启服务"
	@printf "    %-22s %s\n" "make status" "查看服务状态"
	@echo ""
	@echo "  🔧  构建 (支持版本: dev|java|office)"
	@printf "    %-22s %s\n" "make build" "构建镜像 (默认标准版)"
	@printf "    %-22s %s\n" "make build java" "构建 Java 增强版"
	@printf "    %-22s %s\n" "make build office" "构建 Office 办公版"
	@printf "    %-22s %s\n" "make rebuild" "重建并重启"
	@printf "    %-22s %s\n" "make rebuild java" "重建 Java 版并重启"
	@printf "    %-22s %s\n" "make rebuild office" "重建 Office 版并重启"
	@echo ""
	@echo "  🐛  调试与诊断"
	@printf "    %-22s %s\n" "make logs" "查看 Gateway 日志"
	@printf "    %-22s %s\n" "make logs-all" "查看所有容器日志"
	@printf "    %-22s %s\n" "make shell" "进入 Gateway 容器"
	@printf "    %-22s %s\n" "make exec CMD='...'" "在容器中执行命令"
	@printf "    %-22s %s\n" "make cli CMD='...'" "执行 OpenClaw CLI"
	@printf "    %-22s %s\n" "make pairing" "频道配对"
	@printf "    %-22s %s\n" "make gateway-health" "检查 Web UI 健康"
	@printf "    %-22s %s\n" "make test-proxy" "测试代理连接"
	@printf "    %-22s %s\n" "make verify" "验证工具版本"
	@echo ""
	@echo "  💾  备份与恢复"
	@printf "    %-22s %s\n" "make backup-config" "备份配置到 ~/.openclaw-backups/"
	@printf "    %-22s %s\n" "make restore-config FILE=xxx" "恢复配置"
	@echo ""
	@echo "  🧹  维护与清理"
	@printf "    %-22s %s\n" "make update" "从 GitHub 更新源码"
	@printf "    %-22s %s\n" "make check-deps" "检查 Docker 依赖"
	@printf "    %-22s %s\n" "make clean" "清理容器和悬空镜像"
	@printf "    %-22s %s\n" "make clean-volumes" "清理所有数据卷 (危险!)"
	@echo ""
	@echo "════════════════════════════════════════════════════"
	@echo "  版本选择: make <cmd> <version>"
	@echo "    - dev    : 标准版 (默认)"
	@echo "    - java   : Java 增强版"
	@echo "    - office : Office 办公版"
	@echo ""
	@echo "  示例:"
	@printf "    %s\n" "make install java    # 安装 Java 版"
	@printf "    %s\n" "make build office    # 构建 Office 版"
	@printf "    %s\n" "make rebuild office  # 重建 Office 版"
	@echo "════════════════════════════════════════════════════"
	@echo ""
```

---

### Task 3: 测试输出效果

**步骤 1: 执行 make help 查看输出**

```bash
cd /Users/huangzhonghui/openclaw && make help
```

预期输出应包含:
- 分组标题带 emoji (⚡🔄🔧🐛💾🧹)
- 命令名称 + 描述固定宽度对齐
- 底部快捷操作提示

**步骤 2: 执行 make help-full 查看完整输出**

```bash
make help-full
```

预期输出应包含:
- 所有命令的完整列表
- 版本选择的详细说明
- 使用示例

---

### Task 4: 提交变更

**步骤 1: 检查 git 状态**

```bash
git status
git diff Makefile
```

**步骤 2: 提交**

```bash
git add Makefile
git commit -m "refactor: modernize Makefile help UI with grouped commands

- Add emoji分组标题 (⚡🔄🔧🐛💾🧹)
- Use printf for fixed-width command alignment
- Add 底部快捷操作提示
- Support parameterized version selection (java|office|dev)

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

---

## 实现顺序

1. Task 1: 重写 help 目标
2. Task 2: 重写 help-full 目标
3. Task 3: 测试输出效果
4. Task 4: 提交变更

**选择执行方式:**
1. **Subagent-Driven (当前会话)** - 逐任务调度子代理，快速迭代
2. **Parallel Session (新会话)** - 新开会话批量执行

请选择你偏好的执行方式。
