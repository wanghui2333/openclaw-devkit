# OpenClaw 多 Agent 系统优化分析报告

## 一、ultrawork-mode 源流分析

**重要发现**: ultrawork-mode 来自 **oh-my-opencode** (现 oh-my-openagent)，一个有 38k+ stars 的开源 Agent 框架。

---

## 二、oh-my-opencode 核心架构（关键参考）

### 2.1 Discipline Agents（四神）

| Agent | 定位 | 模型 | 特点 |
|-------|------|------|------|
| **Sisyphus** | 主协调者 | claude-opus-4-6 / kimi-k2.5 / glm-5 | 任务规划、委派、强制执行 |
| **Hephaestus** | 深度工作者 | gpt-5.3-codex | 自主研究+执行，端到端完成 |
| **Prometheus** | 战略规划者 | 同 Sisyphus | 面试式问题分析，生成详细计划 |
| **Oracle** | 架构/调试 | (待配置) | 传统问题处理 |

### 2.2 Agent Category 分类

Agent 不指定具体模型，而是指定**类别**：

| Category | 用途 |
|----------|------|
| `visual-engineering` | 前端、UI/UX、设计 |
| `deep` | 自主研究+执行 |
| `quick` | 单文件修改、typo |
| `ultrabrain` | 硬逻辑、架构决策 |

**关键**: Harness（框架）根据 Category 自动选择最佳模型。

### 2.3 关键创新特性

1. **Hash-Anchored Edit Tool (Hashline)**
   - 每行代码带内容哈希标识 (`11#VK|`)
   - 编辑时验证哈希，拒绝内容变化的编辑
   - 消除 stale-line 错误

2. **IntentGate**
   - 分析用户真实意图
   - 在分类或行动前进行意图识别

3. **Skill-Embedded MCPs**
   - Skill 自带 MCP 服务器
   - 按需启动，任务结束即销毁
   - 保持上下文干净

4. **/init-deep**
   - 自动生成层次化 AGENTS.md
   - 项目级 / 目录级 / 组件级

### 2.4 Ralph Loop
- 自我引用循环
- 任务 100% 完成前不停止
- "不达目的不罢休"的执行模式

### 1.1 核心分类

| Agent 类型 | 处理问题 | 特点 |
|-----------|---------|------|
| **Oracle** | 传统问题 | 架构、调试、复杂逻辑 |
| **Artistry** | 非传统问题 | 需要创新方法、不同路径 |
| **Explore** | 代码探索 | 快速定位、模式识别 |
| **Librarian** | 文档查找 | 权威文档、示例检索 |
| **Plan** | 任务规划 | 并行任务图、依赖分析 |

### 1.2 关键机制

- **任务图生成**: Plan Agent 分析任务，创建并行执行图
- **专业化分工**: 根据问题类型匹配最合适的 Agent
- **验证协议**: 预定义成功标准、零容忍失败

---

## 三、OpenClaw 现有 Agent 架构

### 3.1 当前 Agent 列表

| Agent ID | 角色 | 功能 |
|----------|------|------|
| main | Commander | 协调者，总指挥 |
| scout | Scout | 上下文分析、文件识别 |
| developer | Developer | 代码实现 |
| reviewer | Reviewer | 质量审查 |
| tester | Tester | 测试验证 |
| github-ops | GitHub Expert | GitHub 操作 |

### 3.2 当前架构特点

- **工作流角色分配**: Developer → Reviewer → Tester
- **静态委派**: Main Agent 决定委派给哪个子 Agent
- **功能导向**: 按"做什么"（开发、审查、测试）划分

---

## 四、oh-my-opencode vs OpenClaw 详细对比

| 维度 | oh-my-opencode | OpenClaw (当前) | 借鉴可行性 |
|------|----------------|-----------------|------------|
| **Agent 架构** | 四神 (Sisyphus/Hephaestus/Prometheus/Oracle) | Commander + 5 workers | ⭐⭐⭐⭐⭐ |
| **Category 分类** | visual/deep/quick/ultrabrain | 无 | ⭐⭐⭐⭐ |
| **任务规划** | Prometheus 面试式分析 | 手工 | ⭐⭐⭐⭐⭐ |
| **模型选择** | Category → 自动映射 | 静态配置 | ⭐⭐⭐⭐ |
| **编辑工具** | Hashline 哈希验证 | 普通 Edit | ⭐⭐⭐ |
| **意图识别** | IntentGate | 无 | ⭐⭐⭐⭐ |
| **Skill + MCP** | Skill 自带 MCP | 分离 | ⭐⭐⭐⭐⭐ |
| **执行保证** | Ralph Loop 100% 完成 | 基础 | ⭐⭐⭐⭐⭐ |
| **上下文管理** | /init-deep 层次化 | 手工 | ⭐⭐⭐⭐ |

---

## 五、可借鉴的关键特性（按优先级）

### 5.1 高优先级（可直接借鉴）

| 特性 | 说明 | 实现难度 |
|------|------|----------|
| **Agent Category 映射** | 问题类型 → Agent 映射，自动模型选择 | 中 |
| **Prometheus 规划** | 面试式问题分析 + 详细计划生成 | 高 |
| **Ralph Loop** | 不停止执行直到 100% 完成 | 中 |

### 5.2 中优先级（需适配）

| 特性 | 说明 | 实现难度 |
|------|------|----------|
| **Skill-Embedded MCPs** | Skill 携带自己的 MCP 配置 | 高 |
| **IntentGate** | 意图分析 + 误解预防 | 高 |
| **Hashline 编辑** | 哈希验证的编辑工具 | 中 |

### 5.3 低优先级（可选）

| 特性 | 说明 | 实现难度 |
|------|------|----------|
| **/init-deep** | 自动生成 AGENTS.md | 低 |

---

## 六、优化方案（基于 oh-my-opencode 调研）

### 6.1 推荐方案：渐进式对齐 oh-my-opencode

**核心理念对齐**：
- 从"工作流角色" → "问题类型 + 动态路由"
- 引入 Category 分类系统
- 增加执行保证机制

**阶段一：Agent 重命名 + Category 映射**
- 保持现有功能，逐步引入 Category 概念
- 新增 `problem-classifier` (可复用 scout)

**阶段二：引入 Prometheus 风格规划**
- 复杂任务先分析 + 生成计划
- 用户确认后再执行

**阶段三：Ralph Loop 执行保证**
- 任务自动继续直到完成
- 失败自动重试 + 升级

---

## 七、实施风险与回滚

**风险**：
- 破坏现有工作流
- 模型调用成本增加
- 调试复杂度提升

**回滚方案**：
- 保留原有 Agent 配置
- 使用配置文件切换新旧模式

---

## 八、验证计划

### 8.1 功能测试
- 问题分类准确性
- Plan Agent 任务分解质量
- 动态路由正确性

### 8.2 回归测试
- 现有工作流不受影响
- 原有 Agent 功能正常

---

## 九、总结

oh-my-opencode 是目前最成熟的 open-code 框架之一，其核心创新在于：

1. **Agent 专业化**: Sisyphus 编排 + Hephaestus 执行 + Prometheus 规划 + Oracle 决策
2. **Category 驱动**: 问题类型自动映射到最佳模型
3. **执行保证**: Ralph Loop 确保任务完成
4. **工程化工具**: Hashline、IntentGate 等消除常见错误

**OpenClaw 可逐步借鉴这些特性，从 Category 映射开始，渐进式对齐。**

---

## 十一、重大发现：已有现成方案！

### 🚀 OpenAgentsControl (OAC) - 正是我们要找的！

GitHub: https://github.com/darrenhinde/OpenAgentsControl

**这是一个 100% 契合需求的现成方案：**

| 特性 | OAC 提供 | 对应 oh-my-opencode 特性 |
|------|----------|-------------------------|
| **构建于 OpenCode** | ✅ 原生支持 | - |
| **Context-Aware** | ✅ 加载用户编码模式 | Category 映射 |
| **Approval Gates** | ✅ 先计划→审批→执行 | Prometheus 规划 |
| **MVI Token 效率** | ✅ 80% token 减少 | - |
| **6-stage workflow** | ✅ 完整工作流 | Ralph Loop |
| **7 专业子代理** | ✅ | 四神架构 |
| **Claude Code Plugin** | ✅ 可用 | - |

### OAC 核心创新

1. **ContextScout** - 智能模式发现，加载用户的编码规范
2. **Approval Gates** - 人类审查后再执行（不是自动执行）
3. **MVI 原则** - 只加载需要的，80% token 减少
4. **Team Ready** - 团队共享编码规范

### 安装方式

```bash
# 作为 OpenCode 插件安装
curl -fsSL https://raw.githubusercontent.com/darrenhinde/OpenAgentsControl/main/install.sh | bash -s developer

# 或作为 Claude Code 插件（BETA）
/plugin marketplace add darrenhinde/OpenAgentsControl
/plugin install oac
```

### 对比 OAC vs 自研

| 维度 | OAC (现成) | 自研方案 |
|------|------------|----------|
| **时间** | 即装即用 | 2-3 周 |
| **成熟度** | 生产可用 | 需验证 |
| **维护** | 社区维护 | 自主维护 |
| **定制** | 可编辑 Markdown | 完全控制 |
| **团队** | 有文档/支持 | 从零开始 |

---

## 十二、建议行动

### 推荐：直接采用 OAC

理由：
1. ✅ **0 开发成本** - 直接安装使用
2. ✅ **100% 兼容** - Built on OpenCode
3. ✅ **生产验证** - 有用户基础
4. ✅ **持续维护** - 活跃开发
5. ✅ **可定制** - Markdown 文件可编辑

### OAC 安装后的价值

安装 OAC 后即刻获得：
- [ ] ContextScout 智能模式发现
- [ ] 6-stage 工作流 + 审批门控
- [ ] 7 专业子代理
- [ ] 80% token 减少
- [ ] 团队规范共享

---

## 十三、下一步

**建议：安装 OAC 插件，体验后再决定是否需要自研增强。**

```bash
# 快速体验
curl -fsSL https://raw.githubusercontent.com/darrenhinde/OpenAgentsControl/main/install.sh | bash -s developer
```

方案已更新。
