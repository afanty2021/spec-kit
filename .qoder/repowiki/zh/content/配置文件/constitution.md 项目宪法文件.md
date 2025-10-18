# constitution.md 项目宪法文件

<cite>
**本文档引用文件**  
- [memory/constitution.md](file://memory/constitution.md)
- [templates/commands/constitution.md](file://templates/commands/constitution.md)
- [templates/plan-template.md](file://templates/plan-template.md)
- [templates/spec-template.md](file://templates/spec-template.md)
- [templates/tasks-template.md](file://templates/tasks-template.md)
- [README.md](file://README.md)
- [spec-driven.md](file://spec-driven.md)
</cite>

## 目录
1. [引言](#引言)
2. [项目宪法的核心地位](#项目宪法的核心地位)
3. [核心原则与约束条款](#核心原则与约束条款)
4. [在SDD流程中的作用机制](#在sdd流程中的作用机制)
5. [斜杠命令工作流中的应用](#斜杠命令工作流中的应用)
6. [维护与更新最佳实践](#维护与更新最佳实践)
7. [结论](#结论)

## 引言
在规范驱动开发（SDD）方法论中，`memory/constitution.md` 文件扮演着项目“宪法”的核心角色。该文件不仅定义了项目的开发原则、编码规范和技术栈约束，还作为AI代理在自动生成代码和文档时必须遵循的根本准则。本文档深入阐述该文件的实际用途、结构设计及其在整个开发流程中的关键作用。

**Section sources**
- [README.md](file://README.md#L20-L583)
- [spec-driven.md](file://spec-driven.md#L20-L404)

## 项目宪法的核心地位
`memory/constitution.md` 是整个项目架构的“DNA”，它确立了一套不可协商的基本原则，确保所有生成的实现保持一致性、简洁性和高质量。该文件是所有开发活动的最高权威，任何与宪法冲突的设计或实现都必须被修正，而非对原则进行稀释或忽略。

该文件通过模板化结构定义了项目的核心价值观，包括但不限于：库优先原则、CLI接口强制、测试先行（TDD）、集成测试优先、可观测性要求、版本管理策略以及极简主义设计哲学。这些原则共同构成了项目的技术治理框架。

**Section sources**
- [memory/constitution.md](file://memory/constitution.md#L0-L49)
- [spec-driven.md](file://spec-driven.md#L268-L299)

## 核心原则与约束条款
### 核心原则
宪法文件定义了若干核心原则，这些原则在开发过程中具有最高优先级：

#### I. 库优先原则（Library-First）
所有功能必须首先作为独立库实现，禁止直接在应用代码中实现功能。这强制从一开始就进行模块化设计，确保代码的可重用性和独立可测试性。

#### II. CLI接口强制（CLI Interface Mandate）
每个库必须通过命令行接口暴露其功能，输入/输出必须为文本格式（支持JSON和人类可读格式），确保所有功能都可通过标准输入/输出进行访问和验证。

#### III. 测试先行（Test-First, NON-NEGOTIABLE）
必须严格遵循测试驱动开发（TDD）流程：先编写测试 → 用户批准 → 测试失败 → 再实现代码。红-绿-重构循环被严格强制执行。

#### IV. 集成测试优先（Integration-First Testing）
优先使用真实环境进行测试：偏好真实数据库而非模拟，使用实际服务实例而非存根，合同测试在实现前必须完成。

#### V. 简单性与反抽象（Simplicity & Anti-Abstraction）
- 初始实现最多使用3个项目，额外项目需提供文档化理由。
- 直接使用框架功能，避免不必要的包装层。
- 单一模型表示，避免过度抽象。

**Section sources**
- [memory/constitution.md](file://memory/constitution.md#L10-L38)
- [spec-driven.md](file://spec-driven.md#L343-L371)

### 典型约束条款示例
- **禁止使用 `eval()`**：为防止代码注入和安全漏洞，明确禁止动态代码执行。
- **必须编写单元测试**：所有新功能必须附带单元测试，且测试必须先于实现代码编写。
- **禁止未来主义设计（No Future-Proofing）**：仅实现当前需求，不为“可能需要”的功能做设计。
- **依赖管理**：所有外部依赖必须经过审查，避免引入不必要的复杂性。

## 在SDD流程中的作用机制
在规范驱动开发（SDD）流程中，AI代理在生成代码和文档时，必须主动读取并遵循 `constitution.md` 中的规则。该文件的作用机制如下：

1. **原则验证**：在生成任何实现计划或代码前，AI代理必须验证其提议是否符合宪法原则。
2. **自动检查点**：在 `/plan` 和 `/tasks` 命令执行过程中，系统会自动进行“宪法检查”（Constitution Check），确保设计不违反核心原则。
3. **偏差记录**：如果设计必须违反某项原则（如需要超过3个项目），则必须在“复杂性跟踪”（Complexity Tracking）部分明确记录理由，并说明为何更简单的替代方案不可行。

该机制确保了即使在快速迭代中，项目的架构完整性也能得到维护，有效减少了技术债务的积累。

```mermaid
flowchart TD
A[用户需求] --> B[/specify 命令]
B --> C[生成 spec.md]
C --> D[/plan 命令]
D --> E[读取 constitution.md]
E --> F[宪法检查]
F --> G{通过?}
G --> |是| H[生成 plan.md]
G --> |否| I[记录复杂性偏差]
I --> H
H --> J[/tasks 命令]
J --> K[生成 tasks.md]
K --> L[/implement 命令]
L --> M[生成代码]
```

**Diagram sources**
- [templates/plan-template.md](file://templates/plan-template.md#L20-L225)
- [templates/commands/constitution.md](file://templates/commands/constitution.md#L10-L73)

**Section sources**
- [spec-driven.md](file://spec-driven.md#L20-L404)
- [templates/commands/analyze.md](file://templates/commands/analyze.md#L17-L19)

## 斜杠命令工作流中的应用
`constitution.md` 文件在 `/specify`、`/plan`、`/tasks` 等斜杠命令工作流中发挥着关键作用：

### `/constitution` 命令
此命令专门用于创建或更新项目宪法。它会：
- 加载现有的宪法模板
- 收集或推导占位符值
- 精确填充模板
- 将任何修改传播到依赖的模板文件中（如 `plan-template.md`、`spec-template.md` 等）
- 生成同步影响报告，记录版本变更、修改的原则、需更新的模板等

### `/plan` 命令
在生成实现计划时，`/plan` 命令会：
- 读取 `constitution.md` 以理解宪法要求
- 在计划中填充“宪法检查”（Constitution Check）部分
- 在执行流程中评估宪法检查，若存在违规则记录在“复杂性跟踪”中
- 只有通过宪法检查，计划才能继续

### `/analyze` 命令
此命令在任务生成后执行，会对 `spec.md`、`plan.md` 和 `tasks.md` 进行一致性分析。宪法在此被视为**不可协商的权威**，任何与宪法的冲突都会被标记为**严重问题**，必须调整规范、计划或任务来解决。

### `/tasks` 命令
此命令生成的任务列表会反映宪法驱动的任务类型，例如：
- 合同测试任务（源于“集成测试优先”原则）
- 可观测性任务（源于“CLI接口强制”原则）
- 版本管理任务（源于“版本化与破坏性变更”原则）

**Section sources**
- [templates/commands/constitution.md](file://templates/commands/constitution.md#L10-L73)
- [templates/plan-template.md](file://templates/plan-template.md#L20-L225)
- [templates/tasks-template.md](file://templates/tasks-template.md#L10-L127)

## 维护与更新最佳实践
### 版本管理
宪法的版本必须遵循语义化版本控制规则：
- **主版本（MAJOR）**：向后不兼容的治理原则或核心原则的移除或重新定义。
- **次版本（MINOR）**：新增原则/章节或对指导方针进行实质性扩展。
- **补丁版本（PATCH）**：澄清、措辞、拼写错误修复、非语义性改进。

### 修改流程
对宪法的修改必须经过严格流程：
1. **明确记录变更理由**：详细说明为何需要修改原则。
2. **维护者审查与批准**：必须由项目维护者进行审查和批准。
3. **向后兼容性评估**：评估变更对现有代码和流程的影响。
4. **同步更新依赖项**：使用 `/constitution` 命令自动更新所有依赖的模板文件。

### 一致性传播
更新宪法后，必须验证并更新所有依赖的模板：
- `plan-template.md`：确保“宪法检查”规则与更新后的原则一致。
- `spec-template.md`：确保范围/需求与新增或移除的约束对齐。
- `tasks-template.md`：确保任务分类反映新的原则驱动任务类型。
- 所有命令文件：验证是否仍存在过时的引用。

**Section sources**
- [templates/commands/constitution.md](file://templates/commands/constitution.md#L20-L60)
- [spec-driven.md](file://spec-driven.md#L373-L396)

## 结论
`memory/constitution.md` 文件是规范驱动开发（SDD）方法论的基石。它不仅仅是一份文档，更是一种开发哲学的体现，将AI从单纯的代码生成器转变为尊重并强化系统设计原则的架构伙伴。通过将核心原则制度化、自动化检查和强制执行，该项目宪法确保了生成的代码不仅是功能性的，更是可维护、可测试且架构健全的。持续维护和遵循此文件，是确保项目长期健康和减少技术债务的关键。