# OpenClaw × NotebookLM 技能集成指南

本指南介绍如何在 OpenClaw DevKit 中集成和使用 Google NotebookLM CLI 技能，实现通过自然语言操控 NotebookLM 的全部功能。

## 目录

- [概述](#概述)
- [宿主机安装与认证](#宿主机安装与认证)
- [容器环境自动配置](#容器环境自动配置)
- [通过自然语言安装技能](#通过自然语言安装技能)
- [实战案例](#实战案例)
- [故障排除](#故障排除)

---

## 概述

**notebooklm-py** 是 Google NotebookLM 的非官方 Python SDK 和 CLI 工具，提供：

| 功能 | 说明 |
|:-----|:-----|
| 📓 Notebook 管理 | 创建、列表、重命名、删除 |
| 📄 多格式来源 | URLs、YouTube、PDF、Word、音视频、Google Drive |
| 💬 智能对话 | 基于来源的问答、自定义人设 |
| 🔍 研究代理 | 网页/Drive 深度研究，自动导入 |
| 🎙️ 内容生成 | 播客、视频、幻灯片、测验、思维导图等 |
| 📥 批量导出 | MP3、MP4、PDF、PNG、CSV、JSON、Markdown |

> ⚠️ **注意**: 此工具使用未公开的 Google API，可能随时变化。适合原型开发、研究和个人项目。

---

## 宿主机安装与认证

### 1. 安装 CLI 工具

```bash
# 基础安装
pip install notebooklm-py

# 安装浏览器登录支持（首次配置必需）
pip install "notebooklm-py[browser]"
playwright install chromium
```

### 2. Google 账号认证

```bash
# 启动浏览器登录流程
notebooklm login
```

执行后会自动打开浏览器窗口：

1. 登录你的 Google 账号
2. 完成身份验证
3. 认证信息会自动保存到 `~/.notebooklm/storage_state.json`

**企业用户**（需要 Edge SSO）：

```bash
notebooklm login --browser msedge
```

### 3. 验证认证状态

```bash
# 检查认证状态
notebooklm auth check --test
```

输出示例：

```
✓ Storage file exists: /Users/you/.notebooklm/storage_state.json
✓ Authentication valid
✓ API access confirmed
```

---

## 容器环境自动配置

OpenClaw DevKit 已预配置 NotebookLM 支持，实现宿主机认证与容器共享。

### 配置原理

```
┌─────────────────────────────────────────────────────────────┐
│                      宿主机 (Host)                           │
│                                                             │
│  ~/.notebooklm/                                             │
│  └── storage_state.json  ←── Google 认证凭证                 │
│                                                             │
└────────────────────┬────────────────────────────────────────┘
                     │ bind mount (rw)
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   容器 (Container)                           │
│                                                             │
│  /home/node/.notebooklm/                                    │
│  └── storage_state.json  ←── 与宿主机共享                    │
│                                                             │
│  /root/.notebooklm → /home/node/.notebooklm  (符号链接)      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 环境变量配置

在 `.env` 文件中配置自动安装：

```bash
# Python 工具自动安装
# 格式: "包名[:命令名]"（空格分隔多个）
PIP_TOOLS=notebooklm-py:notebooklm
```

**说明**:
- `notebooklm-py` 是 PyPI 包名
- `notebooklm` 是安装后的 CLI 命令
- 容器启动时通过 `uv pip install --system` 自动安装

### 验证容器配置

```bash
# 进入容器
make shell

# 检查 CLI 是否可用
notebooklm auth check

# 列出 notebooks
notebooklm list
```

---

## 通过自然语言安装技能

### 方式一：CLI 安装

在容器内执行：

```bash
notebooklm skill install
```

### 方式二：自然语言安装

直接对 OpenClaw 说：

> "帮我安装 tiangong-notebooklm 技能，这样我就可以通过自然语言操控 NotebookLM 了"

或者：

> "Install the notebooklm skill so I can manage my Google NotebookLM notebooks"

OpenClaw 会自动执行安装流程。

### 验证技能安装

```bash
# 检查技能状态
notebooklm skill status
```

---

## 实战案例

### 案例 1：OpenClaw Skill 编写最佳实践

**场景**: 学习如何为 OpenClaw 编写高质量的 NotebookLM Skill，遵循官方 Agent Skills 最佳实践。

本案例基于 [Claude Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) 官方指南。

#### 核心原则速查

| 原则 | 说明 |
|:-----|:-----|
| **简洁优先** | 假设 OpenClaw 已具备基础能力，只补充必要上下文 |
| **自由度匹配** | 脆弱操作 = 低自由度（精确脚本）；灵活任务 = 高自由度（文本指令） |
| **渐进披露** | SKILL.md < 500 行，细节放入引用文件 |
| **工作流清单** | 复杂任务使用可复制的检查清单 |
| **有效描述** | 第三人称，包含触发关键词 |

#### 推荐的 Skill 目录结构

```
~/.claude/commands/notebooklm/
├── SKILL.md              # 主文件（概览 + 快速开始）
├── REFERENCE.md          # CLI 完整命令参考
└── EXAMPLES.md           # 输入/输出示例
```

#### SKILL.md 模板（遵循最佳实践）

```markdown
---
name: notebooklm
description: Manage Google NotebookLM notebooks, sources, and artifacts. Use when the user mentions NotebookLM, podcasts, generating audio/video from documents, or asks to create learning materials from sources.
---

# NotebookLM Skill

Complete API for Google NotebookLM - create notebooks, add sources, generate podcasts/videos/quizzes, and download in multiple formats.

## Quick start

Create a podcast from web sources:

```bash
notebooklm create "Research Topic"
notebooklm use <id>
notebooklm source add "https://example.com/article" --wait
notebooklm generate audio "engaging discussion style" --wait
notebooklm download audio ./podcast.mp3
```

## Common operations

**Add sources**: See [REFERENCE.md](REFERENCE.md#sources)
**Generate content**: See [REFERENCE.md](REFERENCE.md#generate)
**Examples**: See [EXAMPLES.md](EXAMPLES.md)

## Content types

| Type | CLI command | Output formats |
|:-----|:------------|:---------------|
| Audio (podcast) | `generate audio` | MP3, MP4 |
| Video | `generate video` / `cinematic-video` | MP4 |
| Quiz | `generate quiz` | JSON, Markdown, HTML |
| Flashcards | `generate flashcards` | JSON, Markdown, HTML |

## Auth check

```bash
notebooklm auth check --test
```
```

#### 关键要点说明

**1. 描述字段（description）**

```yaml
# ✅ 好的写法（第三人称 + 触发关键词）
description: Manage Google NotebookLM notebooks, sources, and artifacts. Use when the user mentions NotebookLM, podcasts, generating audio/video from documents...

# ❌ 避免的写法
description: I can help you with NotebookLM...  # 第一人称
description: NotebookLM tool  # 太模糊，缺少触发词
```

**2. 自由度匹配**

```markdown
# 高自由度（灵活任务）
## Code review process
1. Analyze the notebook sources
2. Identify key themes
3. Suggest content improvements

# 低自由度（脆弱操作）
## Database migration
Run exactly: `notebooklm source add-research "topic" --wait`
Do not modify this command.
```

**3. 渐进式披露**

```markdown
# SKILL.md 保持精简，细节放入引用文件
**Advanced features**: See [REFERENCE.md](REFERENCE.md)
**Examples**: See [EXAMPLES.md](EXAMPLES.md)
```

#### 工作流清单模板

对于复杂任务，提供可复制的检查清单：

```markdown
## Podcast generation workflow

Copy this checklist to track progress:

```
Progress:
- [ ] Step 1: Create notebook
- [ ] Step 2: Add sources (wait for processing)
- [ ] Step 3: Verify sources indexed
- [ ] Step 4: Generate audio
- [ ] Step 5: Download and verify
```
```

#### 避免的反模式

| 反模式 | 正确做法 |
|:-------|:---------|
| Windows 路径 `scripts\helper.py` | Unix 路径 `scripts/helper.py` |
| 提供太多选项 "可以用 A、B、C..." | 提供默认方案 + 逃生舱 |
| 深层嵌套引用 A→B→C | 引用保持一层深度 |
| 时间敏感信息 "2025年8月前..." | 使用 "old patterns" 折叠区 |

### 案例 2：批量生成学习材料

**场景**: 为一门课程生成测验题和闪卡。

**自然语言**:

> "使用我当前的 notebook 生成一套难度较高的测验题（20道），再生成一套闪卡，都导出为 Markdown 格式"

**等效 CLI**:

```bash
# 生成测验题
notebooklm generate quiz --difficulty hard --quantity more

# 生成闪卡
notebooklm generate flashcards --quantity more

# 导出
notebooklm download quiz --format markdown ./quiz.md
notebooklm download flashcards --format markdown ./flashcards.md
```

### 案例 3：创建演示视频

**场景**: 为项目文档生成白板风格的讲解视频或纪录片风格视频。

**自然语言**:

> "生成一个白板风格的讲解视频，时长 5 分钟，主题是项目架构概览"

或

> "生成一个纪录片风格的视频概述"

**等效 CLI**:

```bash
# 白板风格视频
notebooklm generate video --style whiteboard --wait
notebooklm download video ./overview.mp4

# 纪录片风格视频（独立命令）
notebooklm generate cinematic-video "documentary-style summary" --wait
notebooklm download cinematic-video ./documentary.mp4
```

### 案例 4：研究与自动导入

**场景**: 自动搜索并导入相关资料。

**自然语言**:

> "帮我研究一下 'LLM Function Calling' 这个主题，从网页搜索相关资料并自动导入到当前 notebook"

**等效 CLI**:

```bash
notebooklm source add-research "LLM Function Calling"
```

### 案例 5：生成思维导图

**场景**: 可视化 notebook 中的知识结构。

**自然语言**:

> "生成当前 notebook 的思维导图，导出为 JSON 格式，我需要在其他工具中可视化"

**等效 CLI**:

```bash
notebooklm generate mind-map
notebooklm download mind-map ./mindmap.json
```

---

## 支持的内容类型

| 类型 | 选项 | 导出格式 |
|:-----|:-----|:---------|
| **Audio Overview** | 4 种风格 (deep-dive/brief/critique/debate)、3 种时长、50+ 语言 | MP3/MP4 |
| **Video Overview** | 3 种风格 (explainer/brief/cinematic)、9 种视觉风格、独立 `cinematic-video` 别名 | MP4 |
| **Slide Deck** | 详细版/演讲版、可调长度 | PDF, PPTX |
| **Infographic** | 3 种方向、3 种细节级别 | PNG |
| **Quiz** | 可配置数量和难度 | JSON, Markdown, HTML |
| **Flashcards** | 可配置数量和难度 | JSON, Markdown, HTML |
| **Report** | 简报/学习指南/博客文章/自定义提示词 | Markdown |
| **Data Table** | 自然语言定义结构 | CSV |
| **Mind Map** | 交互式层级可视化 | JSON |

---

## 故障排除

### 认证失败

```bash
# 检查认证状态
notebooklm auth check --test

# 重新登录
notebooklm login
```

### 容器内找不到命令

```bash
# 检查是否安装
which notebooklm

# 手动安装
uv pip install --system notebooklm-py
```

### 权限问题

如果遇到 `EACCES` 错误：

```bash
# 检查目录权限
ls -la ~/.notebooklm/

# 修复权限
chmod -R 755 ~/.notebooklm/
```

### API 限流

NotebookLM 有请求频率限制。如遇到限流：

1. 减少并发请求
2. 增加请求间隔
3. 等待一段时间后重试

---

## 参考资料

- [notebooklm-py GitHub](https://github.com/teng-lin/notebooklm-py)
- [notebooklm-py PyPI](https://pypi.org/project/notebooklm-py/)
- [Google NotebookLM 官方网站](https://notebooklm.google.com/)

---

## 附录：常用 CLI 命令速查

```bash
# 认证
notebooklm login                    # 浏览器登录
notebooklm auth check --test        # 检查认证

# Notebook 管理
notebooklm list                     # 列出所有 notebooks
notebooklm create "名称"            # 创建新 notebook
notebooklm use <id>                 # 切换当前 notebook
notebooklm metadata --json          # 导出元数据

# 来源管理
notebooklm source add <url|文件>    # 添加来源
notebooklm source list              # 列出来源
notebooklm source add-research "主题" # 研究并导入

# 问答
notebooklm ask "问题"               # 提问

# 内容生成
notebooklm generate audio           # 生成播客
notebooklm generate video           # 生成视频
notebooklm generate cinematic-video # 生成纪录片风格视频
notebooklm generate quiz            # 生成测验
notebooklm generate flashcards      # 生成闪卡
notebooklm generate slide-deck      # 生成幻灯片
notebooklm generate infographic     # 生成信息图
notebooklm generate mind-map        # 生成思维导图

# 下载
notebooklm download audio ./x.mp3   # 下载音频
notebooklm download video ./x.mp4   # 下载视频
notebooklm download cinematic-video ./x.mp4  # 下载纪录片视频
notebooklm download quiz --format markdown ./x.md  # 下载测验

# 技能
notebooklm skill install            # 安装 Claude Code 技能
notebooklm skill status             # 检查技能状态
```
