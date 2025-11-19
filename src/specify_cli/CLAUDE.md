[根目录](../../CLAUDE.md) > [src](../) > **specify_cli**

# Specify CLI - 命令行工具模块

## 变更记录 (Changelog)

**2025-11-17**: 初始化模块文档，从主要CLI文件分析得到

## 模块职责

specify_cli是Spec Kit的核心命令行工具，负责：
- 项目初始化和模板下载
- AI代理支持和配置
- 跨平台脚本执行
- Git仓库初始化
- 环境依赖检查

## 入口与启动

### 主入口文件
- **文件**: `__init__.py`
- **入口函数**: `main()`
- **调用方式**:
  ```bash
  uvx specify-cli.py <command>
  # 或安装后
  specify <command>
  ```

### 核心类和函数
- **`app`**: Typer应用实例，定义CLI命令结构
- **`main()`**: 应用程序入口点
- **`show_banner()`**: 显示ASCII艺术标题
- **`BannerGroup`**: 自定义Typer组，显示帮助前的横幅

### 支持的命令
1. **`init`** - 初始化新的Spec项目
2. **`check`** - 检查已安装工具

## 对外接口

### CLI命令接口

#### `init` 命令参数
```bash
specify init [PROJECT_NAME] [options]
```

**参数**:
- `PROJECT_NAME`: 项目名称（可选，使用--here或.时不需要）

**选项**:
- `--ai <agent>`: AI代理选择 (claude, gemini, copilot等)
- `--script <type>`: 脚本类型 (sh/ps)
- `--ignore-agent-tools`: 跳过AI工具检查
- `--no-git`: 跳过Git初始化
- `--here`: 在当前目录初始化
- `--force`: 强制合并（跳过确认）
- `--skip-tls`: 跳过SSL验证
- `--debug`: 显示详细调试信息
- `--github-token`: GitHub API令牌

#### `check` 命令
```bash
specify check
```
检查系统工具和AI代理安装状态。

### 配置接口
- **AGENT_CONFIG**: AI代理配置字典
- **SCRIPT_TYPE_CHOICES**: 脚本类型选择
- **BANNER/TAGLINE**: CLI横幅和标语

## 关键依赖与配置

### Python依赖 (pyproject.toml)
```toml
dependencies = [
    "typer",           # CLI框架
    "rich",            # 终端美化
    "httpx[socks]",    # HTTP客户端
    "platformdirs",    # 跨平台目录
    "readchar",        # 键盘输入
    "truststore>=0.10.4", # SSL证书
]
```

### 外部工具依赖
- **git**: 版本控制（可选）
- **Python 3.11+**: 运行环境
- **uv**: 包管理器

### AI代理配置
支持14种AI代理，每个代理包含：
- `name`: 显示名称
- `folder`: 项目内配置文件夹
- `install_url`: 安装指南URL
- `requires_cli`: 是否需要CLI工具

### 核心配置常量
- `CLAUDE_LOCAL_PATH`: Claude CLI本地路径
- `MAX_BRANCH_LENGTH`: GitHub分支名长度限制（244字节）

## 数据模型

### 代理配置模型
```python
AGENT_CONFIG = {
    "agent_key": {
        "name": "Human Readable Name",
        "folder": ".agent_folder/",
        "install_url": "https://install.guide",
        "requires_cli": bool
    }
}
```

### 步骤跟踪模型
```python
class StepTracker:
    - title: str
    - steps: List[Dict]
    - status_order: Dict[str, int]
```

### 项目配置模型
- **项目路径**: 当前工作目录或指定路径
- **AI代理**: 用户选择的AI开发工具
- **脚本类型**: bash/ps选择
- **Git配置**: 是否初始化Git仓库

## 测试与质量

### 当前测试状态
- ❌ 无正式测试文件
- ❌ 无测试配置
- ❌ 无CI/CD集成

### 建议测试策略
1. **单元测试**:
   - CLI参数解析验证
   - AI代理配置检查
   - 步骤跟踪器功能

2. **集成测试**:
   - 端到端项目初始化
   - 模板下载和解压
   - Git仓库创建

3. **系统测试**:
   - 跨平台兼容性
   - 不同AI代理支持
   - 网络异常处理

### 质量检查点
- 参数验证完整性
- 错误处理和用户友好提示
- 进度显示和用户体验
- 安全性（令牌处理、SSL验证）

## 常见问题 (FAQ)

### Q: 如何添加新的AI代理支持？
A: 在`AGENT_CONFIG`字典中添加新代理配置，包含name、folder、install_url、requires_cli字段。

### Q: CLI横幅如何自定义？
A: 修改`BANNER`和`TAGLINE`常量，或更新`show_banner()`函数。

### Q: 如何处理网络下载失败？
A: 使用`--debug`选项获取详细错误信息，检查网络连接和GitHub令牌配置。

### Q: 跨平台兼容性如何保证？
A: 使用`platformdirs`处理目录，`os.name`检测操作系统，提供PowerShell脚本版本。

## 相关文件清单

### 核心文件
- `__init__.py` - 主要CLI实现（~1200行）
- `pyproject.toml` - 项目配置和依赖

### 依赖文件
- 无独立的测试文件
- 无独立的配置文件
- 依赖根目录的脚本和模板

### 配置项
- AI代理配置: 内嵌在代码中
- 步骤跟踪: 实时UI更新
- 网络配置: SSL/TLS设置

## 变更记录 (Changelog)

**2025-11-17**:
- 初始化模块文档
- 分析主要CLI功能和架构
- 识别测试覆盖缺口
- 提出质量改进建议

---

*最后更新: 2025-11-17*
*模块版本: 1.0.0*