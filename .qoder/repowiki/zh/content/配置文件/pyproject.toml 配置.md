# pyproject.toml 配置

<cite>
**本文档中引用的文件**  
- [pyproject.toml](file://pyproject.toml)
- [src/specify_cli/__init__.py](file://src/specify_cli/__init__.py)
</cite>

## 目录
1. [简介](#简介)
2. [项目元数据配置](#项目元数据配置)
3. [命令行脚本入口映射](#命令行脚本入口映射)
4. [构建系统配置](#构建系统配置)
5. [依赖管理与运行环境](#依赖管理与运行环境)
6. [可重复构建与跨平台兼容性](#可重复构建与跨平台兼容性)
7. [实际修改示例](#实际修改示例)
8. [结论](#结论)

## 简介
`pyproject.toml` 是 Spec-Kit 项目的核心配置文件，遵循 PEP 518 和 PEP 621 标准，定义了项目的元数据、依赖关系、构建系统和命令行接口。该文件使得 Specify CLI 工具能够被正确打包、安装和执行，是实现可重复构建和跨平台兼容性的基础。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L0-L22)

## 项目元数据配置
`[project]` 段定义了项目的静态元数据，这些信息在包分发和安装过程中被使用。

- **name**: `"specify-cli"` 指定了包的分发名称，用户通过 `pip install specify-cli` 或 `uv tool install specify-cli` 来安装此工具。
- **version**: `"0.0.17"` 遵循语义化版本控制（SemVer），用于管理包的发布周期和依赖解析。
- **description**: 提供了对工具功能的简要描述，说明它是 GitHub Spec Kit 的一部分，用于引导 Spec-Driven Development (SDD) 项目的启动。
- **requires-python**: `">=3.11"` 明确声明了该工具所需的最低 Python 版本，确保了代码中使用的现代 Python 特性（如类型联合运算符 `|`）能够正常工作。

这些元数据不仅用于包管理器，也用于生成 PyPI 页面和命令行帮助信息。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L1-L7)

## 命令行脚本入口映射
`[project.scripts]` 段是实现命令行工具的关键，它将一个可执行命令映射到 Python 包中的特定函数。

```toml
[project.scripts]
specify = "specify_cli:main"
```

此配置的含义是：
- 当用户安装包后运行 `specify` 命令时，Python 包管理器会查找名为 `specify_cli` 的模块。
- 在该模块中，它会执行 `main` 函数作为程序的入口点。

在代码中，`src/specify_cli/__init__.py` 文件定义了 `main` 函数：

```python
def main():
    app()
```

这里的 `app` 是一个由 `Typer` 库创建的命令行应用实例。`Typer` 是一个基于 Python 类型提示构建 CLI 的库，它会自动将函数参数转换为命令行选项。因此，`specify init` 等子命令的逻辑都由 `@app.command()` 装饰的函数（如 `init` 函数）处理。

这种映射机制使得用户无需关心 Python 模块路径，即可通过一个简洁的命令来使用工具。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L15-L16)
- [src/specify_cli/__init__.py](file://src/specify_cli/__init__.py#L1145-L1146)

## 构建系统配置
`[build-system]` 和 `[tool.hatch.build.targets.wheel]` 段定义了如何构建和打包此项目。

- **[build-system]**:
  - `requires = ["hatchling"]`：声明了构建此项目所需的依赖。`hatchling` 是 Hatch 构建后端的轻量级版本，它告诉包管理器（如 pip）在构建时需要先安装 `hatchling`。
  - `build-backend = "hatchling.build"`：指定了实际执行构建过程的后端。`hatchling` 负责读取 `pyproject.toml` 中的配置并生成符合标准的 wheel 包。

- **[tool.hatch.build.targets.wheel]**:
  - `packages = ["src/specify_cli"]`：这是一个 Hatch 特定的配置，明确指定了哪些 Python 包需要被打包。它使用 `src` 源布局（`src/` 目录），这是一种最佳实践，可以避免测试文件或开发脚本被意外打包。

这个配置确保了构建过程是可重复的，无论在哪个环境中，只要安装了 `hatchling`，就能生成完全相同的分发包。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L18-L22)

## 依赖管理与运行环境
`[project]` 段中的 `dependencies` 列表定义了运行此工具所必需的第三方库。

```toml
dependencies = [
    "typer",
    "rich",
    "httpx[socks]",
    "platformdirs",
    "readchar",
    "truststore>=0.10.4",
]
```

- **typer**: 用于构建功能丰富的命令行界面，支持子命令、选项、参数和自动生成帮助文档。
- **rich**: 用于在终端中渲染富文本，如彩色输出、进度条、表格和 ASCII 艺术字（如 `specify` 命令启动时显示的 banner）。
- **httpx[socks]**: 一个现代的 HTTP 客户端，`[socks]` 是一个可选依赖项（extras），用于支持通过 SOCKS 代理进行网络请求，这对于企业网络环境至关重要。
- **platformdirs**: 用于跨平台地确定用户数据、配置和缓存目录的路径（如 `~/.local/share` on Linux, `%APPDATA%` on Windows）。
- **readchar**: 用于跨平台地读取单个字符输入，支持箭头键导航，用于实现交互式选择菜单。
- **truststore>=0.10.4**: 用于在 Python 中自动使用操作系统的根证书颁发机构（CA）证书，无需手动配置 `certifi`，提高了 SSL/TLS 连接的安全性和易用性。

`requires-python = ">=3.11"` 进一步约束了运行环境，确保了与现代 Python 版本的兼容性。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L5-L14)

## 可重复构建与跨平台兼容性
`pyproject.toml` 文件是实现可重复构建和跨平台兼容性的核心。

- **可重复构建**: 通过将所有构建依赖（`hatchling`）和运行依赖（`typer`, `rich` 等）明确声明在 `pyproject.toml` 中，任何人在任何机器上使用 `pip install` 或 `uv tool install` 都能获得完全相同的依赖集合和构建结果。这消除了“在我机器上能运行”的问题。
- **跨平台兼容性**: 依赖项的选择本身就体现了跨平台设计：
  - `typer` 和 `rich` 在 Windows、macOS 和 Linux 上都能正常工作。
  - `platformdirs` 抽象了不同操作系统的路径差异。
  - `readchar` 处理了不同平台的键盘输入差异。
  - `truststore` 利用操作系统的原生证书存储，避免了跨平台证书管理的复杂性。

此外，`src` 源布局和明确的包声明也确保了构建过程的一致性。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L1-L22)
- [src/specify_cli/__init__.py](file://src/specify_cli/__init__.py#L0-L59)

## 实际修改示例
### 修改依赖项
假设需要添加 `requests` 库以支持另一种 HTTP 客户端。修改 `pyproject.toml`：

```toml
[project]
# ... 其他配置
dependencies = [
    "typer",
    "rich",
    "httpx[socks]",
    "platformdirs",
    "readchar",
    "truststore>=0.10.4",
    "requests", # 新增依赖
]
```

**影响**:
- 下次安装或更新 `specify-cli` 时，`pip` 或 `uv` 会自动安装 `requests` 及其所有依赖。
- 开发者可以在 `src/specify_cli/__init__.py` 中安全地 `import requests`。

### 扩展 CLI 命令
假设要添加一个 `validate` 命令。首先，在 `src/specify_cli/__init__.py` 中定义函数：

```python
@app.command()
def validate():
    """Validate the current project's specification."""
    console.print("Validating project specification...")
    # 验证逻辑
```

然后，修改 `pyproject.toml` 中的 `description` 以反映新功能：

```toml
[project]
description = "Specify CLI, part of GitHub Spec Kit. A tool to bootstrap and validate your projects for Spec-Driven Development (SDD)."
# ... 其他配置
```

**影响**:
- 用户运行 `specify --help` 时，会看到新的 `validate` 命令。
- 用户可以直接运行 `specify validate` 来执行新功能。
- 更新 `description` 确保了包的元数据与实际功能保持同步。

**Section sources**
- [pyproject.toml](file://pyproject.toml#L1-L22)
- [src/specify_cli/__init__.py](file://src/specify_cli/__init__.py#L749-L789)

## 结论
`pyproject.toml` 文件是 Specify CLI 工具的基石。它通过标准化的格式，清晰地定义了项目的名称、版本、描述、运行时依赖、构建依赖和命令行入口。这种声明式配置不仅简化了工具的安装和使用，还确保了构建过程的可重复性和跨平台兼容性。理解此文件的各个部分对于维护和扩展 Spec-Kit 项目至关重要。