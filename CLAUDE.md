# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

⚠️ **重要说明**：本文档描述的是项目所使用的开发框架（Specify Framework）及开发模式规范，**不包含项目自身的业务功能**。所有后续生成的文档、代码注释均应使用中文。

## 开发规范概述

本项目采用 **Specify Framework**（规范驱动开发框架，SDD）——一个通过结构化工作流程实现 AI 引导功能开发的元框架。它提供了模板、脚本和 AI 提示，用于管理软件开发生命周期，采用规范优先的方法。

## 开发规范核心架构

该框架实现了跨5个阶段的顺序工作流，每个阶段都建立在前一个阶段之上：

1. **Specify**（规范制定，`/speckit.specify`）→ 从自然语言创建功能规范
2. **Clarify**（澄清确认，`/speckit.clarify`）→ 通过定向问题解决歧义
3. **Plan**（计划制定，`/speckit.plan`）→ 生成技术实现计划
4. **Tasks**（任务分解，`/speckit.tasks`）→ 分解为可执行的、依赖有序的任务
5. **Implement**（实施执行，`/speckit.implement`）→ 执行任务列表并设置验证检查点

### 关键目录结构

```
./
├── .specify/                  # 框架基础设施
│   ├── scripts/bash/          # 核心 bash 脚本
│   │   ├── common.sh          # 共享工具和函数
│   │   ├── check-prerequisites.sh    # 验证工作流状态
│   │   ├── create-new-feature.sh     # 初始化功能分支/目录
│   │   ├── setup-plan.sh            # 第一阶段设置脚本
│   │   └── update-agent-context.sh  # 同步 AI 代理上下文文件
│   ├── templates/             # 文档模板
│   │   ├── spec-template.md          # 功能规范模板
│   │   ├── plan-template.md          # 实施计划模板
│   │   ├── tasks-template.md         # 任务分解模板
│   │   └── checklist-template.md     # 质量验证模板
│   └── memory/
│       └── constitution.md           # 项目治理原则
├── .github/prompts/           # 每个工作流命令的 AI 代理提示定义
├── specs/[###-feature]/       # 每个功能的文档（自动生成）
└── .claude/commands/          # 自动生成的 AI 命令快捷方式
```

## 常用命令与脚本

所有脚本使用**绝对路径**，并支持通过 `--json` 标志提供**JSON输出**以便程序化使用。

### 核心脚本

```bash
# 初始化新功能（创建分支 + 功能目录）
./.specify/scripts/bash/create-new-feature.sh "feature description"

# 检查工作流先决条件
./.specify/scripts/bash/check-prerequisites.sh --json

# 验证实施阶段（需要 tasks.md）
./.specify/scripts/bash/check-prerequisites.sh --json --require-tasks

# 更新 AI 代理上下文文件
./.specify/scripts/bash/update-agent-context.sh claude

# 仅获取功能路径（无验证）
./.specify/scripts/bash/check-prerequisites.sh --paths-only
```

### 可用命令（斜杠命令）

该框架支持以下 AI 代理命令（通过 `/` 在 Claude/Cursor 中访问）：

- `/speckit.specify` - 从自然语言创建功能规范
- `/speckit.clarify` - 通过定向问题解决歧义
- `/speckit.plan` - 生成技术实施计划
- `/speckit.tasks` - 分解为可执行的、依赖有序的任务
- `/speckit.implement` - 执行任务列表并验证
- `/speckit.checklist` - 生成质量验证清单
- `/speckit.analyze` - 跨工件一致性分析

每个命令的提示定义在 `.github/prompts/` 中。

## 功能分支工作流

**分支命名规范**：`###-描述性名称`（例如 `001-user-auth`、`002-payment-processing`）

功能使用零填充的3位数字前缀（001, 002, 003...）。脚本自动检测下一个可用编号。

```bash
# 创建新功能（自动创建正确命名的分支）
./.specify/scripts/bash/create-new-feature.sh "Add user authentication system"

# 对于非 git 仓库，设置环境变量
export SPECIFY_FEATURE="001-user-authentication"
```

## 每个功能的文档结构

当你运行 `/speckit.specify` 时，它会创建：

```
specs/[###-功能名称]/
├── spec.md              # 业务需求和用户故事（带 P1/P2/P3 优先级）
├── plan.md              # 技术架构和决策
├── tasks.md             # 实施任务分解（由 /speckit.tasks 生成）
├── research.md          # 技术研究和决策
├── data-model.md        # 实体关系和验证
├── quickstart.md        # 集成场景
├── contracts/           # API 规范（OpenAPI/GraphQL）
└── checklists/          # 质量验证清单
```

## 宪章原则

该框架受 **Specify 宪章**（`.specify/memory/constitution.md`）治理。核心原则：

1. **用户故事驱动开发** - 所有功能必须具有可独立测试的用户故事，并具有明确的 P1/P2/P3 优先级
2. **测试优先（强制）** - 所有故事都需要 TDD 周期；严格执行红-绿-重构
3. **独立可测试性** - 每个故事必须能够独立测试
4. **分阶段实施** - 有序阶段：设置 → 基础 → 用户故事 → 完善
5. **简洁性与宪章合规** - 倾向于简洁；所有决策根据宪章验证

**宪章检查** 是 `plan.md` 中的强制门槛 - 违反必须书面说明理由。

## 开发指南

### 规范制定

- **用户故事必须有优先级**（P1, P2, P3）以实现独立交付
- **需求必须可测试** - 避免没有指标的模糊形容词
- **成功标准必须可衡量** - 量化性能/UX 目标
- **每个故事应该能够独立实施**

### 任务组织

- **分阶段执行**：设置 → 基础 → 用户故事 → 完善
- **并行标记 `[P]`** 表示可同时运行的任务
- **故事映射 `[US1]`, `[US2]`** 启用独立故事交付
- **依赖关系必须在任务描述中明确**

### 脚本集成点

`.specify/scripts/bash/` 中的所有 bash 脚本遵循这些模式：
- 在所有脚本调用中使用**绝对路径**
- 支持通过 `--json` 标志提供**JSON输出**以便程序化使用
- 包含带有清晰错误代码和消息的**错误处理**
- 通过 `SPECIFY_FEATURE` 环境变量为非 git 仓库提供**回退支持**

## 多Agent支持

该框架为不同的 AI Agent 维护单独的上下文文件：
- **GitHub Copilot**: `.github/copilot-instructions.md`
- **Claude**: `CLAUDE.md`（此文件）
- **Cursor**: `.cursor/rules/specify-rules.mdc`
- **其他**：参见 `update-agent-context.sh` 获取完整列表

在修改宪章或模板后运行 `.specify/scripts/bash/update-agent-context.sh claude` 以同步代理指令。

## 错误处理与恢复

### 常见问题

- **分支不符合功能格式** → 功能分支必须匹配 `###-name` 模式
- **缺少先决条件** → 运行 `check-prerequisites.sh` 验证工作流状态
- **宪章违规** → 在继续之前解决 plan.md 中的原则冲突
- **清单不完整** → 质量门槛必须通过才能实施

### 恢复模式

- **从规范重新开始** - 重新运行 `/speckit.specify` 从需求重建
- **增量修复** - 使用 `/speckit.clarify` 解决规范歧义
- **任务重新生成** - 计划更改后重新运行 `/speckit.tasks`

## 质量门槛与审查流程

每个实施都经过：

1. **宪章合规检查** - 所有计划根据 `.specify/memory/constitution.md` 验证
2. **覆盖验证** - 每个需求必须映射到任务
3. **清单验证** - 质量清单必须在实施前通过
4. **可追溯性** - 需求 → 任务 → 实施链接

## 宪章驱动开发

`.specify/memory/constitution.md` 文件定义了**不可协商的项目原则**。所有计划阶段都根据这些原则和明确的合规门槛进行验证。

宪章治理：
- **超越所有其他实践**
- **修订要求**：变更文档 + 理由 + 语义版本递增
- **主版本**：原则移除/重新定义
- **次版本**：新原则
- **修订版本**：澄清

## 文件命名规范

- **功能 ID**：零填充的3位数字前缀（001, 002, 003...）
- **模板**：`.specify/templates/` 中的所有文件，后缀为 `-template.md`
- **脚本**：`.specify/scripts/bash/` 中的 bash 脚本，扩展名为 `.sh`
- **提示**：`.github/prompts/` 中的 AI 命令，后缀为 `.prompt.md`

---

**版本**：1.0.0 | **框架**：Specify SDD | **最后更新**：2025-10-31
