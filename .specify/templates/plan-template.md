# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**注意**：此模板由`/speckit.plan`命令填充。执行工作流请参见`.specify/templates/commands/plan.md`。

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  所需操作：用项目的技术细节替换此部分内容。
  此处结构以咨询性质呈现，用于指导迭代过程。
-->

**语言/版本**: [例如，Python 3.11, Swift 5.9, Rust 1.75 或需要澄清]
**主要依赖**: [例如，FastAPI, UIKit, LLVM 或需要澄清]
**存储**: [如适用，例如，PostgreSQL, CoreData, 文件 或 N/A]
**测试**: [例如，pytest, XCTest, cargo test 或需要澄清]
**目标平台**: [例如，Linux服务器, iOS 15+, WASM 或需要澄清]
**项目类型**: [单项目/web/移动端 - 决定源码结构]
**性能目标**: [领域特定，例如，1000 请求/秒, 10k 行/秒, 60 fps 或需要澄清]
**约束**: [领域特定，例如，<200ms p95, <100MB 内存, 离线可用 或需要澄清]
**规模/范围**: [领域特定，例如，10k用户, 1M代码行, 50个屏幕 或需要澄清]

## Constitution Check

*门禁：必须在阶段0研究前通过。阶段1设计后重新检查。*

[基于宪章文件确定的门禁]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # 此文件（/speckit.plan 命令输出）
├── research.md          # 阶段0输出（/speckit.plan 命令）
├── data-model.md        # 阶段1输出（/speckit.plan 命令）
├── quickstart.md        # 阶段1输出（/speckit.plan 命令）
├── contracts/           # 阶段1输出（/speckit.plan 命令）
└── tasks.md             # 阶段2输出（/speckit.tasks 命令 - 非 /speckit.plan 创建）
```

### 源码（仓库根目录）
<!--
  所需操作：用功能的实际布局替换下方的占位符树。
  删除未使用的选项并用实际路径扩展所选结构（例如，apps/admin, packages/something）。
  交付的计划不得包含选项标签。
-->

```text
# [如未使用请删除] 选项1：单项目（默认）
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [如未使用请删除] 选项2：Web应用（当检测到"前端"+"后端"时）
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [如未使用请删除] 选项3：移动端+API（当检测到"iOS/Android"时）
api/
└── [same as backend above]

ios/ or android/
└── [平台特定结构：功能模块、UI流、平台测试]
```

**结构决策**: [记录所选结构并引用上面捕获的实际目录]

## Complexity Tracking

> **仅在宪章检查有必须证明的违规时填写**

| 违反 | 为什么需要 | 被拒绝的更简单替代方案及原因 |
|-----------|------------|-------------------------------------|
| [例如，第4个项目] | [当前需求] | [为什么3个项目不够] |
| [例如，仓储模式] | [具体问题] | [为什么直接数据库访问不够] |
