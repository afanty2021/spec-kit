# /clarify 与 /analyze 命令详解

<cite>
**本文档引用文件**  
- [clarify.md](file://templates/commands/clarify.md)
- [analyze.md](file://templates/commands/analyze.md)
- [constitution.md](file://memory/constitution.md)
- [spec-template.md](file://templates/spec-template.md)
- [plan-template.md](file://templates/plan-template.md)
- [tasks-template.md](file://templates/tasks-template.md)
- [CHANGELOG.md](file://CHANGELOG.md)
- [README.md](file://README.md)
- [spec-driven.md](file://spec-driven.md)
</cite>

## 目录
1. [引言](#引言)
2. [/clarify 命令解析](#clarify-命令解析)
3. [/analyze 命令解析](#analyze-命令解析)
4. [反馈闭环与质量保障机制](#反馈闭环与质量保障机制)
5. [实际交互示例](#实际交互示例)
6. [总结](#总结)

## 引言
在基于规范驱动的开发（Spec-Driven Development, SDD）流程中，`/clarify` 和 `/analyze` 是两个关键的辅助性命令，它们共同构建了一个闭环的反馈机制，确保 AI 生成内容的准确性、一致性和可维护性。这两个命令分别在开发流程的不同阶段介入，通过结构化的问题引导和跨文档一致性分析，有效防止模糊需求、技术债务和实现偏差的累积。

`/clarify` 命令在规划（`/plan`）之前运行，旨在识别并解决功能规范（spec）中的模糊点。而 `/analyze` 命令则在任务生成（`/tasks`）之后、实现（`/implement`）之前运行，对 `spec.md`、`plan.md` 和 `tasks.md` 三份核心文档进行非破坏性的一致性校验。本文将深入解析这两个命令的工作机制、执行流程及其在提升开发质量中的核心作用。

**Section sources**
- [README.md](file://README.md#L212-L215)
- [spec-driven.md](file://spec-driven.md#L20-L26)

## /clarify 命令解析

`/clarify` 命令的核心目标是检测并减少当前功能规范中的歧义或缺失的决策点，并将澄清后的答案直接记录回规范文件中。它是一个主动的、交互式的质量保障工具，强制在进入技术设计和任务分解阶段前解决关键的不确定性。

### 执行流程与工作机制
`/clarify` 的执行流程遵循一个严谨的结构化步骤：

1.  **环境检查与文件加载**：命令首先通过脚本（`check-prerequisites.sh`）获取当前功能目录（`FEATURE_DIR`）和规范文件路径（`FEATURE_SPEC`），然后加载该规范文件。
2.  **结构化模糊性扫描**：命令会根据一个预定义的分类法对规范文件进行系统性扫描。该分类法涵盖功能范围、领域数据模型、用户交互、非功能性需求、外部依赖、边缘情况、约束条件、术语一致性等多个维度。对于每个维度，系统会标记其状态为“清晰”、“部分”或“缺失”。
3.  **生成优先级问题队列**：基于扫描结果，系统内部生成一个最多包含5个问题的优先级队列。这些问题的选择遵循严格标准：
    *   **数量限制**：单次会话最多提出5个问题。
    *   **可回答性**：问题必须能通过简短的多选题或不超过5个词的短语来回答。
    *   **影响优先**：优先选择那些答案会显著影响架构、数据建模、任务分解或测试设计的问题。
    *   **避免低价值问题**：排除已回答、风格偏好或计划执行细节类问题。
4.  **交互式提问循环**：命令以交互方式逐个提出问题。对于多选题，会以Markdown表格形式呈现选项；对于开放式问题，则明确要求“短答案（<=5词）”。用户回答后，系统会验证答案的有效性，并在确认后将其记录在内存中。
5.  **增量式集成与更新**：在每次接受一个答案后，系统会立即更新规范文件：
    *   在规范中创建或定位 `## Clarifications` 章节。
    *   为当天的会话创建 `### Session YYYY-MM-DD` 子章节。
    *   将问答对（`- Q: <问题> → A: <答案>`）追加到会话中。
    *   根据答案内容，将澄清信息**增量式地**应用到规范的相应部分（如功能需求、数据模型、非功能性要求等），并立即保存文件，确保上下文不丢失。
6.  **最终报告**：会话结束后，命令会生成一份报告，包括提问数量、更新的文件路径、涉及的章节、各分类的覆盖状态摘要（已解决、已推迟、清晰、未解决），并给出下一步建议。

### 关键行为规则
*   `/clarify` 必须在 `/plan` 之前运行，除非用户明确声明跳过（如探索性原型）。
*   始终保持只读状态，不修改任何文件，仅输出分析报告。
*   项目宪法（`constitution.md`）是分析中的“非协商”权威，任何与宪法的冲突都被视为**严重**问题。
*   如果规范文件中没有发现关键歧义，会直接报告“未检测到值得正式澄清的关键歧义”。

**Section sources**
- [clarify.md](file://templates/commands/clarify.md#L1-L161)
- [spec-template.md](file://templates/spec-template.md#L4-L116)

## /analyze 命令解析

`/analyze` 命令是一个非破坏性的、跨文档的一致性与质量分析工具。它的目标是在实施前，识别出规范（`spec.md`）、计划（`plan.md`）和任务（`tasks.md`）三者之间的不一致、重复、模糊和未充分说明的项目。

### 执行流程与检测机制
`/analyze` 的执行流程如下：

1.  **环境检查与文件加载**：运行脚本获取 `FEATURE_DIR`，并确认 `spec.md`、`plan.md` 和 `tasks.md` 三个文件都存在。如果缺少任一文件，命令将中止。
2.  **加载与解析文档**：分别解析三份文档的关键内容：
    *   `spec.md`：功能需求、非功能需求、用户故事、边缘情况。
    *   `plan.md`：技术栈选择、数据模型、阶段划分、技术约束。
    *   `tasks.md`：任务ID、描述、阶段分组、并行标记、引用的文件路径。
    *   同时加载项目宪法（`/memory/constitution.md`）用于原则校验。
3.  **构建内部语义模型**：系统会构建一个需求清单（为每个需求生成稳定的关键字）、用户故事清单、任务覆盖映射（将任务映射到需求或故事）以及宪法规则集。
4.  **多轮检测分析**：系统执行多轮检测，识别以下问题：
    *   **重复检测**：识别内容相似的需求。
    *   **模糊性检测**：标记缺乏可衡量标准的模糊形容词（如“快速”、“安全”）和未解决的占位符（如“TODO”）。
    *   **未充分说明**：识别缺少对象或可衡量结果的需求，或缺少验收标准的用户故事。
    *   **宪法对齐**：检查是否有任何需求或计划元素违反了宪法中的“必须”（MUST）原则。
    *   **覆盖缺口**：识别没有关联任务的需求，或没有映射到任何需求/故事的任务。
    *   **不一致性**：检查术语漂移（同一概念在不同文件中命名不同）、数据实体在计划和规范中的不一致、任务顺序的矛盾等。
5.  **严重性分级与报告生成**：根据预设的启发式方法对问题进行严重性分级（严重、高、中、低），并生成一份结构化的Markdown报告。报告包含问题表格、覆盖摘要、宪法对齐问题、未映射任务和各项指标（总需求、总任务、覆盖率等）。
6.  **建议与交互**：报告末尾会给出明确的后续行动建议（如“在`/implement`前解决严重问题”），并询问用户是否希望为前N个问题提供具体的修复编辑建议。

### 关键行为规则
*   `/analyze` 是严格的只读命令，不会修改任何文件。
*   宪法冲突是最高优先级的**严重**问题，必须通过修改规范、计划或任务来解决，而不是弱化原则。
*   分析结果必须是确定性的，确保在输入不变的情况下，重复运行能产生一致的输出。
*   如果没有发现问题，会生成一份包含覆盖率统计的成功报告。

**Section sources**
- [analyze.md](file://templates/commands/analyze.md#L1-L104)
- [plan-template.md](file://templates/plan-template.md#L1-L225)
- [tasks-template.md](file://templates/tasks-template.md#L1-L126)

## 反馈闭环与质量保障机制

`/clarify` 和 `/analyze` 共同构成了一个强大的反馈闭环，为AI生成内容的准确性和可靠性提供了双重保障。

### 防止技术债务累积
技术债务往往源于模糊的需求和不一致的实现。`/clarify` 通过在早期强制澄清，将模糊的“TODO”和“待定”决策转化为明确的、可测试的规范条目，从源头上减少了因误解而导致的返工。`/analyze` 则在后期充当“守门员”，通过跨文档分析，确保从需求到任务的整个链条是完整且一致的。它能发现那些在单一文档中看似合理，但在整体上下文中却存在冲突或遗漏的问题，从而防止这些“债务”被带入实现阶段。

### 提升AI生成内容的可靠性
这两个命令通过结构化和自动化的方式，弥补了AI在理解上下文和保持长期一致性方面的不足。
*   `/clarify` 的结构化提问流程确保了AI不会对关键决策进行“猜测”，而是主动寻求人类的明确输入。
*   `/analyze` 的一致性检查充当了AI的“同行评审”，验证其生成的计划和任务是否忠实于原始规范，并符合项目的核心原则（宪法）。

这种机制将AI从一个“黑盒”生成器转变为一个可审计、可验证的协作伙伴，极大地提升了其输出的可信度。

```mermaid
flowchart TD
A[功能描述] --> B[/specify]
B --> C[spec.md]
C --> D[/clarify]
D --> E{澄清完成?}
E --> |是| F[/plan]
E --> |否| D
F --> G[plan.md]
G --> H[/tasks]
H --> I[tasks.md]
I --> J[/analyze]
J --> K{分析通过?}
K --> |是| L[/implement]
K --> |否| M[手动修复或重新生成]
M --> H
style D fill:#f9f,stroke:#333
style J fill:#f9f,stroke:#333
click D "templates/commands/clarify.md" "查看 /clarify 模板"
click J "templates/commands/analyze.md" "查看 /analyze 模板"
```

**Diagram sources**
- [clarify.md](file://templates/commands/clarify.md)
- [analyze.md](file://templates/commands/analyze.md)

**Section sources**
- [README.md](file://README.md#L409-L416)
- [CHANGELOG.md](file://CHANGELOG.md#L19-L20)

## 实际交互示例

以下是一个使用 `/clarify` 和 `/analyze` 发现潜在冲突并推动文档迭代的示例场景。

### 场景：用户要求“创建一个快速的文件上传功能”

1.  **初始规范生成**：`/specify` 命令生成 `spec.md`，其中包含“系统必须允许用户上传文件”和“系统必须快速处理上传”等需求。
2.  **运行 `/clarify`**：
    *   `/clarify` 检测到“快速”是一个模糊的非功能性需求。
    *   它提出问题：“性能目标：请为文件上传定义具体的性能目标。”
    *   选项：
        *   A: 平均延迟 < 500ms
        *   B: 支持 1000 次请求/秒
        *   C: 用户感知为即时
        *   短答案：提供其他目标（<=5词）
    *   用户选择 A。
    *   `/clarify` 将答案记录在 `## Clarifications` 部分，并将“快速”更新为“平均延迟 < 500ms”。
3.  **生成计划与任务**：用户依次运行 `/plan` 和 `/tasks`，生成 `plan.md` 和 `tasks.md`。
4.  **运行 `/analyze`**：
    *   `/analyze` 加载所有三个文件和 `constitution.md`。
    *   检测到 `constitution.md` 中有一条“必须”原则：“所有API必须提供OpenAPI规范”。
    *   分析 `plan.md`，发现其中提到了一个 `/api/upload` 端点，但 `plan.md` 中没有提及生成OpenAPI规范的任务。
    *   分析 `tasks.md`，确认没有与“生成API文档”相关的任务。
    *   `/analyze` 报告一个**严重**问题：“需求 `api-upload-endpoint` 违反宪法原则‘所有API必须提供OpenAPI规范’。计划中提到了端点，但任务列表中缺少生成OpenAPI规范的任务。”
5.  **推动文档迭代**：用户根据 `/analyze` 的报告，手动编辑 `tasks.md`，添加一个新任务：“T024 [P] 为 /api/upload 生成 OpenAPI 规范文档”。然后可以重新运行 `/analyze` 进行验证，直到所有严重问题都得到解决。

这个例子清晰地展示了 `/clarify` 如何解决需求层面的模糊性，而 `/analyze` 如何发现跨文档的、与项目核心原则相违背的结构性缺陷，两者协同工作，共同推动了高质量、一致性的开发文档的形成。

**Section sources**
- [spec-template.md](file://templates/spec-template.md#L70-L75)
- [analyze.md](file://templates/commands/analyze.md#L17-L18)
- [plan-template.md](file://templates/plan-template.md#L100-L105)

## 总结
`/clarify` 和 `/analyze` 是规范驱动开发流程中不可或缺的两个质量保障支柱。`/clarify` 通过结构化的、交互式的提问，在开发早期主动识别并消除需求规范中的歧义，确保了“做什么”的清晰性。`/analyze` 则在开发后期，通过非破坏性的、跨文档的一致性分析，验证了“如何做”的计划与任务是否与规范和项目宪法保持一致，确保了实现路径的正确性。

这两个命令共同构建了一个强大的反馈闭环，将AI生成内容的不确定性降至最低，有效防止了技术债务的累积，显著提升了开发过程的效率和最终产出的可靠性。通过强制执行清晰的规范和严格的对齐检查，它们确保了团队在正确的方向上协同工作，是实现高质量、可预测的软件交付的关键工具。

**Section sources**
- [spec-driven.md](file://spec-driven.md#L20-L26)
- [README.md](file://README.md#L416-L421)