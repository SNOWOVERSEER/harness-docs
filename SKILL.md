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

This skill implements the core insight from OpenAI's Codex team: **the agent instruction file should be a directory, not an encyclopedia**. A concise index (under ~200 lines) that points to deeper, structured documentation in `docs/`, with JSON for anything that needs to be tracked as state.

### Agent instruction file naming

Different tools use different file names for the same concept. This skill supports all of them:

| Tool | File name |
|---|---|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex / generic | `AGENTS.md` |
| Cursor | `.cursorrules` or `.cursor/rules` |
| GitHub Copilot | `.github/copilot-instructions.md` |

**Auto-detection**: At the start of any mode, check which file already exists in the project root. Use that name throughout. If none exists, ask the user which tool they primarily use, or default to `CLAUDE.md` for Claude Code users and `AGENTS.md` otherwise. In the rest of this skill, "AGENTS.md" refers to whichever file name is in use for the project.

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

**Goal**: Scan the entire codebase's documentation, score it against Harness principles, and produce a concrete remediation plan with ready-to-apply changes.

### Step 1: Scan

Detect the agent instruction file (see "Agent instruction file naming" above), then read these (in order, skip if not found):
- The agent instruction file (`CLAUDE.md` / `AGENTS.md` / etc.) at repo root
- `docs/` directory listing
- Any other `*.md` files in the repo root
- Any additional agent config files (`.cursor/rules`, `.github/copilot-instructions.md`, etc.)

Also read `references/principles.md` from this skill for the scoring rubric.

### Step 2: Score

Evaluate against these 8 dimensions (each scored 0-3):

| Dimension | What 3 looks like |
|---|---|
| **Conciseness** | AGENTS.md ≤ 200 lines, no redundancy |
| **Directory structure** | AGENTS.md points to organized `docs/` with clear topics |
| **Progressive disclosure** | Information layered: index → topic docs → deep references |
| **Machine readability** | Structured formats (JSON for state, clear headers, no ambiguity) |
| **Actionability** | Every instruction is executable, not advisory ("run X" not "consider X") |
| **Freshness signals** | Dates, version markers, or mechanisms to detect staleness |
| **Feedback loop** | Evidence that docs get updated when agents fail |
| **Observability** | Agent has documented access to logs, metrics, or traces for self-verification |

Total: 0-24. Output the score as a JSON block for consistency:

```json
{
  "scores": {
    "conciseness": { "score": 0, "reason": "" },
    "directory_structure": { "score": 0, "reason": "" },
    "progressive_disclosure": { "score": 0, "reason": "" },
    "machine_readability": { "score": 0, "reason": "" },
    "actionability": { "score": 0, "reason": "" },
    "freshness": { "score": 0, "reason": "" },
    "feedback_loop": { "score": 0, "reason": "" },
    "observability": { "score": 0, "reason": "" }
  },
  "total": 0,
  "top_issues": ["issue 1", "issue 2", "issue 3"]
}
```

Then provide a prioritized remediation plan.

### Step 3: Remediate

For each issue found, produce one of:
- **Rewrite**: Full replacement text for the file, with diff markers
- **Split**: Instructions to extract sections from a bloated file into `docs/`
- **Create**: New file content for missing documentation
- **Delete**: Files or sections that are outdated/harmful

Present as a structured execution plan the user can approve before applying. Then apply all approved changes in one pass.

### Step 4: Generate missing pieces

After fixing existing docs, check what's still missing against the standard structure (see `assets/docs-structure-template.md`). Offer to generate any missing files.

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

Read `assets/agents-md-template.md` and fill it in. Keep the result under 200 lines — this is the hard ceiling. Aim for 80-150 lines for most projects. If you find yourself going over 200, move content to `docs/`.

Core sections (in order):
1. **Project overview** (2-3 lines)
2. **Quick commands** (build, test, lint, run — just the commands, no explanation)
3. **Architecture pointer** → `docs/architecture.md`
4. **Conventions pointer** → `docs/conventions.md`
5. **Common mistakes** (only the top 3-5 most frequent, one line each)
6. **Documentation map** (list of files in `docs/` with one-line descriptions)

### Step 3: Generate docs/ directory

Read `assets/docs-structure-template.md` for the standard structure. Generate these files:

- `docs/architecture.md` — Dependency layers, module boundaries, data flow
- `docs/conventions.md` — Naming, file organization, code style rules
- `docs/commands.md` — Every tool/script available, with exact invocation syntax
- `docs/decisions.md` — Key architectural decisions and their rationale (ADR format)
- `docs/troubleshooting.md` — Known failure modes and their fixes (the "linter error = fix guide" principle)
- `docs/observability.md` — How agents can access and query logs, metrics, and traces to verify their own work (see `assets/observability-template.md`)

Each file should follow the writing guide in `references/writing-guide.md`.

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

### Step 3: Write the prevention entry

Format it as an actionable rule following `references/writing-guide.md`:

**Bad** (advisory, vague):
> Try to avoid importing service modules in UI components.

**Good** (executable, specific):
> NEVER import from `src/services/` in files under `src/ui/`. Dependencies flow: Types → Config → Repo → Service → Runtime → UI. If you need service data in UI, pass it through the Runtime layer. Violation triggers linter rule `no-cross-layer-import`.

### Step 4: Apply and verify

Insert the rule in the correct file, at the correct location. Then run the same mechanical verification as Mode B Step 5: count AGENTS.md lines (must be ≤ 200), verify pointer integrity, scan for advisory language. If adding to AGENTS.md would push it over 200 lines, move the least-critical existing entry to the appropriate `docs/` file instead.

---

## Writing principles (summary)

These are summarized from `references/writing-guide.md` — read the full version for details.

1. **Zero-context assumption**: Write as if the reader knows nothing about the project
2. **Imperative voice**: "Run `make build`" not "You can build by running make"
3. **One fact per line**: No compound sentences mixing different instructions
4. **Executable over advisory**: Every statement should be something an agent can act on
5. **Error messages are documentation**: When writing linter rules or checks, the error message itself should contain the fix
6. **JSON for state, Markdown for prose**: Never track mutable state (feature lists, progress, status) in Markdown

---

## Important reminders

- Read `references/writing-guide.md` before writing any documentation content
- Read `references/principles.md` when auditing to ensure consistent scoring
- Read `references/progressive-disclosure.md` when deciding how to structure information layers
- Always verify AGENTS.md stays under 200 lines after any modification (run `wc -l` to check)
- When in doubt about where content belongs, put it in `docs/`, not in AGENTS.md
- **Detect mode is always on**: Even when the user's request isn't about documentation, stay alert for signals that documentation needs updating. The best feedback loops are the ones the user doesn't have to think about triggering
