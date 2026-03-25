# AGENTS.md 标准模板

以下是一个可直接填写的 AGENTS.md 模板。目标: 控制在 100 行以内。

---

```markdown
<!-- last_verified: YYYY-MM-DD -->

# [Project Name]

[一句话描述项目做什么]

## Quick Commands

```bash
# Build
[build command]

# Test (all)
[test command]

# Test (single file)
[single test command with placeholder]

# Lint
[lint command]

# Run (development)
[dev run command]
```

## Architecture

→ `docs/architecture.md`
When to read: creating new modules, modifying cross-module calls, adding dependencies

Key constraint: Dependencies flow [direction description, e.g., "Types → Config → Repo → Service → Runtime → UI"]. Never import against this direction.

## Code Conventions

→ `docs/conventions.md`
When to read: writing new code, naming files/functions/variables

## Common Mistakes

1. [Most frequent mistake + one-line fix]
2. [Second most frequent mistake + one-line fix]
3. [Third most frequent mistake + one-line fix]

## Available Tools

→ `docs/commands.md`
When to read: need to run any build/test/deploy/utility command

## Troubleshooting

→ `docs/troubleshooting.md`
When to read: encountering errors, build failures, unexpected behavior

## Documentation Map

| File | When to read | Contains |
|---|---|---|
| `docs/architecture.md` | Modifying module structure | Dependencies, boundaries, data flow |
| `docs/conventions.md` | Writing new code | Naming, file org, style rules |
| `docs/commands.md` | Running any command | All tools with exact syntax |
| `docs/decisions.md` | Understanding design rationale | ADRs with context and outcome |
| `docs/troubleshooting.md` | Hitting errors | Known issues + fix steps |
| `docs/observability.md` | Verifying changes work | Log/metric/trace queries + self-check |
| `docs/feature-tracker.json` | Checking feature status | Feature list + state (JSON) |
```

---

## 使用说明

1. 用实际内容替换所有 `[placeholder]`
2. Common Mistakes 保留出现频率最高的 5-10 条
3. 如果某个 docs/ 文件不存在，从 Documentation Map 中删除该行
4. 填写完成后运行 `wc -l AGENTS.md` 验证——目标 80-150 行，硬上限 200 行
5. 如果超过 200 行，把最不常用的内容移到对应的 docs/ 文件中
