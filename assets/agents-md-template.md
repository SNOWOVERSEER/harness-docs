# AGENTS.md Standard Template

Below is a ready-to-fill AGENTS.md template. Target: ~100 lines (matching OpenAI's practice of keeping the instruction file as a "table of contents, not an encyclopedia"). Hard ceiling: 200 lines.

---

```markdown
<!-- last_verified: YYYY-MM-DD -->

# [Project Name]

[One sentence describing what the project does]

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

## Usage notes

1. Replace all `[placeholder]` entries with actual content
2. Keep Common Mistakes to the top 5-10 most frequent entries
3. If a docs/ file does not exist, remove that row from the Documentation Map
4. After filling in, run `wc -l AGENTS.md` to verify — target ~100 lines, hard ceiling 200 lines
5. If over 200 lines, move the least frequently needed content to the corresponding docs/ file
6. For Claude Code: consider using `@docs/conventions.md` for files needed in 80%+ of sessions; use plain text pointers (`→ docs/file.md`) for on-demand files
