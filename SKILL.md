---
name: harness-docs
description: |
  Agent-optimized documentation system based on OpenAI Harness Engineering. Audits, creates, and maintains AGENTS.md + docs/ directories for maximum agent readability.
  TRIGGERS: AGENTS.md, harness docs, agent documentation, "audit my docs", "set up documentation", documentation system, 文档规范化, 一键整理文档. Also trigger on repeated agent mistakes or user complaints about agent behavior — even without explicit mention of "documentation" (e.g., "又搞错了", "it keeps doing X wrong", "I told you this before"). These signal documentation gaps needing updates.
---

# Harness Docs

A skill for building and maintaining documentation systems optimized for AI agents, based on OpenAI's Harness Engineering principles.

## Why this exists

Most project documentation is written for humans who already have context. Agents don't have that luxury — they only know what's in their context window. A 500-line AGENTS.md full of outdated rules is worse than no AGENTS.md at all: it wastes tokens, sends agents down wrong paths, and nobody maintains it because it's too big to review.

This skill implements the core insight from OpenAI's Codex team: **the agent instruction file should be a directory, not an encyclopedia**. A concise index (~100 lines, hard ceiling 200) that points to deeper, structured documentation in `docs/`, with JSON for anything that needs to be tracked as state.

### Agent instruction file naming

Different tools use different file names for the same concept. This skill supports all of them:

| Tool | File name |
|---|---|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex / generic | `AGENTS.md` |
| Cursor | `.cursorrules` or `.cursor/rules` |
| GitHub Copilot | `.github/copilot-instructions.md` |

**Auto-detection**: At the start of any mode, check which file already exists in the project root. Use that name throughout. If none exists, ask the user which tool they primarily use, or default to `CLAUDE.md` for Claude Code users and `AGENTS.md` otherwise. In the rest of this skill, "AGENTS.md" refers to whichever file name is in use for the project.

**Claude Code progressive disclosure**: When the detected tool is Claude Code, three mechanisms map to the three-layer architecture:

| Layer | Mechanism | Loading behavior |
|---|---|---|
| Layer 1 (always in context) | `@docs/file.md` in CLAUDE.md | **Eager** — expanded at session start, consumes tokens every session |
| Layer 2 (conditionally loaded) | `.claude/rules/rule.md` with `paths` frontmatter | **Conditional** — loaded only when Claude opens files matching the path pattern |
| Layer 2 (on-demand) | Plain text pointer: `→ docs/file.md` | **Manual** — agent reads the file only when the task requires it |
| Layer 3 (deep reference) | Links from Layer 2 docs | **Manual** — agent follows links when it needs specific details |

**`@` import rules**:
- `@` files are eagerly loaded — use only for docs needed in 80%+ of sessions
- Never `@`-import all docs/ files — 5 files × 200 lines = 1000+ lines consumed every session
- Relative paths resolve from the file containing the import; recursive up to 5 levels; `@~/` for home directory
- First-time imports show an approval dialog to the user

**`.claude/rules/` for path-scoped rules** (the true lazy loading mechanism):
```yaml
# .claude/rules/api-conventions.md
---
paths: ["src/api/**/*.ts"]
---
All API endpoints MUST use the `/api/v1/` prefix...
```
This rule loads only when Claude works with files matching `src/api/**/*.ts`. Use this for domain-specific conventions that don't belong in every session.

Example — a CLAUDE.md that uses all three layers correctly:
```markdown
# MyProject
@docs/conventions.md          # Layer 1: always needed — eagerly loaded

## Architecture
→ `docs/architecture.md` — read when creating/modifying modules
## Troubleshooting
→ `docs/troubleshooting.md` — read when encountering errors

# Path-scoped rules live in .claude/rules/ (loaded conditionally)
```

## How to determine which mode to use

Read the user's request and match it to one of four modes:

- **Audit**: User has an existing project and wants to check/fix their documentation. Keywords: "audit", "check", "review", "fix", "optimize", "clean up", "整理", "检查", "规范化"
- **Generate**: User wants to create a documentation system from scratch. Keywords: "create", "set up", "initialize", "new project", "从零开始", "新建", "搭建"
- **Update**: User encountered a specific agent failure and wants to encode prevention. Keywords: "agent made a mistake", "keeps doing X wrong", "add a rule", "记录错误", "防止再犯"
- **Detect**: User is NOT explicitly asking for documentation changes, but their message contains implicit signals that documentation should be updated. This is the most important mode because it closes the feedback loop automatically.

After determining the mode, read the relevant reference files before proceeding.

### How Detect mode works

Not every problem is a documentation problem. Before suggesting a doc update, check whether the user's complaint matches one of these patterns:

| Signal pattern | Likely doc issue | Example |
|---|---|---|
| Same mistake repeated 2+ times | Missing or unclear rule | "又用错了", "it keeps doing X" |
| User corrects agent's tool/command choice | Wrong or missing command docs | "不是 npm，是 pnpm" |
| Output format consistently wrong | Missing convention rules | "格式不对", "style is wrong again" |
| Agent makes architecture violation | Missing constraint docs | "不应该在这里 import 那个" |
| Agent doesn't know where to find things | Missing documentation map | "你怎么不知道这个文件在哪" |
| User gives same instruction across sessions | Instruction not persisted in docs | "我说过很多次了" |

When you detect one of these patterns:
1. Complete the user's immediate request first (fix the actual problem)
2. Then proactively suggest: "This looks like a recurring issue. I can encode this as a rule in your documentation so it doesn't happen again. Want me to update [specific file]?"
3. If the user agrees, switch to Update mode (Mode C) to write the prevention entry
4. If the user declines or ignores, drop it — don't push

The key principle: **solve the immediate problem first, then offer to prevent recurrence.** Never delay the user's actual task to discuss documentation.

---

## Mode A: Audit

**Goal**: Analyze every documentation file in the codebase, identify concrete problems, restructure where needed, and produce measurable improvement — not just add headers.

**Brownfield note**: For existing projects with significant legacy documentation, read `references/harness-concepts.md` § "Brownfield Project Adaptation" before starting. Don't try to achieve full compliance on day one — focus on the feedback loop (each agent mistake → one new rule) and adopt incrementally by domain.

### Step 1: Scan all docs

Detect the agent instruction file (see "Agent instruction file naming" above), then build a complete inventory:

1. Read the agent instruction file (`CLAUDE.md` / `AGENTS.md` / etc.) at repo root
2. List everything in `docs/` directory (if it exists)
3. Find ALL `*.md` files in the repo using Glob: `**/*.md` (prefer Glob tool over shell `find`)
4. Find state-tracking files: Glob `docs/**/*.json`, and any `todo.*`, `progress.*`, `tracker.*`, `status.*` files
5. Check for other agent config files (`.cursor/rules`, `.github/copilot-instructions.md`, etc.)
6. Check for agent memory/config files in `.claude/` directory (e.g., memory files, settings)

Record every file found with its line count (`wc -l`). This inventory is the basis for all analysis.

Also read `references/principles.md` from this skill for the scoring rubric.

### Step 2: Analyze each file individually

**You MUST read the actual content of every documentation file found in Step 1.** Listing file names and line counts is not analysis — you cannot identify issues in a file you haven't read. For each file, use the Read tool to read its full content before evaluating it.

**Batching for efficiency**: If there are many files, read them in parallel batches (use multiple Read tool calls in one message). For files over 500 lines, read the first 300 lines and last 100 lines to get a representative sample, then read the middle if issues are found.

**No file cap**: Analyze ALL documentation files, regardless of how many there are or how deep in the directory tree they sit. A 600-line file buried in `src/utils/docs/guide.md` needs the same analysis as `CLAUDE.md` at the root. If there are more than 30 files, process them in batches but do NOT skip any.

For every documentation file found, answer these questions:

#### 2a. Content analysis

| Question | Action if YES |
|---|---|
| Is this file > 300 lines? | Must split — identify which sections can be extracted to `docs/` |
| Does it duplicate content from another file? | Mark for consolidation — keep one source of truth |
| Does it contain mutable state in Markdown? (task lists, status tracking, progress, feature lists with checkboxes) | Mark for JSON conversion — agents corrupt Markdown state |
| Does it contain stale information? (outdated versions, deprecated APIs, wrong commands) | Mark for update or deletion |
| Does it contain advisory language? (consider, try to, you can, should, 建议, 考虑, 尽量) | Mark for rewrite to imperative |
| Is it unreachable? (no pointer from AGENTS.md or any other doc) | Mark — needs pointer or deletion |
| Does it mix concerns? (architecture + conventions + commands in one file) | Mark for topic-based split |

#### 2b. Structure analysis

| Question | Action if YES |
|---|---|
| Are headers inconsistent or missing? | Standardize to match templates in `assets/` |
| Are code examples missing for commands/tools? | Add exact invocation syntax |
| Are error scenarios undocumented? | Add Symptom → Cause → Fix entries |
| Is the file missing a freshness marker? | Note for Step 6 (freshness markers are added AFTER all substantive remediation) |

Output a per-file findings table:

```json
{
  "file_analysis": [
    {
      "path": "CLAUDE.md",
      "lines": 340,
      "issues": [
        {"type": "oversized", "detail": "340 lines, limit is 200", "action": "split"},
        {"type": "advisory_language", "detail": "12 instances of 'consider', 'should'", "action": "rewrite"},
        {"type": "mixed_concerns", "detail": "architecture + conventions + commands in one file", "action": "split_by_topic"}
      ]
    },
    {
      "path": "todo.md",
      "lines": 45,
      "issues": [
        {"type": "mutable_state_in_markdown", "detail": "Task list with checkboxes, status tracking", "action": "convert_to_json"}
      ]
    },
    {
      "path": "docs/setup.md",
      "lines": 80,
      "issues": [
        {"type": "stale", "detail": "References Node 16, project uses Node 20", "action": "update"},
        {"type": "unreachable", "detail": "No pointer from CLAUDE.md", "action": "add_pointer"}
      ]
    }
  ]
}
```

### Step 3: Score (after analysis, not before)

Score AFTER the per-file analysis, so the scores reflect real findings — not impressions.

Evaluate against 8 dimensions (each scored 0-3):

| Dimension | Principle | What 3 looks like |
|---|---|---|
| **Conciseness & structure** | P1 | AGENTS.md ~100 lines, points to organized `docs/` with clear topics, no redundancy |
| **Progressive disclosure** | P2 | Information layered: index → topic docs → deep references |
| **Machine readability** | P3 | JSON for mutable state, structured formats, no ambiguity |
| **Actionability** | P4 | Every instruction is executable, not advisory |
| **Freshness signals** | P5 | Dates, version markers, no stale content detected |
| **Feedback loop** | P6 | Evidence that docs get updated when agents fail |
| **Architecture constraints** | P7 | Dependency direction documented, mechanically enforced, error messages include fixes |
| **Observability** | P8 | Agent has documented access to logs, metrics, or traces |

```json
{
  "scores": {
    "conciseness_and_structure": { "score": 0, "reason": "" },
    "progressive_disclosure": { "score": 0, "reason": "" },
    "machine_readability": { "score": 0, "reason": "" },
    "actionability": { "score": 0, "reason": "" },
    "freshness": { "score": 0, "reason": "" },
    "feedback_loop": { "score": 0, "reason": "" },
    "architecture_constraints": { "score": 0, "reason": "" },
    "observability": { "score": 0, "reason": "" }
  },
  "total": 0,
  "top_issues": ["issue 1", "issue 2", "issue 3"]
}
```

**Scoring adjustments**: For projects with no runtime component (static sites, CLI tools, libraries), mark **Observability** as N/A and compute the total out of 21 instead of 24. Similarly, if a project has no multi-module architecture, **Architecture constraints** may be N/A.

**Minimum action threshold**: Merely adding freshness headers (`last_verified`) is NEVER sufficient remediation — regardless of score. Every issue identified in Step 2 must be addressed with the corresponding action from Step 4. Freshness markers are a Step 6 bookkeeping task, not a remediation action.

### Step 4: Remediate (with no-regression rule)

**CRITICAL: You MUST act on every issue identified in Step 2.** Adding `last_verified` timestamps is a Step 6 task — it is NOT remediation. If Step 2 found an oversized file, you must split it. If it found redundancy, you must consolidate. Skipping hard actions and only adding timestamps means the audit produced zero value.

#### 4a. Execution order (mandatory)

Process issues in this exact priority order. Do NOT skip to easier actions:

1. **Split** — resolve all oversized/mixed-concern files first (highest impact)
2. **Consolidate** — eliminate all redundancy between files
3. **Rewrite** — fix content quality issues (advisory language, vague instructions, missing context)
4. **Convert to JSON** — migrate mutable state
5. **Update** — fix stale content
6. **Connect** — link orphan files
7. **Delete** — remove obsolete files
8. **Create** — generate missing docs from templates

#### 4b. Action reference

| Action | When to use | How to execute |
|---|---|---|
| **Split** | File > 300 lines OR mixes 3+ unrelated topics | See "How to Split" below |
| **Consolidate** | Same information in 2+ files | See "How to Consolidate" below |
| **Convert to JSON** | Mutable state in Markdown (todo lists, feature trackers, progress, status boards) | Create `.json` with structured schema. Migrate all entries. Delete the `.md` version. |
| **Rewrite** | Advisory language, vague instructions, missing examples, content that violates any of the 7 writing-guide principles | Read `references/writing-guide.md`. Apply ALL 7 principles to the file: zero-context assumption, imperative voice, one fact per line, exact command format, error-messages-as-fix-guides, JSON for state, freshness markers. This is a content rewrite, not just word substitution. |
| **Update** | Stale content (wrong versions, deprecated APIs, outdated commands) | Read the actual code/configs to verify current state, then rewrite the documentation to match reality. |
| **Connect** | Orphan file with no incoming pointer | Add to AGENTS.md documentation map with one-line description and "When to read" context. |
| **Delete** | Fully obsolete, or duplicate after consolidation | Remove the file. Remove any pointers to it. |
| **Create** | Gap identified — standard doc type missing entirely | Generate from templates in `assets/`. |

#### How to Split

Splitting is not just "move text to another file." Follow these steps:

1. **Identify topics**: Read the file and list every distinct topic it covers (e.g., architecture, conventions, commands, troubleshooting). Each topic becomes a candidate output file.
2. **Map sections to topics**: For each section/paragraph, assign it to exactly one topic. If a section spans multiple topics, split the section itself.
3. **Create output files**: For each topic, create `docs/{topic}.md`. Write the content — do not just copy-paste. Reorganize and improve as you move content:
   - Add missing context (zero-context assumption)
   - Convert advisory language to imperatives
   - Add exact command syntax where missing
   - Remove duplication within the extracted content
4. **Update AGENTS.md**: Add a pointer for each new file with "When to read" guidance.
5. **Verify completeness**: Every fact from the original file must exist in exactly one output file.

#### How to Consolidate

When the same information appears in 2+ files:

1. **Compare the versions**: Read both files. Identify which version is more complete, more accurate, and better written.
2. **Merge into one**: Keep the better version. Integrate any unique information from the other version(s) into it. Do not just pick one and delete the other — you may lose unique content.
3. **Rewrite the merged result**: Apply writing-guide principles to the merged content. The merge is an opportunity to improve quality.
4. **Update all pointers**: Every reference to the deleted file(s) must point to the surviving file.
5. **Delete the redundant file(s)**: Follow the Cleanup rule below.

#### 4c. Completion gate

Before proceeding to Step 5, verify:

```
For each issue in Step 2's file_analysis:
  ✓ An action was taken (specify which action and what changed)
  ✗ NOT acceptable: "added last_verified" for an oversized/redundant file
  ✗ NOT acceptable: skipping an issue without explanation
```

If any issue from Step 2 was not addressed, go back and address it now. Do NOT proceed to Step 5 with unresolved issues.

**Cleanup rule**: When a remediation action replaces a file (Split, Convert to JSON, Consolidate), follow this exact sequence:

1. **Create** the new file(s) with all migrated content
2. **Verify** no information was lost (compare old vs new)
3. **Update all pointers** — every reference to the old file in AGENTS.md, docs/, and other files must point to the new file(s). Do this NOW, before any deletion
4. **List files to delete** — present the user with a clear summary:
   - Which files will be deleted and why
   - What replaced them (new file path)
   - Confirmation that all references have been updated (quote the updated pointers)
5. **Wait for user confirmation** — do NOT delete until the user explicitly approves (e.g., "删除", "delete", "go ahead", "确认")
6. **Delete** only after approval — run `rm` on the approved files

Example output to user:

> **Ready to clean up replaced files:**
>
> | File to delete | Replaced by | Reason |
> |---|---|---|
> | `todo.md` | `todo.json` | Mutable state converted to JSON |
> | `docs/guide.md` | `docs/architecture.md` + `docs/conventions.md` | Split by topic |
>
> All references have been updated:
> - AGENTS.md line 42: `docs/guide.md` → `docs/architecture.md` and `docs/conventions.md`
> - AGENTS.md line 58: `todo.md` → `todo.json`
>
> Confirm deletion? (These files are fully superseded and safe to remove.)

**No-regression rule**: After every Split or Consolidate action, verify:
1. No information was lost (every fact from the original exists in the result)
2. No pointer is broken (every referenced file exists)
3. No stale file remains (the original is deleted, not left alongside the replacement)
4. Discoverability is not reduced (agent can still find the information with the same or fewer hops)

If a split would make information harder to find (e.g., splitting a short, well-organized file just to hit a template), do NOT split. The goal is better agent performance, not structural purity.

### Step 5: Generate missing pieces

After remediation, check what's still missing against the standard structure (see `assets/docs-structure-template.md`). Only generate files that would actually help the agent — skip templates that don't apply to the project (e.g., don't create `docs/observability.md` for a static site with no logs).

### Step 6: Final verification and freshness markers

**This is where `last_verified` timestamps get added** — after all substantive remediation is complete. Adding timestamps is a bookkeeping step, not a remediation action.

Run all mechanical checks:
1. `wc -l` on AGENTS.md — must be ≤ 200
2. Pointer integrity — every file in the documentation map exists on disk
3. Advisory language scan — zero forbidden words in AGENTS.md and all `docs/` files
4. No orphan docs — every file in `docs/` is reachable from AGENTS.md
5. No Markdown state files — any `todo.md`, `progress.md`, `tracker.md` should have been converted to JSON
6. Freshness markers — NOW add `<!-- last_verified: DATE -->` to every doc file that was created or modified

Present a before/after summary showing:
- **Per-file diff**: For each file from Step 2's analysis, what issue was found and what action was taken. "Added last_verified" alone is NOT a valid action for oversized, redundant, or poorly-written files.
- **Score change**: Re-score against the 8 dimensions and show the delta.

### Step 7: Recommend doc-gardening (optional)

For projects with active agent usage, suggest setting up automated doc-gardening (see `references/harness-concepts.md` § "Doc-Gardening"). A scheduled agent can periodically scan for stale `last_verified` dates, broken pointers, and advisory language — turning documentation maintenance from a manual chore into an automated system. This is the mechanical enforcement of Principle 6 (Feedback Loop).

---

## Mode B: Generate

**Goal**: Create a complete Harness-compliant documentation system from scratch.

### Step 1: Gather context

Ask the user (or infer from the codebase):
1. What does the project do? (one sentence)
2. What language/framework?
3. How to build, test, and run?
4. Any architectural patterns or conventions?
5. What tools/commands does an agent need to know about?

If the user has given you access to the codebase, scan it first to pre-fill answers. Don't ask questions you can answer by reading code.

### Step 2: Generate AGENTS.md

Read `assets/agents-md-template.md` and fill it in. Target ~100 lines (matching OpenAI's practice), hard ceiling 200. If you find yourself going over 200, move content to `docs/`.

Core sections (in order):
1. **Project overview** (2-3 lines)
2. **Quick commands** (build, test, lint, run — just the commands, no explanation)
3. **Architecture pointer** → `docs/architecture.md`
4. **Conventions pointer** → `docs/conventions.md`
5. **Common mistakes** (only the top 3-5 most frequent, one line each)
6. **Documentation map** (list of files in `docs/` with one-line descriptions)

### Step 3: Generate docs/ directory

Read `assets/docs-structure-template.md` for the standard structure. Only generate files relevant to the project — not every project needs every file:

- `docs/architecture.md` — Dependency layers, module boundaries, data flow
- `docs/conventions.md` — Naming, file organization, code style rules
- `docs/commands.md` — Every tool/script available, with exact invocation syntax
- `docs/decisions.md` — Key architectural decisions and their rationale (ADR format). Skip for solo projects or early-stage projects with no meaningful ADR history.
- `docs/troubleshooting.md` — Known failure modes and their fixes (the "linter error = fix guide" principle). Skip if the project has no known failure modes yet.
- `docs/observability.md` — How agents can access and query logs, metrics, and traces to verify their own work (see `assets/observability-template.md`). Skip for static sites, CLIs, or projects with no runtime logs/metrics.

Each file should follow the writing guide in `references/writing-guide.md`.

For mature projects, also read `references/harness-concepts.md` and consider generating:
- `docs/plans/` — Directory for execution plans as first-class artifacts (see template in `references/harness-concepts.md`). Plans give agents explicit verification targets.
- `docs/quality-grades.json` — Quality grades for product domains (see template in `references/harness-concepts.md`). Skip for small projects.

### Step 4: Generate feature tracker (if applicable)

If the project has trackable features or tasks, create `docs/feature-tracker.json` using the template in `assets/feature-tracker-template.json`. Use JSON, not Markdown — agents are less likely to accidentally corrupt structured data.

### Step 5: Verify (mechanical check)

After generating all files, run these checks programmatically — do not skip this step:

1. **Line count**: Count AGENTS.md lines. Must be ≤ 200. Run: `wc -l AGENTS.md`
2. **Pointer integrity**: For every file mentioned in the Documentation Map, verify it actually exists on disk
3. **Advisory language scan**: Search AGENTS.md for forbidden words (建议, 考虑, 尽量, consider, try to, you can, should). Replace any found with imperative alternatives
4. **Freshness marker**: Confirm `last_verified` comment exists at top of AGENTS.md

If any check fails, fix it immediately before presenting results to the user.

---

## Mode C: Update (Feedback Loop)

**Goal**: Turn a specific agent failure into a permanent prevention mechanism.

This mode can be entered in two ways:
- **Explicit**: User directly asks to add a rule or encode an error
- **From Detect**: User agreed to a doc update suggestion after Detect mode identified a pattern

### Step 1: Understand the failure

If entered explicitly, ask the user:
1. What did the agent do wrong?
2. What should it have done instead?
3. Has this happened before?

If entered from Detect mode, you already have this context from the conversation. Summarize your understanding and confirm with the user before proceeding: "I understand the issue is [X], and the correct behavior should be [Y]. I'll add this to [target file]. Sound right?"

### Step 2: Classify and route

Determine where the fix belongs:

| Failure type | Target file | Example |
|---|---|---|
| Wrong command/tool usage | `docs/commands.md` | "Agent used `npm test` but should use `pnpm test`" |
| Architecture violation | `docs/architecture.md` | "Agent imported from UI layer in a service file" |
| Convention violation | `docs/conventions.md` | "Agent used camelCase for file names" |
| Repeated common mistake | `AGENTS.md` (common mistakes section) | "Agent keeps forgetting to run migrations" |
| New failure mode | `docs/troubleshooting.md` | "Build fails silently when X" |
| Tool not agent-accessible | `docs/commands.md` | "Agent didn't know how to query the database" |
| Quality regression | `docs/quality-grades.json` | "Agent shipped code below domain quality standard" |

### Step 3: Write the prevention entry

Format it as an actionable rule following `references/writing-guide.md`:

**Bad** (advisory, vague):
> Try to avoid importing service modules in UI components.

**Good** (executable, specific):
> NEVER import from `src/services/` in files under `src/ui/`. Dependencies flow: Types → Config → Repo → Service → Runtime → UI. If you need service data in UI, pass it through the Runtime layer. Violation triggers linter rule `no-cross-layer-import`.

### Step 4: Apply and verify

Insert the rule in the correct file, at the correct location. Then run the same mechanical verification as Mode B Step 5: count AGENTS.md lines (must be ≤ 200), verify pointer integrity, scan for advisory language. If adding to AGENTS.md would push it over 200 lines, move the least-critical existing entry to the appropriate `docs/` file instead.

---

## Important reminders

- Read `references/writing-guide.md` before writing any documentation content
- Read `references/principles.md` when auditing to ensure consistent scoring
- Read `references/progressive-disclosure.md` when deciding how to structure information layers
- Read `references/harness-concepts.md` for mature projects needing plans, quality grades, doc-gardening, or brownfield adaptation
- Always verify AGENTS.md stays under 200 lines after any modification (target ~100, run `wc -l` to check)
- When in doubt about where content belongs, put it in `docs/`, not in AGENTS.md
- **Detect mode is always on**: Even when the user's request isn't about documentation, stay alert for signals that documentation needs updating. The best feedback loops are the ones the user doesn't have to think about triggering
- **Harness engineering is iterative**: The best documentation systems are built from real agent failures, not from templates. Start small, encode failures, and compound over time
