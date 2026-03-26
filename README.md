# harness-docs

An agent skill that builds and maintains documentation systems optimized for AI agents, based on [OpenAI's Harness Engineering](https://openai.com/index/harness-engineering/) principles.

## What it does

Most project docs are written for humans with context. Agents don't have that luxury. This skill gives agents the ability to **audit**, **generate**, and **maintain** documentation that follows Harness Engineering best practices.

### Four modes

- **Audit** — Read and analyze every documentation file (regardless of directory depth), score against 8 Harness dimensions, then remediate: split oversized files, consolidate redundancy, rewrite content for agent readability, and verify all issues are resolved before completion
- **Generate** — Create a complete AGENTS.md/CLAUDE.md + `docs/` directory from scratch
- **Update** — Encode a specific agent failure into documentation to prevent recurrence
- **Detect** — Implicitly recognize when user complaints or repeated mistakes signal a documentation gap, then proactively suggest updates

### Key principles implemented


| Harness Principle                               | How                                                                                      |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Instruction file as directory, not encyclopedia | ~100 line target with mechanical `wc -l` verification (200 ceiling)                       |
| Progressive disclosure                          | Three-layer info architecture (index → topic docs → deep refs)                           |
| Documentation as feedback loop                  | Detect mode + Update mode close the loop automatically                                   |
| Structured formats over Markdown                | JSON for state tracking, structured audit scoring                                        |
| Actionable over advisory                        | Forbidden word scanning, imperative voice enforcement                                    |
| Error messages = fix guides                     | Troubleshooting template: Symptom → Cause → Fix → Prevention                             |
| Architecture constraints                        | Dependency direction docs + module boundary tables                                       |
| Observability                                   | `docs/observability.md` with log/metric/trace query commands for agent self-verification |


### Multi-tool support

Auto-detects and works with `CLAUDE.md` (Claude Code), `AGENTS.md` (OpenAI Codex), `.cursorrules` (Cursor), and `.github/copilot-instructions.md` (GitHub Copilot).

## Installation

### Quick install (recommended)

```bash
npx skills add SNOWOVERSEER/harness-docs
```

### Other methods

```bash
# Clone directly
git clone https://github.com/SNOWOVERSEER/harness-docs.git ~/.claude/skills/harness-docs
```

### As a standalone reference

The files in `references/` and `assets/` are useful on their own as templates and writing guides for agent documentation.

## File structure

```
harness-docs/
├── SKILL.md                              # Main skill instructions
├── references/
│   ├── principles.md                     # 8 Harness principles with scoring rubrics
│   ├── writing-guide.md                  # How to write for agents (not humans)
│   ├── progressive-disclosure.md         # Three-layer info architecture guide
│   └── harness-concepts.md              # Extended concepts: plans, tools, doc-gardening, brownfield
└── assets/
    ├── agents-md-template.md             # AGENTS.md / CLAUDE.md template
    ├── docs-structure-template.md        # Standard docs/ directory structure
    ├── feature-tracker-template.json     # JSON feature tracking template
    └── observability-template.md         # docs/observability.md template
```

## Eval results

Tested across 10 scenarios (3 iterations), including ambiguous requests, mixed modes, non-code projects, and implicit detection:


| Configuration       | Pass Rate                   |
| ------------------- | --------------------------- |
| With skill          | **100%** (38/38 assertions) |
| Baseline (no skill) | 55% (24/44 assertions)      |


## Credits

Based on ideas from:

- [Harness Engineering](https://openai.com/index/harness-engineering/) — OpenAI
- [My AI Adoption Journey](https://mitchellh.com/writing/my-ai-adoption-journey) — Mitchell Hashimoto
- [The Emerging Harness Engineering Playbook](https://www.ignorance.ai/p/the-emerging-harness-engineering) — ignorance.ai
- [Self-Improving Agents](https://arize.com/blog/closing-the-loop-coding-agents-telemetry-and-the-path-to-self-improving-software/) — Arize AI

## License

MIT