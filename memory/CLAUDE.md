[根目录](../../CLAUDE.md) > **memory**

# Memory - 知识库模块

## 变更记录 (Changelog)

**2025-11-17**: 初始化知识库模块文档

## 模块职责

memory模块是Spec Kit的知识库和记忆系统，负责：
- 存储项目治理原则和开发指南
- 维护团队共识和最佳实践
- 提供AI代理的上下文信息
- 支持版本化的知识管理
- 确保开发决策的一致性和可追溯性

## 入口与启动

### 知识库入口文件
- **项目宪法**: `constitution.md` - 项目治理原则模板

### 访问方式
- **AI代理读取**: 通过`/memory/constitution.md`路径访问
- **人类维护**: 直接编辑Markdown文件
- **版本控制**: 通过Git管理变更历史

## 对外接口

### 宪法模板接口
**文件**: `constitution.md`

**核心结构**:
```markdown
# [PROJECT_NAME] Constitution

## Core Principles
### [PRINCIPLE_1_NAME]
[PRINCIPLE_1_DESCRIPTION]

## [SECTION_2_NAME]
[SECTION_2_CONTENT]

## Governance
[GOVERNANCE_RULES]

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
```

### 占位符系统
- `[PROJECT_NAME]`: 项目名称
- `[PRINCIPLE_*_NAME]`: 原则名称
- `[PRINCIPLE_*_DESCRIPTION]`: 原则描述
- `[SECTION_*_NAME]`: 章节名称
- `[CONSTITUTION_VERSION]`: 版本号
- `[RATIFICATION_DATE]`: 批准日期
- `[LAST_AMENDED_DATE]`: 最后修改日期

### 更新触发机制
- **用户命令**: `/speckit.constitution`命令触发更新
- **模板同步**: 修改后自动同步相关模板
- **版本管理**: 语义化版本控制（MAJOR.MINOR.PATCH）

## 关键依赖与配置

### 模板依赖关系
- **spec-template.md**: 需要引用宪法原则
- **plan-template.md**: 需要遵循宪法约束
- **agent-file-template.md**: 需要包含宪法指导

### 同步机制
宪法更新时会触发以下同步检查：
1. 检查所有模板文件的一致性
2. 更新agent-file-template中的技术栈信息
3. 验证命令模板中的引用正确性
4. 生成同步影响报告

### 版本控制规则
- **MAJOR版本**: 向后不兼容的治理变更
- **MINOR版本**: 新增原则或重要扩展
- **PATCH版本**: 澄清、措辞修改、非语义调整

## 数据模型

### 原则模型
```markdown
### [PRINCIPLE_NAME]
<!-- 示例: I. Library-First -->
[PRINCIPLE_DESCRIPTION]
<!-- 示例: 每个功能都作为独立库开始；库必须是自包含、可独立测试、有文档；需要明确目的 - 不允许仅组织性的库 -->
```

### 治理模型
```markdown
## Governance
[GOVERNANCE_RULES]
<!-- 示例: 宪法高于所有其他实践；修订需要文档、批准、迁移计划 -->
```

### 元数据模型
```markdown
**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
<!-- 示例: Version: 2.1.1 | Ratified: 2025-06-13 | Last Amended: 2025-07-16 -->
```

### 同步报告模型
```html
<!-- Sync Impact Report -->
- Version change: old → new
- Modified principles: [列表]
- Added sections: [列表]
- Removed sections: [列表]
- Templates requiring updates: ✅/⚠ 状态
- Follow-up TODOs: [列表]
```

## 测试与质量

### 宪法质量标准
1. **原则清晰性**: 每个原则都是可声明、可测试的
2. **无模糊语言**: 避免使用"应该"，替换为"必须/应该"的理由
3. **完整性**: 覆盖项目的关键治理方面
4. **可执行性**: 原则能够指导实际开发决策

### 同步验证标准
1. **模板一致性**: 所有模板都反映最新的宪法原则
2. **引用完整性**: 所有引用都是准确和当前的
3. **版本一致性**: 版本号与变更内容匹配
4. **日期格式**: 使用ISO 8601格式（YYYY-MM-DD）

### 占位符验证
- 无未解释的括号标记
- 所有占位符都有明确的替换值
- 保留的占位符有明确的理由说明

## 常见问题 (FAQ)

### Q: 如何添加新的治理原则？
A: 在Core Principles部分添加新原则，增加MINOR版本号。

### Q: 宪法修改后如何同步模板？
A: 使用`/speckit.constitution`命令会自动触发同步检查。

### Q: 版本号如何确定？
A: 根据变更的影响程度：MAJOR（不兼容）、MINOR（新增）、PATCH（修复）。

### Q: 如何处理宪法冲突？
A: 通过治理流程解决，记录冲突和解决方案。

## 相关文件清单

### 核心文件
- `constitution.md` - 项目宪法模板

### 依赖文件
- `../templates/spec-template.md` - 需要引用宪法约束
- `../templates/plan-template.md` - 需要遵循宪法指导
- `../templates/agent-file-template.md` - 需要包含宪法信息
- `../templates/commands/*.md` - 需要更新宪法引用

## 最佳实践

### 宪法写作指南
1. **声明式语言**: 使用"系统必须"而非"系统应该"
2. **具体可测**: 每个原则都能被客观验证
3. **业务导向**: 原则应反映业务价值和技术目标
4. **简洁明了**: 避免冗长复杂的描述

### 版本管理建议
1. **语义化版本**: 严格遵循MAJOR.MINOR.PATCH规则
2. **变更日志**: 记录每次版本变更的原因和影响
3. **向后兼容**: 尽量保持向后兼容性
4. **迁移计划**: 为重大变更提供迁移路径

### 团队协作指南
1. **共识建立**: 重要原则变更需要团队共识
2. **文档同步**: 确保所有团队成员了解最新宪法
3. **定期审查**: 定期审查和更新宪法内容
4. **培训支持**: 为新成员提供宪法培训

## 使用示例

### 宪法内容示例
```markdown
# TaskFlow Constitution

## Core Principles

### I. Library-First
Every feature starts as a standalone library. Libraries must be self-contained, independently testable, and documented. Clear purpose is required - no organizational-only libraries.

### II. Test-First (NON-NEGOTIABLE)
TDD is mandatory: Tests written → User approved → Tests fail → Then implement. Red-Green-Refactor cycle is strictly enforced.

## Governance
This constitution supersedes all other practices. Amendments require documentation, approval, and migration plans. All PRs/reviews must verify compliance.

**Version**: 1.0.0 | **Ratified**: 2025-01-15 | **Last Amended**: 2025-01-15
```

## 变更记录 (Changelog)

**2025-11-17**:
- 初始化知识库模块文档
- 分析宪法模板结构和占位符系统
- 定义数据模型和质量标准
- 建立最佳实践和使用指南

---

*最后更新: 2025-11-17*
*模块版本: 1.0.0*