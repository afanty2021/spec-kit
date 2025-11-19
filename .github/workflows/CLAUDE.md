[根目录](../../CLAUDE.md) > [.github](../) > **workflows**

# GitHub Actions CI/CD 模块

## 变更记录 (Changelog)

**2025-11-17 17:34**: 创建CI/CD模块文档，完成增量更新

## 模块职责

本模块包含项目的自动化工作流，负责：
- **自动化发布流程**: 版本管理、发布包创建、GitHub Release
- **文档自动部署**: DocFX构建和GitHub Pages部署
- **代码质量检查**: Markdown文件格式验证
- **多AI代理支持**: 为14种不同AI代理生成专用模板包

## 工作流结构

### 主要工作流

| 工作流文件 | 触发条件 | 主要功能 | 关键特性 |
|------------|----------|----------|----------|
| `release.yml` | 主分支推送(memory/scripts/templates/workflows变更) | 自动发布流程 | 版本递增、多AI代理包生成、GitHub Release创建 |
| `docs.yml` | docs目录变更 | 文档构建部署 | DocFX构建、GitHub Pages部署、.NET 8.x运行时 |
| `lint.yml` | 主分支推送/PR | 代码质量检查 | Markdownlint集成、全量Markdown文件检查 |

### 自动化脚本

| 脚本文件 | 功能 | 使用场景 |
|----------|------|----------|
| `get-next-version.sh` | 版本计算 | 基于最新Git标签计算下一个版本号 |
| `check-release-exists.sh` | 发布检查 | 验证指定版本的Release是否已存在 |
| `create-release-packages.sh` | 包生成 | 为14种AI代理生成专用模板包 |
| `generate-release-notes.sh` | 发布笔记 | 基于Git历史生成自动发布说明 |
| `create-github-release.sh` | 发布创建 | 创建GitHub Release并上传资产 |
| `update-version.sh` | 版本更新 | 更新pyproject.toml中的版本信息 |

## 支持的AI代理

### 完整列表
- claude, gemini, copilot, cursor-agent, qwen
- opencode, codex, windsurf, kilocode, auggie
- codebuddy, roo, q, amp

### 包生成特性
- **多脚本类型**: 支持Bash(.sh)和PowerShell(.ps)脚本
- **路径重写**: 自动调整为`.specify/`子目录结构
- **命令生成**: 为每个AI代理生成专用的斜杠命令实现
- **模板定制**: 基于AI代理特性调整命令参数格式

## 触发机制

### 自动发布触发
```yaml
paths:
  - 'memory/**'      # 项目原则变更
  - 'scripts/**'     # 自动化脚本变更
  - 'templates/**'   # 模板系统变更
  - '.github/workflows/**'  # 工作流自身变更
```

### 文档部署触发
```yaml
paths:
  - 'docs/**'        # 文档内容变更
```

### 质量检查触发
```yaml
branches: ["main"]   # 主分支推送
pull_request:        # PR创建/更新
```

## 权限与安全

### Release工作流权限
```yaml
permissions:
  contents: write    # 仓库内容写入
  pull-requests: write  # PR操作权限
```

### 文档部署权限
```yaml
permissions:
  contents: read     # 仓库内容读取
  pages: write       # GitHub Pages写入
  id-token: write    # OIDC认证
```

## 关键配置

### 并发控制
```yaml
concurrency:
  group: "pages"
  cancel-in-progress: false  # 允许生产部署完成
```

### 运行环境
- **主要**: ubuntu-latest
- **文档构建**: .NET 8.x运行时
- **工具链**: DocFX, markdownlint-cli2

## 集成特性

### 版本管理
- 自动基于Git标签递增版本号
- 严格版本格式验证(vX.Y.Z)
- 防重复发布检查

### 多格式支持
- **Markdown**: 所有文档和模板
- **Shell脚本**: Bash和PowerShell双重支持
- **YAML配置**: GitHub Actions工作流定义
- **TOML配置**: Python项目配置

### 自动化程度
- **完全自动化**: 从代码变更到发布部署
- **手动触发**: 支持workflow_dispatch手动执行
- **智能跳过**: 已存在版本自动跳过发布

## 维护与监控

### 日常维护
- 监控工作流执行状态
- 定期检查发布包完整性
- 更新Action依赖版本

### 故障排查
- 检查脚本权限设置(chmod +x)
- 验证环境变量和secrets配置
- 查看工作流日志获取详细错误信息

## 相关文件清单

### 工作流定义
- `.github/workflows/release.yml` - 发布工作流
- `.github/workflows/docs.yml` - 文档部署工作流
- `.github/workflows/lint.yml` - 质量检查工作流

### 脚本工具
- `.github/workflows/scripts/get-next-version.sh` - 版本计算
- `.github/workflows/scripts/check-release-exists.sh` - 发布检查
- `.github/workflows/scripts/create-release-packages.sh` - 包生成
- `.github/workflows/scripts/generate-release-notes.sh` - 发布笔记
- `.github/workflows/scripts/create-github-release.sh` - 发布创建
- `.github/workflows/scripts/update-version.sh` - 版本更新

### 配置文件
- `pyproject.toml` - Python项目配置(版本信息更新目标)
- `docs/docfx.json` - DocFX文档配置
- `docs/toc.yml` - 文档目录结构

---

*最后更新: 2025-11-17 17:34*
*模块版本: 1.0.0*