<docs>
# /specify 命令详解

<cite>
**本文档引用的文件**   
- [spec-template.md](file://templates/spec-template.md) - *定义了规范文档的结构和AI生成规则*
- [__init__.py](file://src/specify_cli/__init__.py) - *CLI工具的核心实现，包含初始化逻辑*
- [specify.md](file://templates/commands/specify.md) - *命令的元数据和执行脚本定义*
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh) - *POSIX系统下的功能创建脚本*
- [create-new-feature.ps1](file://scripts/powershell/create-new-feature.ps1) - *PowerShell系统下的功能创建脚本*
</cite>

## 更新摘要
**已做更改**   
- 根据 `README.md` 的更新，补充了 `/specify` 命令在整体SDD工作流中的上下文和使用顺序。
- 在“引言”部分增加了与 `/constitution`, `/plan`, `/tasks`, `/implement` 等其他核心命令的关联说明。
- 更新了“完整执行流程示例”中的用户输入示例，使其更贴近 `README.md` 中提供的实际用例。
- 所有文件引用标题（如 `[spec-template.md]`）均已强制转换为中文。

## 目录
1. [引言](#引言)
2. [核心功能与工作流](#核心功能与工作流)
3. [交互式流程与输入验证](#交互式流程与输入验证)
4. [模板驱动的AI集成](#模板驱动的ai集成)
5. [完整执行流程示例](#完整执行流程示例)
6. [错误处理与常见问题](#错误处理与常见问题)
7. [结论](#结论)

## 引言

`/specify` 命令是 Spec-Driven Development (SDD) 工作流中的核心指令，它充当了从模糊的用户需求到精确、结构化功能规范文档的桥梁。该命令通过一个精心设计的交互式流程，引导用户将自然语言描述的功能需求，转化为符合 `spec-template.md` 严格结构的、可执行的规范文档。此过程不仅自动化了文档创建，更重要的是，它通过模板约束和AI集成，确保了生成内容的一致性、完整性和业务导向性，为后续的 `/plan` 和 `/tasks` 阶段提供了坚实可靠的输入基础。

在完整的SDD工作流中，`/specify` 命令通常在使用 `/constitution` 命令建立项目原则后被调用，用于定义具体功能。其输出的规范文档随后会作为 `/plan` 命令的输入，以生成技术实现计划。

**Section sources**
- [specify.md](file://templates/commands/specify.md)

## 核心功能与工作流

`/specify` 命令的核心功能是启动一个端到端的规范生成工作流。其工作流并非单一操作，而是一个由多个脚本和模板协同完成的复杂流程。

1.  **命令触发与参数解析**：当用户在支持的AI环境中（如Claude Code）输入 `/specify` 并附带功能描述时，该命令被触发。AI助手会解析用户提供的自然语言描述作为参数。
2.  **脚本执行与环境准备**：根据用户的操作系统，`/specify` 命令会调用相应的底层脚本——`create-new-feature.sh`（POSIX Shell）或 `create-new-feature.ps1`（PowerShell）。该脚本是工作流的“执行引擎”。
3.  **分支与目录创建**：脚本首先在Git仓库中创建一个新的功能分支。分支名称遵循 `[序号]-[功能描述关键词]` 的格式（例如 `001-user-login`），确保了分支的唯一性和可追溯性。同时，它会在 `specs/` 目录下创建一个与分支同名的文件夹，用于存放该功能的所有相关文档。
4.  **规范文档初始化**：脚本会将 `templates/spec-template.md` 文件复制到新创建的功能目录中，并重命名为 `spec.md`。这一步完成了规范文档的物理创建和结构初始化。
5.  **信息传递**：脚本以JSON格式输出关键信息，包括新创建的 `BRANCH_NAME` 和 `SPEC_FILE` 的绝对路径。这些信息被返回给AI助手，作为后续填充模板的上下文。

```mermaid
flowchart TD
A[用户输入 /specify <功能描述>] --> B[AI助手解析命令]
B --> C{调用对应脚本}
C --> D[create-new-feature.sh]
C --> E[create-new-feature.ps1]
D --> F[创建Git功能分支]
E --> F
F --> G[创建功能目录 specs/001-feature-name]
G --> H[复制 spec-template.md 到 spec.md]
H --> I[输出 BRANCH_NAME 和 SPEC_FILE 路径]
I --> J[AI助手获取输出]
J --> K[开始填充规范模板]
```

**Diagram sources**
- [specify.md](file://templates/commands/specify.md)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh)
- [create-new-feature.ps1](file://scripts/powershell/create-new-feature.ps1)

**Section sources**
- [specify.md](file://templates/commands/specify.md)
- [create-new-feature.sh](file://scripts/bash/create-new-feature.sh)
- [create-new-feature.ps1](file://scripts/powershell/create-new-feature.ps1)

## 交互式流程与输入验证

`/specify` 命令的交互式流程主要体现在其底层支持工具 `specify-cli` 的 `init` 命令中，这为整个SDD工作流奠定了基础。虽然 `/specify` 本身直接处理功能描述，但其依赖的环境是通过一个高度交互的初始化流程建立的。

1.  **选择AI助手**：在项目初始化阶段，`init` 命令会通过一个基于箭头键的交互式菜单，让用户从 `claude`, `gemini`, `copilot`, `cursor`, `qwen`, `opencode` 等选项中选择其偏好的AI助手。这确保了后续 `/specify` 等命令能与正确的AI工具链集成。
2.  **选择脚本类型**：同样，用户可以选择使用 `sh` (Bash) 还是 `ps` (PowerShell) 脚本，系统会根据用户的操作系统提供默认选项。
3.  **输入验证**：`init` 命令在执行前会进行严格的输入验证。例如，它不允许同时指定 `--here` 标志和项目名称，会检查目标目录是否已存在或是否为空，防止意外覆盖。对于 `/specify` 命令，其输入验证主要由底层脚本 `create-new-feature.sh` 和 `create-new-feature.ps1` 完成。它们会检查用户是否提供了功能描述，如果为空，则会输出用法提示并退出，确保了工作流的起点是有效的。

```mermaid
flowchart TD
A[运行 specify init] --> B[显示ASCII艺术横幅]
B --> C[验证参数]
C --> D{参数有效?}
D --> |否| E[输出错误并退出]
D --> |是| F[确定项目路径]
F --> G[检查Git]
G --> H[交互式选择AI助手]
H --> I[交互式选择脚本类型]
I --> J[下载并解压模板]
J --> K[设置脚本权限]
K --> L[初始化Git仓库]
L --> M[显示下一步骤]
```

**Diagram sources**
- [__init__.py](file://src/specify_cli/__init__.py)

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py)

## 模板驱动的AI集成

`/specify` 命令与AI助手的集成是其最强大的特性，而 `spec-template.md` 是实现这种集成的关键约束机制。

1.  **模板作为AI的“指南针”**：`spec-template.md` 不仅仅是一个文档框架，它是一个包含明确指令的“智能模板”。模板中的“Execution Flow (main)”部分定义了AI在生成规范时必须遵循的逻辑步骤，例如“提取关键概念”、“标记不明确之处”、“生成可测试的功能需求”等。这确保了AI的输出不是随意的，而是有逻辑、有结构的。
2.  **强制业务导向**：模板通过“Quick Guidelines”明确要求AI“关注用户需要什么和为什么（WHAT和WHY）”，并禁止提及“如何实现（HOW）”，如技术栈、API或代码结构。这迫使AI从商业和用户价值的角度思考，生成的文档面向利益相关者而非开发者。
3.  **处理不确定性**：模板要求AI在遇到任何不明确的地方时，使用 `[NEEDS CLARIFICATION: ...]` 标记。例如，如果用户描述“登录系统”但未指定认证方式，AI必须生成 `System MUST authenticate users via [NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]`。这将模糊性显式化，而不是让AI自行猜测，保证了规范的完整性。
4.  **AI的执行过程**：当AI助手收到 `/specify` 命令的输出（即 `SPEC_FILE` 路径）后，它会：
    *   读取 `spec-template.md` 以理解结构和规则。
    *   将用户的功能描述作为输入。
    *   严格按照模板中的执行流程，将用户描述的内容填充到“User Scenarios & Testing”、“Requirements”等相应章节。
    *   在填充过程中，主动识别并标记所有模糊点。
    *   最终生成一个结构完整、内容清晰、且明确标出所有待澄清项的 `spec.md` 文件。

```mermaid
classDiagram
class AITool {
+string userInput
+string templatePath
+string specFilePath
+loadTemplate() SpecTemplate
+parseUserInput() KeyConcepts
+fillSection(sectionName, content) void
+markAmbiguity(question) void
+generateSpec() SpecDocument
}
class SpecTemplate {
+string executionFlow
+string quickGuidelines
+string sectionRequirements
+string userScenariosSection
+string requirementsSection
+string reviewChecklist
}
class SpecDocument {
+string featureName
+string featureBranch
+string createdDate
+string status
+UserScenarios userScenarios
+Requirements requirements
+ReviewChecklist reviewChecklist
}
AITool --> SpecTemplate : "uses to understand structure"
AITool --> SpecDocument : "creates and populates"
SpecDocument --> UserScenarios : "contains"
SpecDocument --> Requirements : "contains"
SpecDocument --> ReviewChecklist : "contains"
class UserScenarios {
+string primaryUserStory
+string[] acceptanceScenarios
+string[] edgeCases
}
class Requirements {
+string[] functionalRequirements
+string[] keyEntities
}
class ReviewChecklist {
+CheckItem[] contentQuality
+CheckItem[] requirementCompleteness
}
class CheckItem {
+string description
+bool status
}
```

**Diagram sources**
- [spec-template.md](file://templates/spec-template.md)

**Section sources**
- [spec-template.md](file://templates/spec-template.md)

## 完整执行流程示例

