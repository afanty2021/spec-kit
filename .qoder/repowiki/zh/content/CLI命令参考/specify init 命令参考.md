# specify init 命令参考

<cite>
**本文档中引用的文件**  
- [__init__.py](file://src/specify_cli/__init__.py)
- [README.md](file://README.md)
</cite>

## 目录
1. [简介](#简介)
2. [参数与选项](#参数与选项)
3. [内部执行流程](#内部执行流程)
4. [关键函数解析](#关键函数解析)
5. [命令示例](#命令示例)
6. [用户交互与终端输出](#用户交互与终端输出)
7. [初始化完成后的引导信息](#初始化完成后的引导信息)

## 简介
`specify init` 命令是 `spec-kit` 项目的核心初始化入口，用于从最新的 GitHub 模板创建一个新的 Specify 项目。该命令通过下载并解压模板、设置脚本权限、初始化 Git 仓库等一系列操作，为用户提供一个结构完整、即开即用的开发环境。此命令支持多种 AI 代理和脚本类型，允许用户根据具体需求进行灵活配置。

## 参数与选项
`specify init` 命令支持以下参数和选项，用于定制化项目初始化过程。

### 项目名称 (project_name)
- **类型**: 位置参数
- **描述**: 指定新项目的目录名称。如果使用 `.` 或 `--here` 选项，则可省略。
- **默认行为**: 无
- **合法取值**: 任何有效的目录名称。

### AI 代理 (--ai)
- **类型**: 选项
- **描述**: 指定要使用的 AI 代理。
- **默认行为**: 如果未指定，将通过交互式菜单让用户选择。
- **合法取值**: `claude`, `gemini`, `copilot`, `cursor`, `qwen`, `opencode`, `codex`, `windsurf`, `kilocode`, `auggie`, `roo`。

### 脚本类型 (--script)
- **类型**: 选项
- **描述**: 选择要使用的脚本类型。
- **默认行为**: 在非 Windows 系统上默认为 `sh`，在 Windows 系统上默认为 `ps`。
- **合法取值**: `sh` (POSIX Shell), `ps` (PowerShell)。

### 在当前目录初始化 (--here)
- **类型**: 标志
- **描述**: 在当前目录而非创建新目录来初始化项目。
- **默认行为**: `False`
- **合法取值**: 无（作为标志使用）。

### 跳过 Git 初始化 (--no-git)
- **类型**: 标志
- **描述**: 跳过 Git 仓库的初始化。
- **默认行为**: `False`
- **合法取值**: 无（作为标志使用）。

### 强制覆盖 (--force)
- **类型**: 标志
- **描述**: 当使用 `--here` 且当前目录非空时，跳过确认步骤，强制合并/覆盖文件。
- **默认行为**: `False`
- **合法取值**: 无（作为标志使用）。

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py#L749-L765)

## 内部执行流程
`specify init` 命令的内部执行流程是一个完整的生命周期，涵盖了从参数校验到最终引导的各个阶段。

### 1. 参数校验与项目路径确定
命令首先校验参数的有效性。如果同时指定了 `project_name` 和 `--here`，则会报错。如果使用 `--here` 或 `.`，则项目路径为当前工作目录；否则，项目路径为指定的 `project_name`。如果目标目录已存在，除非使用 `--force`，否则会提示用户确认。

### 2. AI 代理与脚本类型选择
如果通过 `--ai` 指定了 AI 代理，则进行有效性检查；否则，调用 `select_with_arrows` 函数启动交互式选择菜单。脚本类型的选择逻辑类似，会根据操作系统自动选择默认值，并在终端交互模式下提供选择菜单。

### 3. 下载与解压模板
调用 `download_and_extract_template` 函数，该函数会：
- 通过 GitHub API 获取最新发布的模板信息。
- 根据选定的 AI 代理和脚本类型，找到匹配的 ZIP 文件。
- 下载该 ZIP 文件。
- 将其解压到目标项目路径。如果是在当前目录初始化，会先解压到临时目录，然后将内容合并到当前目录。

### 4. 确保脚本可执行权限
调用 `ensure_executable_scripts` 函数，递归地为 `.specify/scripts` 目录下的所有 `.sh` 脚本设置可执行权限（在 Windows 上此操作无效）。

### 5. 初始化 Git 仓库
如果未指定 `--no-git`，则尝试初始化 Git 仓库。如果目标路径已是一个 Git 仓库，则跳过此步骤。

### 6. 显示“下一步”引导信息
初始化完成后，向用户显示一个包含后续步骤的引导面板。

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py#L761-L1099)

## 关键函数解析
`specify init` 命令的实现依赖于 `src/specify_cli/__init__.py` 文件中的几个关键函数。

### download_and_extract_template
此函数负责下载并解压模板。它接收项目路径、AI 代理、脚本类型等参数，使用 `httpx` 客户端从 GitHub 下载对应的 ZIP 文件，并使用 `zipfile` 模块进行解压。该函数能够处理 GitHub 风格的 ZIP 文件（即包含一个根目录），并将其“展平”到目标路径。在 `--here` 模式下，它会先将文件解压到临时目录，然后合并到当前目录，处理文件和目录的覆盖。

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py#L547-L702)

### ensure_executable_scripts
此函数确保所有 POSIX shell 脚本具有可执行权限。它遍历 `.specify/scripts` 目录下的所有 `.sh` 文件，检查其是否为 shebang 脚本（以 `#!` 开头），然后使用 `os.chmod` 设置相应的执行权限位。该函数在 Windows 上会静默跳过。

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py#L705-L747)

## 命令示例
以下是 `specify init` 命令的实用示例。

### 在当前目录初始化 Claude 项目
```bash
specify init . --ai claude
```
此命令将在当前目录下初始化一个使用 Claude 作为 AI 代理的项目。

### 为 Codex 项目设置环境变量
```bash
specify init my-codex-project --ai codex
```
初始化完成后，CLI 会提示用户设置 `CODEX_HOME` 环境变量，例如在 Unix 系统上执行 `export CODEX_HOME=/path/to/my-codex-project/.codex`。

### 使用 PowerShell 脚本初始化
```bash
specify init my-project --ai copilot --script ps
```
此命令将创建一个使用 GitHub Copilot 并包含 PowerShell 脚本的项目。

**Section sources**
- [README.md](file://README.md#L163-L203)

## 用户交互与终端输出
`specify init` 命令在执行过程中会与用户进行交互，并在终端输出详细信息。

### 交互式选择菜单
当未指定 `--ai` 或 `--script` 时，命令会显示一个使用箭头键导航的菜单。用户可以使用 ↑/↓ 键选择选项，按 Enter 键确认，按 Esc 键取消。

### 终端输出
命令会输出一个包含项目设置信息的面板，包括项目名称、工作路径和目标路径。随后，它会显示一个树状的进度跟踪器，实时更新每个步骤的状态（如“获取最新发布”、“下载模板”、“提取模板”等）。成功后，会输出绿色的“Project ready.”消息。

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py#L849-L864)

## 初始化完成后的引导信息
初始化完成后，CLI 会显示一个“下一步”面板，引导用户进入后续的斜杠命令工作流。

### 标准步骤
1. 进入项目目录：`cd <project_name>`（如果使用 `--here`，则此步省略）。
2. 启动 AI 代理并使用斜杠命令：
   - `/constitution`：建立项目原则。
   - `/specify`：创建基线规范。
   - `/plan`：创建实现计划。
   - `/tasks`：生成可执行任务。
   - `/implement`：执行实现。

### 可选增强命令
- `/clarify`：在规划前对模糊区域进行结构化提问。
- `/analyze`：在实现前进行跨工件一致性分析。

### Codex 特定提示
如果选择了 Codex 代理，会特别提示用户设置 `CODEX_HOME` 环境变量，并指出 Codex 目前不支持带参数的自定义提示。

**Section sources**
- [__init__.py](file://src/specify_cli/__init__.py#L1076-L1099)