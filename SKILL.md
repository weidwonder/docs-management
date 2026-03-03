---
name: docs-management
description: 指导 Claude Code 在项目开发过程中如何创建、更新、管理 docs 文件的最佳实践。
---

# Docs 管理最佳实践

> 本文档指导 Claude Code 在项目开发过程中如何创建、更新、管理 docs 文件。

## 核心原则

### 1. 最小化披露原则

**目标**: 减少大模型处理文档时的上下文消耗，提高效率。

- **入口文档** (CLAUDE.md) 只包含立即需要的信息
- **详细内容** 通过链接渐进式披露到专门文档
- **避免重复** 同一信息只在一处维护
- **避免冗余** 对于你已经知道的常识、开发实践等，不要写到文档中

```
❌ 错误: 在 CLAUDE.md 中写 500 行架构说明
✅ 正确: CLAUDE.md 写 10 行概述 + 链接到 docs/architecture.md
```

### 2. 渐进式披露原则

**目标**: 让大模型按需获取信息，而非一次性加载所有文档。

文档层次结构：
```
第1层: CLAUDE.md (入口 + 导航)
       ↓ 需要时才读取
第2层: docs/*.md (核心文档)  或 README.md
       ↓ 需要时才读取
第3层: docs/guides/*.md (辅助文档)
       docs/chat-service/*.md (模块专用)
```

### 3. 单一职责原则

**目标**: 每个文档只负责一个主题，便于维护和定位。

| 文档类型 | 职责 | 示例 |
|---------|------|------|
| 入口文档 | 导航 + 核心概念 | CLAUDE.md |
| 架构文档 | 系统设计 | architecture.md |
| 需求文档 | 功能规格 | requirements.md |
| 指南文档 | 操作步骤 | guides/deployment.md |
| 模块文档 | 子系统详情 | chat-service/api-docs.md |

---

## 目录结构规范

### 标准结构

```
项目根目录/
├── CLAUDE.md              # 【必需】大模型开发入口
├── QUICKSTART.md          # 【可选】快速开始指南
└── docs/
    ├── architecture.md    # 【必需】系统架构，没有的时候需要完整理解项目进行构建。
    ├── requirements.md    # 【必需】功能需求，没有的时候需要完整理解项目进行构建。
    ├── personal.md        # 用于记录当前主机环境、认证信息、当前环境需要持久化记录的特殊情况等。
    ├── changelog.md       # 变更日志
    ├── guides/            # 辅助指南
    │   ├── deployment.md
    │   ├── code-standards.md
    │   └── ...
    ├── {module-name}/     # 模块专用文档
    │   ├── api-docs.md
    │   ├── architecture.md
    │   └── ...
    └── plans/             # 开发计划
        └── ...
```

### 目录职责

| 目录 | 用途 | 生命周期 |
|-----|------|---------|
| `docs/` | 核心文档 | 长期维护 |
| `docs/guides/` | 操作指南、问题解决方案 | 长期维护 |
| `docs/{module}/` | 子系统/模块专用文档 | 随模块生命周期 |
| `docs/plans/` | 实施计划、开发记录 | 完成后可删除 |

### 多插件环境下的两区制（推荐）

**背景**：当项目同时使用 bmad、superpowers、oh-my-claudecode 等多个插件时，各插件会通过 hook 向每次对话注入各自的文档路径约定（如 bmad 写 `_bmad-output/`，superpowers 写 `docs/plans/`，omc 写 `plans/`），AI 无法判断优先级，导致文档散落各处。

**解法：两区制 + 仲裁规则文件**

```
{project}/
├── dev_docs/          # 开发者/AI 协作文档（开发计划、架构、需求等）
│   ├── artifacts/     # 产品规划文档（PRD、架构、需求）
│   ├── implementation/# 实现计划、Sprint 工件
│   ├── reports/       # Agent 报告
│   └── ...
├── run_docs/          # 用户/运维文档（部署、操作手册）
└── .claude/rules/doc-routing.md  # 仲裁规则（见下）
```

**目录命名可根据项目自定义**（如保留 `docs/` + `dev_docs/`），核心是"两区分离"。

**必须创建 `.claude/rules/doc-routing.md`**，内容模板：

```markdown
# 文档路由规则（最高优先级，覆盖所有插件默认行为）

## 两区制
| 目录 | 受众 | 内容 |
|------|------|------|
| `run_docs/` | 用户/运维 | 部署、操作手册 |
| `dev_docs/` | 开发者/AI | 规划、计划、报告 |

## 覆盖规则
- superpowers:writing-plans 默认路径 `docs/plans/` → 改为 `dev_docs/`
- oh-my-claudecode hook 注入路径 `plans/` → 实际使用 `dev_docs/`
- bmad 输出路径 → 在 `_bmad/bmm/config.yaml` 中配置为 `dev_docs/`

## 禁止
- 不在 `run_docs/` 创建计划/报告类文档
- 不在 `dev_docs/` 创建用户手册/部署指南
```

**为什么这样有效**：`.claude/rules/` 内容在每次对话注入优先级高于插件 hook，AI 遇到冲突时遵守项目本地规则。

**更新抗性**：只改项目文件（`CLAUDE.md`、`.claude/rules/`、插件的项目级 config），不改插件 cache 文件，插件更新后规则依然生效。

**识别文件来源**：靠目录命名空间（bmad → `_bmad-output/` 或配置的 `dev_docs/artifacts/`），不靠内容判断。

**全局规则**：`~/.claude/CLAUDE.md` 可写跨项目默认约定，项目 `CLAUDE.md` 覆盖它。

---

## 文件命名规范

### 命名规则

```
✅ 正确:
- architecture.md
- code-standards.md
- mcp-remote-guide.md
- api-docs.md

❌ 错误:
- 系统架构.md          # 避免中文文件名
- Architecture.md      # 避免大写
- code_standards.md    # 使用连字符而非下划线
- arch.md              # 避免过度缩写
```

### 命名模式

| 类型 | 命名模式 | 示例 |
|-----|---------|------|
| 架构文档 | `architecture.md` | `architecture.md`, `chat-service/architecture.md` |
| API 文档 | `api-docs.md` 或 `api-reference.md` | `api-docs.md` |
| 用户指南 | `user-guide.md` | `user-guide.md` |
| 本地环境特殊记录 | `personal.md` | `personal.md` |
| 开发指南 | `{topic}-guide.md` | `mcp-remote-guide.md` |
| 问题解决 | `{topic}-debugging.md` | `mcp-dotnet-debugging.md` |
| 变更日志 | `changelog.md` | `changelog.md` |
| 需求文档 | `requirements.md` | `requirements.md` |
| 开发记录 | `pdr.md` (Product Development Record) | `pdr.md` |

---

## CLAUDE.md 编写规范

### 必需章节

```markdown
# CLAUDE.md

> 一句话项目描述

## 📖 文档导航系统
[决策树式导航，指导何时读取哪个文档]

## 📚 项目概述
[核心特性，不超过 5 点]

## 🗂️ 项目结构
[目录布局 + 子系统说明]

## 🎯 核心概念
[关键术语定义，每个不超过 2 行]

## 🔧 核心 API (可选)
[最常用的 API，只展示签名和用法]

## 📝 更多信息 (可选)
[链接到详细文档]
```

### 文档导航设计

```markdown
## 📖 文档导航系统

### 阅读决策树

**首次开发任务**:
1. ✅ 阅读本文档 (CLAUDE.md)
2. → 阅读 QUICKSTART.md

**理解系统架构**:
→ 阅读 `docs/architecture.md`

**特定问题**:
- 部署问题 → `docs/guides/deployment.md`
- API 参考 → `docs/chat-service/api-docs.md`

**{模块名} 开发**:
- API 参考 → `docs/{module}/api-docs.md`
- 详细架构 → `docs/{module}/architecture.md`
```

### 行数控制

| 章节 | 建议行数 | 超出处理 |
|-----|---------|---------|
| 文档导航 | 30-50 行 | - |
| 项目概述 | 10-20 行 | - |
| 项目结构 | 30-60 行 | 复杂结构拆分到 architecture.md |
| 核心概念 | 20-40 行 | 详细说明拆分到专门文档 |
| 核心 API | 50-100 行 | 完整 API 拆分到 api-docs.md |
| **总计** | **200-400 行** | 超出需要精简或拆分 |

---

## 特殊文档技巧

### 需求文档 (requirements.md)

- 采用无序序号：由于需求文档可能会经常增加或者删除特性，所以不要用序号类的需求ID, 应采用 PRD-<特性大类>-<子类>-<YYMMDD><INDEX>的格式, 比如 登录 中的管理员属性控制的需求，26年2月5日创建，可能命名为 PRD-LOGIN-ADMIN-25020501， 避免需求增减需要改动其他需求的ID且可以保持唯一。

### 本地环境特殊记录 (personal.md)

- 用于记录当前主机环境、认证信息、当前环境需要持久化记录的特殊情况等。
- ** 注意 **：
    - 一旦定义了该文档，请务必要在 CLAUDE.md、AGENTS.md 中添加导航链接。且要求AI编码Agent必读。

### 需求文档 

## 文档生命周期管理（重要！！）

### 创建文档

**触发条件**:
1. 新功能开发完成，需要记录架构或用法
2. 遇到复杂问题，需要记录解决方案
3. 需要为特定模块建立专用文档

**创建流程**:
```
1. 确定文档类型和存放位置并 **取得用户同意**。
2. 使用标准命名
3. 编写内容
4. 在 CLAUDE.md 中添加导航链接
```

### 更新文档

**每次完成开发或者bug修正任务后**:
- review 计划文档，如果有偏离更新计划文档。
- review 需求文档，如果有更新，按照最小化更新原则，言简意赅的修订需求文档。
- review 架构文档，如果有更新，按照最小化更新原则，言简意赅的修订架构文档。如果涉及到多个模块的基础组件的更新，则需要适当的检视是否存在对其他模块的影响。并相比其他模块描述相对详细一些。
- 若存在文档增减，检查 CLAUDE.md 导航是否需要更新

**更新原则**:
- 小的更新、修改，可以不更新文档。以下情况列示：
    - 增加、删除、修改特性： 需要修订需求文档、视情况更新架构文档
    - 问题修正： 小问题不需要更新，大问题、需要编码时避坑的问题需要作为注意事项更新到架构文档中。
    - 调整了公共部分的底层模块：如果有该模块的文档，必须要更新，如果没有，则提示用户应该创建。

### 删除/归档文档

**应删除的文档**:
- ✅ 已完成的实施计划 (`docs/plans/YYY-MM-DD-HHMM-<name>-plan.md`)
- ✅ 过时的版本文档（被新版本替代）
- ✅ 重复内容（已合并到其他文档）

**应保留的文档**:
- ❌ 不要删除正在使用的 API 文档
- ❌ 不要删除 changelog（历史记录）
- ❌ 不要删除架构文档（除非完全重写）

**归档流程**:
```
1. 确认文档已过时或不再需要
2. 检查是否有其他文档引用它
3. 更新 CLAUDE.md 移除导航链接
4. 删除文件
```

---

## 针对大模型的优化

### 文档长度控制

| 文档类型 | 建议行数 | 超出处理 |
|---------|---------|---------|
| CLAUDE.md | 200-400 行 | 拆分到专门文档 |
| 核心文档 | 500-1500 行 | 拆分章节到子文档 |
| 指南文档 | 200-500 行 | 保持聚焦单一主题 |
| API 文档 | 300-800 行 | 按模块拆分 |

### 信息密度优化

```markdown
❌ 低密度 (冗余):
## 用户认证模块详细说明
本模块负责处理用户的认证相关功能。用户认证是系统中非常重要的一个模块，
它确保只有合法的用户才能访问系统资源。下面我们将详细介绍这个模块的各个方面...

✅ 高密度 (精炼):
## 用户认证模块
- 职责: JWT 令牌签发与验证
- 入口: `auth_router.py`
- 配置: `AUTH_SECRET_KEY` 环境变量
```

### 结构化表达

优先使用：
- ✅ 表格（对比信息）
- ✅ 列表（步骤、要点）
- ✅ 代码块（示例、配置）
- ✅ ASCII 图（架构图）

避免使用：
- ❌ 长段落文字
- ❌ 嵌套过深的列表
- ❌ 模糊的描述

---

## 总结

1. **CLAUDE.md 是唯一入口** - 所有文档导航从这里开始
2. **最小化披露** - 只展示必需信息，详情通过链接获取
3. **按职责组织** - 框架级、模块级、指南类文档分开存放
4. **定期清理** - 删除过时文档，合并重复内容
5. **保持同步** - 代码变更时同步更新文档

---

## 附加资源

- **工作流与检查清单** → `reorganization.md`

---

*Skill Version: 1.0*
*Last Updated: 2026-02-15*
