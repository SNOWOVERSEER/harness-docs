# docs/ 目录标准结构

## 标准文件清单

```
docs/
├── architecture.md        — 模块边界、依赖方向、数据流
├── conventions.md         — 命名规范、文件组织、代码风格
├── commands.md            — 所有可用工具/脚本的精确调用语法
├── decisions.md           — 架构决策记录 (ADR format)
├── troubleshooting.md     — 已知失败模式及修复步骤
├── observability.md       — Agent 可用的日志/指标/链路查询方式及自我验证清单
└── feature-tracker.json   — 功能追踪 (JSON, 非 Markdown)
```

## 可选扩展文件（大型项目按需添加）

```
docs/
├── api-reference.md       — API endpoint 清单及请求/响应格式
├── deployment.md          — 部署流程及环境配置
├── testing-guide.md       — 测试策略、fixture 说明、mock 规范
├── migration-guide.md     — 数据库/API 版本迁移步骤
└── references/            — 深层参考资料目录
    ├── schema-definitions.md
    └── third-party-apis.md
```

---

## 每个文件的结构模板

### architecture.md

```markdown
<!-- last_verified: YYYY-MM-DD -->
<!-- related_paths: src/ -->

# Architecture

## Module Map

[列出所有顶层模块及其职责，每个模块 1-2 行]

## Dependency Direction

[声明依赖流向规则，例:]
Types → Config → Repo → Service → Runtime → UI

禁止反向导入。违规检测: `[lint command]`

## Data Flow

[描述数据如何从输入到输出流经各模块]

## Module Boundaries

| Module | Responsibility | Allowed imports | Forbidden imports |
|---|---|---|---|
| types/ | 类型定义 | (none) | everything else |
| service/ | 业务逻辑 | types, config, repo | runtime, ui |
```

### conventions.md

```markdown
<!-- last_verified: YYYY-MM-DD -->
<!-- related_paths: src/ -->

# Conventions

## File Naming
[精确规则 + 示例]

## Function/Variable Naming
[精确规则 + 示例]

## File Organization
[目录结构规则]

## Import Order
[import 排序规则 + 自动格式化命令]

## Error Handling
[统一错误处理模式 + 示例代码]
```

### commands.md

```markdown
<!-- last_verified: YYYY-MM-DD -->

# Commands

每个命令包含: 精确调用语法、工作目录、预期输出、常见失败原因。

## Build
\`\`\`bash
# Working directory: project root
[exact command]
# Expected: [what success looks like]
# Common failure: [most common failure + fix]
\`\`\`

## Test
\`\`\`bash
# Working directory: project root
# All tests:
[command]
# Single file:
[command with path placeholder]
# With coverage:
[command]
\`\`\`

[Continue for: lint, format, run-dev, deploy, db-migrate, etc.]
```

### decisions.md

```markdown
<!-- last_verified: YYYY-MM-DD -->

# Architecture Decision Records

## ADR-001: [Decision Title]
- **Date**: YYYY-MM-DD
- **Status**: accepted | superseded | deprecated
- **Context**: [为什么需要做这个决策，1-2句]
- **Decision**: [做了什么决策]
- **Consequences**: [这个决策带来的影响]
- **Alternatives considered**: [考虑过的其他方案及为什么不选]
```

### troubleshooting.md

```markdown
<!-- last_verified: YYYY-MM-DD -->

# Troubleshooting

每条包含: 症状、原因、修复步骤。格式遵循「错误消息即修复指南」原则。

## [Error symptom or message]
**Cause**: [root cause]
**Fix**:
\`\`\`bash
[exact fix commands]
\`\`\`
**Prevention**: [how to avoid this in future]
```
