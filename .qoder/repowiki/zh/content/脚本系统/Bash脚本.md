# Bash脚本

<cite>
**本文档中引用的文件**   
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh)
- [get-feature-paths.sh](file://scripts/bash/get-feature-paths.sh)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh)
- [common.sh](file://scripts/bash/common.sh)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构概述](#架构概述)
5. [详细组件分析](#详细组件分析)
6. [依赖分析](#依赖分析)
7. [性能考虑](#性能考虑)
8. [故障排除指南](#故障排除指南)
9. [结论](#结论)

## 简介
本文档详细说明了Spec-Driven Development（SDD）工具包中的Bash脚本系统。这些脚本构成了自动化开发流水线的核心，通过标准化的流程协调从功能创建到AI代理上下文更新的各个阶段。脚本设计遵循意图驱动开发原则，在编码之前优先定义"做什么"而非"如何做"。系统包含五个主要功能脚本和一个共享函数库，支持JSON模式输出以实现与其他工具的集成，并通过严格的分支命名约定（如001-feature-name）确保工作流的一致性。

## 项目结构
Bash脚本系统位于`scripts/bash/`目录下，与PowerShell脚本并列，为不同操作系统提供跨平台支持。脚本通过`common.sh`共享核心功能，并与`templates/`目录中的文档模板协同工作，形成完整的SDD自动化流水线。

```mermaid
graph TB
subgraph "脚本目录"
Bash[scripts/bash]
Powershell[scripts/powershell]
end
subgraph "模板目录"
Templates[templates]
Commands[templates/commands]
AgentTemplate[agent-file-template.md]
PlanTemplate[plan-template.md]
SpecTemplate[spec-template.md]
TasksTemplate[tasks-template.md]
end
subgraph "内存目录"
Memory[memory]
Constitution[constitution.md]
end
Bash --> Common[common.sh]
Bash --> CheckPrereq[check-task-prerequisites.sh]
Bash --> CreateFeature[create-new-feature.sh]
Bash --> GetPaths[get-feature-paths.sh]
Bash --> SetupPlan[setup-plan.sh]
Bash --> UpdateContext[update-agent-context.sh]
Common --> Git[git]
Common --> RepoRoot[REPO_ROOT]
Common --> CurrentBranch[CURRENT_BRANCH]
Templates --> PlanTemplate
Templates --> SpecTemplate
Templates --> TasksTemplate
Commands --> PlanCmd[plan.md]
Commands --> TasksCmd[tasks.md]
Commands --> SpecifyCmd[specify.md]
PlanTemplate --> UpdateContext
PlanCmd --> SetupPlan
TasksCmd --> CheckPrereq
AgentTemplate --> UpdateContext
Constitution --> SDDProcess
style Bash fill:#f9f,stroke:#333
style Powershell fill:#f9f,stroke:#333
style Templates fill:#bbf,stroke:#333
style Commands fill:#bbf,stroke:#333
style Memory fill:#bbf,stroke:#333
```

**Diagram sources**
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh)
- [get-feature-paths.sh](file://scripts/bash/get-feature-paths.sh)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh)
- [common.sh](file://scripts/bash/common.sh)

**Section sources**
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh)
- [get-feature-paths.sh](file://scripts/bash/get-feature-paths.sh)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh)
- [common.sh](file://scripts/bash/common.sh)

## 核心组件
核心组件由五个功能脚本和一个共享函数库组成。`common.sh`提供了路径解析、分支验证和文件检查等基础功能，被所有其他脚本引用。`create-new-feature.sh`负责初始化新功能的整个环境，包括创建Git分支和目录结构。`setup-plan.sh`协调计划阶段，为实施准备模板。`check-task-prerequisites.sh`在任务生成前验证系统依赖和环境配置。`update-agent-context.sh`作为AI集成的关键，动态更新多个AI代理的上下文文件。`get-feature-paths.sh`提供路径解析服务，确保所有脚本使用一致的路径约定。

**Section sources**
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh#L1-L58)
- [get-feature-paths.sh](file://scripts/bash/get-feature-paths.sh#L1-L7)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh#L1-L17)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)
- [common.sh](file://scripts/bash/common.sh#L1-L37)

## 架构概述
Bash脚本系统采用模块化架构，以`common.sh`为核心共享库，其他脚本作为独立的功能模块。系统通过环境变量和函数调用进行通信，遵循Unix哲学的"做一件事并做好"原则。脚本间通过标准化的路径约定和JSON输出格式实现松耦合集成，支持与CLI工具和AI代理的无缝交互。

```mermaid
graph LR
CLI[CLI/用户] --> CreateFeature
CLI --> SetupPlan
CLI --> CheckPrereq
CLI --> UpdateContext
CLI --> GetPaths
CreateFeature --> Common
SetupPlan --> Common
CheckPrereq --> Common
UpdateContext --> Common
GetPaths --> Common
Common --> Git[Git命令]
Common --> FS[文件系统]
CreateFeature --> Template[spec-template.md]
SetupPlan --> Template2[plan-template.md]
UpdateContext --> Template3[agent-file-template.md]
UpdateContext --> Agents[Claude,Gemini,Copilot等]
style CreateFeature fill:#f96,stroke:#333
style SetupPlan fill:#f96,stroke:#333
style CheckPrereq fill:#f96,stroke:#333
style UpdateContext fill:#f96,stroke:#333
style GetPaths fill:#f96,stroke:#333
style Common fill:#69f,stroke:#333
style CLI fill:#6f9,stroke:#333
```

**Diagram sources**
- [common.sh](file://scripts/bash/common.sh#L1-L37)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh#L1-L58)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh#L1-L17)
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)

## 详细组件分析

### check-task-prerequisites.sh 分析
该脚本验证任务生成前的系统依赖和环境配置，确保SDD流程的完整性。

```mermaid
flowchart TD
Start([开始]) --> ParseArgs["解析命令行参数"]
ParseArgs --> SourceCommon["加载 common.sh"]
SourceCommon --> GetPaths["执行 get_feature_paths"]
GetPaths --> CheckBranch["验证功能分支"]
CheckBranch --> CheckFeatureDir{"功能目录存在?"}
CheckFeatureDir --> |否| Error1["报错: 运行 /specify"]
CheckFeatureDir --> |是| CheckPlan{"plan.md 存在?"}
CheckPlan --> |否| Error2["报错: 运行 /plan"]
CheckPlan --> |是| JsonMode{"JSON模式?"}
JsonMode --> |是| BuildJson["构建JSON输出"]
JsonMode --> |否| OutputList["输出可用文档列表"]
BuildJson --> OutputJson
OutputList --> End([结束])
OutputJson --> End
Error1 --> End
Error2 --> End
style Start fill:#aqua,stroke:#333
style End fill:#aqua,stroke:#333
style Error1 fill:#ff6,stroke:#333
style Error2 fill:#ff6,stroke:#333
```

**Diagram sources**
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)

**Section sources**
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)

### create-new-feature.sh 分析
该脚本自动化创建新功能分支和相关目录结构，是SDD流程的起点。

```mermaid
flowchart TD
Start([开始]) --> ParseArgs["解析参数"]
ParseArgs --> ValidateInput{"功能描述为空?"}
ValidateInput --> |是| Error["报错: 提供功能描述"]
ValidateInput --> |否| GetRepoRoot["获取仓库根目录"]
GetRepoRoot --> EnsureSpecs["确保 specs 目录"]
EnsureSpecs --> FindHighest["查找最高功能编号"]
FindHighest --> CalcNext["计算下一个编号"]
CalcNext --> FormatBranch["格式化分支名称"]
FormatBranch --> CreateBranch["创建Git分支"]
CreateBranch --> CreateDir["创建功能目录"]
CreateDir --> CopyTemplate["复制 spec 模板"]
CopyTemplate --> OutputResult["输出结果"]
Error --> End([结束])
OutputResult --> End
style Start fill:#aqua,stroke:#333
style End fill:#aqua,stroke:#333
style Error fill:#ff6,stroke:#333
```

**Diagram sources**
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh#L1-L58)

**Section sources**
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh#L1-L58)

### get-feature-paths.sh 分析
该脚本解析并输出当前功能的项目路径，为其他脚本提供路径信息。

```mermaid
flowchart TD
Start([开始]) --> SetE["set -e"]
SetE --> SourceCommon["加载 common.sh"]
SourceCommon --> GetPaths["执行 get_feature_paths"]
GetPaths --> CheckBranch["验证功能分支"]
CheckBranch --> OutputVars["输出路径变量"]
OutputVars --> End([结束])
style Start fill:#aqua,stroke:#333
style End fill:#aqua,stroke:#333
```

**Diagram sources**
- [get-feature-paths.sh](file://scripts/bash/get-feature-paths.sh#L1-L7)

**Section sources**
- [get-feature-paths.sh](file://scripts/bash/get-feature-paths.sh#L1-L7)

### setup-plan.sh 分析
该脚本协调SDD流程中的计划阶段，为实施准备环境。

```mermaid
flowchart TD
Start([开始]) --> ParseArgs["解析参数"]
ParseArgs --> SourceCommon["加载 common.sh"]
SourceCommon --> GetPaths["执行 get_feature_paths"]
GetPaths --> CheckBranch["验证功能分支"]
CheckBranch --> EnsureDir["确保功能目录"]
EnsureDir --> CopyTemplate["复制 plan 模板"]
CopyTemplate --> OutputResult["输出结果"]
OutputResult --> End([结束])
style Start fill:#aqua,stroke:#333
style End fill:#aqua,stroke:#333
```

**Diagram sources**
- [setup-plan.sh](file://scripts/bash/setup-plan.sh#L1-L17)

**Section sources**
- [setup-plan.sh](file://scripts/bash/setup-plan.sh#L1-L17)

### update-agent-context.sh 分析
该脚本动态更新AI代理的上下文文件，是人机协作的关键组件。

```mermaid
flowchart TD
Start([开始]) --> GetInfo["获取分支和计划信息"]
GetInfo --> CheckPlan{"plan.md 存在?"}
CheckPlan --> |否| Error["报错: 无 plan.md"]
CheckPlan --> |是| ExtractInfo["从 plan.md 提取技术栈"]
ExtractInfo --> UpdateFiles["更新代理文件"]
UpdateFiles --> CaseStmt["根据代理类型分支"]
CaseStmt --> UpdateClaude["更新Claude文件"]
CaseStmt --> UpdateGemini["更新Gemini文件"]
CaseStmt --> UpdateCopilot["更新Copilot文件"]
CaseStmt --> UpdateCursor["更新Cursor文件"]
CaseStmt --> UpdateQwen["更新Qwen文件"]
CaseStmt --> UpdateOpencode["更新opencode文件"]
CaseStmt --> UpdateAll["更新所有存在文件"]
UpdateClaude --> Summary
UpdateGemini --> Summary
UpdateCopilot --> Summary
UpdateCursor --> Summary
UpdateQwen --> Summary
UpdateOpencode --> Summary
UpdateAll --> Summary
Summary --> OutputSummary["输出变更摘要"]
OutputSummary --> End([结束])
Error --> End
style Start fill:#aqua,stroke:#333
style End fill:#aqua,stroke:#333
style Error fill:#ff6,stroke:#333
```

**Diagram sources**
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)

**Section sources**
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)

### common.sh 分析
该脚本封装了共享函数和变量，是整个脚本系统的基础。

```mermaid
classDiagram
class common.sh {
+get_repo_root() string
+get_current_branch() string
+check_feature_branch(branch) bool
+get_feature_dir(repo_root, branch) string
+get_feature_paths() string
+check_file(path, label) void
+check_dir(path, label) void
}
class create-new-feature.sh {
+FEATURE_DESCRIPTION string
+FEATURE_NUM string
+BRANCH_NAME string
}
class setup-plan.sh {
+JSON_MODE bool
}
class check-task-prerequisites.sh {
+JSON_MODE bool
}
class update-agent-context.sh {
+AGENT_TYPE string
}
common.sh <|-- create-new-feature.sh : "source"
common.sh <|-- setup-plan.sh : "source"
common.sh <|-- check-task-prerequisites.sh : "source"
common.sh <|-- get-feature-paths.sh : "source"
common.sh <|-- update-agent-context.sh : "source"
style common.sh fill : #69f,stroke : #333
style create-new-feature.sh fill : #f96,stroke : #333
style setup-plan.sh fill : #f96,stroke : #333
style check-task-prerequisites.sh fill : #f96,stroke : #333
style get-feature-paths.sh fill : #f96,stroke : #333
style update-agent-context.sh fill : #f96,stroke : #333
```

**Diagram sources**
- [common.sh](file://scripts/bash/common.sh#L1-L37)

**Section sources**
- [common.sh](file://scripts/bash/common.sh#L1-L37)

## 依赖分析
脚本系统依赖Git进行版本控制，依赖文件系统进行目录和文件操作，并通过模板文件与文档系统集成。脚本间通过`common.sh`形成依赖关系，所有功能脚本都依赖于这个共享库。`update-agent-context.sh`还依赖Python 3进行复杂的文本处理。

```mermaid
graph TD
Git[Git] --> Common
FS[文件系统] --> Common
Templates[模板文件] --> CreateFeature
Templates --> SetupPlan
Templates --> UpdateContext
Python[Python 3] --> UpdateContext
Common --> CreateFeature
Common --> SetupPlan
Common --> CheckPrereq
Common --> GetPaths
Common --> UpdateContext
style Git fill:#6f9,stroke:#333
style FS fill:#6f9,stroke:#333
style Templates fill:#6f9,stroke:#333
style Python fill:#6f9,stroke:#333
style Common fill:#69f,stroke:#333
style CreateFeature fill:#f96,stroke:#333
style SetupPlan fill:#f96,stroke:#333
style CheckPrereq fill:#f96,stroke:#333
style GetPaths fill:#f96,stroke:#333
style UpdateContext fill:#f96,stroke:#333
```

**Diagram sources**
- [common.sh](file://scripts/bash/common.sh#L1-L37)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh#L1-L58)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh#L1-L17)
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)

**Section sources**
- [common.sh](file://scripts/bash/common.sh#L1-L37)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh#L1-L58)
- [setup-plan.sh](file://scripts/bash/setup-plan.sh#L1-L17)
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)

## 性能考虑
脚本系统性能主要受Git操作和文件I/O影响。`create-new-feature.sh`中的分支创建和`update-agent-context.sh`中的Python文本处理是潜在的性能瓶颈。脚本使用`set -e`确保错误立即终止，避免了无效操作的累积。对于大型项目，建议在SSD上运行以优化文件操作性能。

## 故障排除指南
常见问题包括分支命名不符合规范、缺少必要的模板文件、Git未正确安装等。`check-task-prerequisites.sh`提供了内置的依赖检查功能。当`update-agent-context.sh`失败时，应检查`plan.md`文件是否存在且格式正确。权限问题可能导致脚本无法创建目录或文件，应确保执行用户具有适当的文件系统权限。

**Section sources**
- [check-task-prerequisites.sh](file://scripts/bash/check-task-prerequisites.sh#L1-L15)
- [update-agent-context.sh](file://scripts/bash/update-agent-context.sh#L1-L66)

## 结论
Bash脚本系统为Spec-Driven Development提供了强大而灵活的自动化基础。通过模块化设计和清晰的职责划分，系统实现了从功能创建到AI集成的完整闭环。脚本的可组合性和JSON输出支持使其易于集成到更大的开发工作流中，为现代软件开发提供了一套高效的工具集。