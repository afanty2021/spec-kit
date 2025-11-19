[根目录](../../CLAUDE.md) > **scripts**

# Scripts - 自动化脚本模块

## 变更记录 (Changelog)

**2025-11-17**: 初始化自动化脚本模块文档

## 模块职责

scripts模块提供跨平台的自动化脚本，负责：
- 项目初始化和环境配置
- 功能分支创建和管理
- 开发环境工具检查
- Git工作流自动化
- AI代理上下文更新

## 入口与启动

### 脚本组织结构
```
scripts/
├── bash/           # Unix/Linux/macOS脚本
│   ├── check-prerequisites.sh
│   ├── common.sh
│   ├── create-new-feature.sh
│   ├── setup-plan.sh
│   └── update-agent-context.sh
└── powershell/     # Windows PowerShell脚本
    ├── check-prerequisites.ps1
    ├── common.ps1
    ├── create-new-feature.ps1
    ├── setup-plan.ps1
    └── update-agent-context.ps1
```

### 脚本执行入口
- **Bash**: `bash scripts/bash/<script>.sh`
- **PowerShell**: `powershell scripts/powershell/<script>.ps1`

## 对外接口

### 核心脚本接口

#### create-new-feature脚本
**功能**: 创建新功能分支和规格文件
**用法**:
```bash
# Bash
./scripts/bash/create-new-feature.sh "Add user authentication" --short-name "user-auth"

# PowerShell
./scripts/powershell/create-new-feature.ps1 "Add user authentication" -ShortName "user-auth"
```

**参数**:
- 位置参数: 功能描述
- `--short-name/-ShortName`: 自定义短名称（2-4个词）
- `--number/-Number`: 手动指定分支编号
- `--json/-Json`: JSON格式输出

**输出**:
- 新的Git分支: `XXX-feature-name`
- 规格文件: `specs/XXX-feature-name/spec.md`
- 环境变量: `SPECIFY_FEATURE`

#### check-prerequisites脚本
**功能**: 检查开发环境工具安装状态
**检查项**:
- Git版本控制
- Python环境
- AI代理工具安装
- uv包管理器
- 操作系统兼容性

#### setup-plan脚本
**功能**: 设置开发环境和项目配置
**任务**:
- 环境变量配置
- 工具链验证
- 项目模板准备
- 权限设置

#### update-agent-context脚本
**功能**: 更新AI代理的上下文信息
**更新内容**:
- 项目结构变更
- 新增模板和脚本
- 配置文件同步
- 开发指南更新

### 公共库接口

#### common.sh/common.ps1
**功能**: 提供公共函数和变量
**主要函数**:
- 日志输出函数
- 错误处理
- 路径解析
- 系统检测

## 关键依赖与配置

### 外部工具依赖
- **git**: 版本控制系统
- **Python 3.11+**: 运行环境
- **uv**: 包管理工具
- **各AI代理CLI**: 根据使用情况

### 操作系统支持
- **Linux**: 完整支持
- **macOS**: 完整支持
- **Windows**: PowerShell支持

### Git工作流集成
- **分支命名**: `XXX-feature-name`格式
- **规格目录**: `specs/XXX-feature-name/`
- **模板复制**: 自动从`.specify/templates/`复制
- **环境变量**: 设置`SPECIFY_FEATURE`

## 数据模型

### 分支命名模型
```bash
# 格式: [编号]-[功能短名称]
BRANCH_NAME="${FEATURE_NUM}-${BRANCH_SUFFIX}"
```

### 功能目录结构
```
specs/
└── XXX-feature-name/
    ├── spec.md          # 功能规格
    ├── plan.md          # 技术计划
    ├── tasks.md         # 任务分解
    └── ...             # 其他生成文件
```

### JSON输出模型
```json
{
  "BRANCH_NAME": "XXX-feature-name",
  "SPEC_FILE": "specs/XXX-feature-name/spec.md",
  "FEATURE_NUM": "XXX"
}
```

## 测试与质量

### 脚本质量标准
1. **错误处理**: 使用`set -e`确保错误时退出
2. **参数验证**: 检查必需参数和格式
3. **跨平台兼容**: Bash和PowerShell功能对等
4. **用户友好**: 清晰的帮助信息和使用示例

### 测试策略
- **单元测试**: 每个函数的独立测试
- **集成测试**: 脚本间协作测试
- **系统测试**: 跨平台兼容性测试
- **端到端测试**: 完整工作流测试

### 安全考虑
- **路径验证**: 防止路径遍历攻击
- **权限控制**: 最小权限原则
- **输入清理**: 用户输入验证和清理
- **敏感信息**: 避免在日志中暴露令牌

## 常见问题 (FAQ)

### Q: 脚本在Windows上运行有问题？
A: 确保使用PowerShell版本，检查执行策略设置。

### Q: 分支名过长如何处理？
A: 脚本会自动截断到GitHub限制的244字节。

### Q: 如何自定义模板位置？
A: 修改脚本中的模板路径变量或设置环境变量。

### Q: Git操作失败如何处理？
A: 检查Git配置、网络连接和权限设置。

## 相关文件清单

### Bash脚本
- `check-prerequisites.sh` - 环境检查脚本
- `common.sh` - 公共函数库
- `create-new-feature.sh` - 功能创建脚本
- `setup-plan.sh` - 环境设置脚本
- `update-agent-context.sh` - 上下文更新脚本

### PowerShell脚本
- `check-prerequisites.ps1` - 环境检查脚本（Windows版）
- `common.ps1` - 公共函数库（Windows版）
- `create-new-feature.ps1` - 功能创建脚本（Windows版）
- `setup-plan.ps1` - 环境设置脚本（Windows版）
- `update-agent-context.ps1` - 上下文更新脚本（Windows版）

## 子模块文档

### bash子模块
详见: [bash/CLAUDE.md](./bash/CLAUDE.md)

### powershell子模块
详见: [powershell/CLAUDE.md](./powershell/CLAUDE.md)

## 最佳实践

### 脚本开发指南
1. **一致性**: Bash和PowerShell版本保持功能一致
2. **可读性**: 使用清晰的函数命名和注释
3. **可维护性**: 模块化设计，避免重复代码
4. **健壮性**: 完善的错误处理和日志记录

### 使用建议
1. **执行权限**: 确保脚本有执行权限
2. **环境检查**: 运行前检查prerequisites
3. **备份重要数据**: 特别是在处理Git仓库时
4. **日志监控**: 关注脚本输出的警告和错误信息

## 变更记录 (Changelog)

**2025-11-17**:
- 初始化脚本模块文档
- 分析脚本结构和功能
- 定义数据模型和质量标准
- 建立跨平台兼容性指导

---

*最后更新: 2025-11-17*
*模块版本: 1.0.0*