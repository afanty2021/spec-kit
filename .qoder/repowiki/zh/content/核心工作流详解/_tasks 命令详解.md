# /tasks 命令详解

<cite>
**本文档引用文件**  
- [tasks.md](file://templates/commands/tasks.md)
- [tasks-template.md](file://templates/tasks-template.md)
- [check-prerequisites.sh](file://scripts/bash/check-prerequisites.sh)
- [spec-driven.md](file://spec-driven.md)
</cite>

## 目录
1. [简介](#简介)
2. [任务分解机制](#任务分解机制)
3. [环境依赖检查](#环境依赖检查)
4. [任务条目结构](#任务条目结构)
5. [任务生成与验证示例](#任务生成与验证示例)
6. [模板定制与工具集成](#模板定制与工具集成)
7. [结论](#结论)

## 简介
`/tasks` 命令是规范驱动开发（SDD）工作流中的关键环节，负责将高层次的技术实现计划转化为一系列原子化、可追踪的开发任务。该命令基于 `templates/commands/tasks.md` 的指令设计，通过分析 `plan.md` 及相关设计文档，生成符合 `tasks-template.md` 格式的任务清单。此过程确保了从需求到实现的无缝转换，同时支持与 Jira、Trello 等项目管理工具的语义对齐。

**Section sources**
- [spec-driven.md](file://spec-driven.md#L95-L118)

## 任务分解机制
`/tasks` 命令的核心功能是将技术实现计划拆解为具体的开发任务。这一过程遵循严格的规则和顺序，以确保任务的完整性和可执行性。

1. **输入分析**：读取 `plan.md`（必需），并根据存在情况读取 `data-model.md`、`contracts/` 和 `research.md`。
2. **任务派生**：将合同、实体和场景转换为具体任务。
3. **并行化处理**：标记独立任务 `[P]`，并列出安全的并行组。
4. **输出生成**：在功能目录中写入 `tasks.md`，供任务代理执行。

任务生成规则包括：
- 每个合同文件 → 合同测试任务 `[P]`
- 每个实体 → 模型创建任务 `[P]`
- 每个端点 → 实现任务（如果共享文件则不并行）
- 每个用户故事 → 集成测试 `[P]`
- 不同文件 = 可以并行 `[P]`
- 相同文件 = 顺序执行（无 `[P]`）

任务按依赖关系排序：
- 设置优先于一切
- 测试先于实现（TDD）
- 模型先于服务
- 服务先于端点
- 核心先于集成
- 所有任务先于润色

**Section sources**
- [tasks.md](file://templates/commands/tasks.md#L13-L60)
- [tasks-template.md](file://templates/tasks-template.md#L0-L42)

## 环境依赖检查
`/tasks` 命令使用 `check-prerequisites.sh` 脚本来检查环境依赖。该脚本提供统一的先决条件检查，确保开发环境满足要求。

### 脚本选项
- `--json`：以 JSON 格式输出
- `--require-tasks`：要求 `tasks.md` 存在（用于实现阶段）
- `--include-tasks`：将 `tasks.md` 包含在 `AVAILABLE_DOCS` 列表中
- `--paths-only`：仅输出路径变量（无验证）
- `--help, -h`：显示帮助信息

### 输出格式
- **JSON 模式**：`{"FEATURE_DIR":"...", "AVAILABLE_DOCS":["..."]}`
- **文本模式**：`FEATURE_DIR:... \n AVAILABLE_DOCS: \n ✓/✗ file.md`
- **路径仅模式**：`REPO_ROOT: ... \n BRANCH: ... \n FEATURE_DIR: ... etc.`

脚本首先解析命令行参数，然后获取特征路径并验证分支。如果启用了 `--paths-only` 模式，则仅输出路径并退出。接着，脚本验证所需的目录和文件是否存在，构建可用文档列表，并根据模式输出结果。

**Section sources**
- [check-prerequisites.sh](file://scripts/bash/check-prerequisites.sh#L0-L165)

## 任务条目结构
每个任务条目都包含详细的结构信息，以便于项目管理和跟踪。

### 任务格式
- `[ID] [P?] Description`
- **[P]**：可以并行运行（不同文件，无依赖）
- 在描述中包含确切的文件路径

### 路径约定
- **单个项目**：`src/`, `tests/` 在仓库根目录
- **Web 应用**：`backend/src/`, `frontend/src/`
- **移动应用**：`api/src/`, `ios/src/` 或 `android/src/`

### 任务阶段
- **Phase 3.1: Setup**
  - [ ] T001 创建项目结构
  - [ ] T002 初始化 [语言] 项目
  - [ ] T003 [P] 配置 linting 和格式化工具
- **Phase 3.2: Tests First (TDD) ⚠️ 必须在 3.3 之前完成**
  - [ ] T004 [P] 合同测试 POST /api/users
  - [ ] T005 [P] 合同测试 GET /api/users/{id}
  - [ ] T006 [P] 集成测试用户注册
  - [ ] T007 [P] 集成测试认证流程
- **Phase 3.3: Core Implementation (ONLY after tests are failing)**
  - [ ] T008 [P] 用户模型
  - [ ] T009 [P] UserService CRUD
  - [ ] T010 [P] CLI --create-user
  - [ ] T011 POST /api/users 端点
  - [ ] T012 GET /api/users/{id} 端点
  - [ ] T013 输入验证
  - [ ] T014 错误处理和日志记录
- **Phase 3.4: Integration**
  - [ ] T015 连接 UserService 到 DB
  - [ ] T016 认证中间件
  - [ ] T017 请求/响应日志
  - [ ] T018 CORS 和安全头
- **Phase 3.5: Polish**
  - [ ] T019 [P] 单元测试验证
  - [ ] T020 性能测试 (<200ms)
  - [ ] T021 [P] 更新文档/api.md
  - [ ] T022 去除重复
  - [ ] T023 运行 manual-testing.md

### 依赖关系
- 测试 (T004-T007) 在实现 (T008-T014) 之前
- T008 阻塞 T009, T015
- T016 阻塞 T018
- 实现先于润色 (T019-T023)

### 并行示例
```
# 同时启动 T004-T007：
Task: "Contract test POST /api/users in tests/contract/test_users_post.py"
Task: "Contract test GET /api/users/{id} in tests/contract/test_users_get.py"
Task: "Integration test registration in tests/integration/test_registration.py"
Task: "Integration test auth in tests/integration/test_auth.py"
```

**Section sources**
- [tasks-template.md](file://templates/tasks-template.md#L43-L126)

## 任务生成与验证示例
以下是一个实际的任务生成与验证示例，展示了如何从一个简单的聊天功能需求生成任务清单。

### 示例：构建聊天功能
1. **创建功能规范**：
   ```bash
   /specify 实时聊天系统，具有消息历史和用户在线状态
   ```
   - 自动创建分支 "003-chat-system"
   - 生成 `specs/003-chat-system/spec.md`
   - 填充结构化需求

2. **生成实施计划**：
   ```bash
   /plan WebSocket 用于实时消息传递，PostgreSQL 用于历史记录，Redis 用于在线状态
   ```

3. **生成可执行任务**：
   ```bash
   /tasks
   ```
   - 生成 `specs/003-chat-system/plan.md`
   - 生成 `specs/003-chat-system/research.md`（WebSocket 库比较）
   - 生成 `specs/003-chat-system/data-model.md`（Message 和 User 模式）
   - 生成 `specs/003-chat-system/contracts/`（WebSocket 事件，REST 端点）
   - 生成 `specs/003-chat-system/quickstart.md`（关键验证场景）
   - 生成 `specs/003-chat-system/tasks.md`（从计划派生的任务列表）

### 验证清单
- [ ] 所有合同都有相应的测试
- [ ] 所有实体都有模型任务
- [ ] 所有测试都在实现之前
- [ ] 并行任务真正独立
- [ ] 每个任务指定确切的文件路径
- [ ] 没有任务修改另一个 `[P]` 任务的同一文件

**Section sources**
- [spec-driven.md](file://spec-driven.md#L119-L144)
- [tasks-template.md](file://templates/tasks-template.md#L118-L126)

## 模板定制与工具集成
为了实现与 Jira、Trello 等项目管理工具的语义对齐，可以通过模板定制来调整任务生成过程。

### 模板定制
- **路径约定**：根据项目结构调整路径约定。
- **任务阶段**：根据团队的工作流调整任务阶段。
- **依赖关系**：根据项目的依赖关系调整任务依赖。
- **并行示例**：根据团队的并行能力调整并行示例。

### 工具集成
- **Jira**：将任务 ID 映射到 Jira 问题 ID，使用 Jira API 创建和更新问题。
- **Trello**：将任务 ID 映射到 Trello 卡片 ID，使用 Trello API 创建和更新卡片。
- **GitHub Issues**：将任务 ID 映射到 GitHub 问题 ID，使用 GitHub API 创建和更新问题。

通过这些定制和集成，`/tasks` 命令可以更好地适应不同的项目管理和开发环境，提高任务的可追踪性和团队协作效率。

**Section sources**
- [tasks.md](file://templates/commands/tasks.md#L62-L65)

## 结论
`/tasks` 命令通过将高层次的技术实现计划拆解为一系列原子化、可追踪的开发任务，极大地提高了开发效率和质量。结合 `check-prerequisites.sh` 脚本的环境依赖检查，以及 `tasks-template.md` 的标准化任务格式，该命令确保了任务的完整性和可执行性。通过模板定制和工具集成，`/tasks` 命令可以灵活适应不同的项目管理和开发环境，实现与 Jira、Trello 等工具的语义对齐，从而提升团队的协作效率和项目管理能力。