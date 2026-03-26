# Extended Harness Engineering Concepts

Beyond the 8 core principles in `principles.md`, the following concepts emerge from production teams using agent-first workflows. Read this file when generating or auditing documentation for mature projects.

---

## Plans as First-Class Artifacts

**Source**: OpenAI Harness team, Boris Tane (Cloudflare)

Plans are not throwaway prompts — they are versioned, co-located documentation artifacts. OpenAI's team treats plans the same as design docs: stored in the repository, indexed from AGENTS.md, and maintained over time.

Two types:
- **Ephemeral plans**: For small, well-scoped changes. A single prompt suffices. No file needed.
- **Execution plans**: For complex, multi-step work. Written as a Markdown file in `docs/plans/`, with clear acceptance criteria and decomposed subtasks.

Why this matters for agents: a plan gives the agent a verification target. Without a plan, the agent "tries to one-shot the entire project or prematurely declares victory" (Anthropic). With a plan, the agent can check its own work against explicit criteria.

**Template**:
```markdown
<!-- status: active | completed | abandoned -->
# Plan: [Feature/Task Name]

## Goal
[One sentence: what does "done" look like?]

## Acceptance Criteria
- [ ] [Specific, verifiable criterion]
- [ ] [Specific, verifiable criterion]

## Subtasks
1. [First subtask — small enough for one agent session]
2. [Second subtask]

## Constraints
- [Architectural constraint, e.g., "must not add new dependencies"]
- [Performance constraint, e.g., "page load < 2s"]

## Verification
[How the agent should verify completion — test commands, URLs to check, metrics to query]
```

---

## Tool Accessibility

**Source**: Greg Brockman, Stripe Minions team

Brockman's recommendation: "Maintain a list of tools that your team relies on, and make sure someone takes point on making it agent-accessible (such as via a CLI or MCP server)."

If agents can't access your tools, they can't help. Document tool accessibility in `docs/commands.md` with:

1. **What the tool does** (one line)
2. **How to invoke it** (exact CLI command or MCP server name)
3. **What the output looks like** (example output so the agent can parse it)
4. **Common failure modes** (what goes wrong and how to fix it)

Categories of tools to document:
- Build/test/lint commands (most projects already have this)
- Database tools (migration, seeding, querying)
- Deployment tools (deploy, rollback, status check)
- Monitoring tools (log queries, health checks, metrics)
- Browser automation (screenshot, DOM snapshot, navigation) — critical for QA
- Internal APIs and services the agent might need to call

The key insight from Stripe: agents need the **same context and tooling as human engineers**, not a bolted-on afterthought. Their agents connect to 400+ internal tools via MCP.

---

## Doc-Gardening

**Source**: OpenAI Harness team

Documentation rots. OpenAI addresses this with automated "doc-gardening" — background agents that periodically scan for stale documentation and open cleanup PRs.

Implementation pattern:
1. Every doc file has a `last_verified` date and `related_paths`
2. A scheduled agent (weekly or per-sprint) scans for:
   - `last_verified` dates older than 30 days
   - `related_paths` that have changed significantly since last verification
   - Pointers in AGENTS.md that reference non-existent files
   - Advisory language that slipped in
3. The agent opens a PR with fixes, or flags files that need human review
4. Most cleanup PRs require under-a-minute reviews

This turns documentation from "a static artifact that rots" into "a living system with automated maintenance." It is the mechanical enforcement of Principle 6 (Feedback Loop).

---

## Golden Principles and Entropy Management

**Source**: OpenAI Harness team

Full agent autonomy introduces entropy. Agents replicate existing patterns — including suboptimal ones — causing drift. OpenAI initially spent every Friday manually cleaning up "AI slop." This didn't scale.

The solution: encode **"golden principles"** — opinionated, mechanical rules that prevent entropy accumulation. Examples:
- Prefer shared utility packages over hand-rolled helpers
- Validate data at boundaries rather than probing inline
- File size limits (e.g., no single file > 500 lines)
- Schema naming conventions enforced by linter
- Structured logging requirements

On regular cadences, background agents scan for deviations, update quality grades, and open targeted refactoring PRs. This functions as **garbage collection** — technical debt treated as continuous small payments rather than compounding interest.

For documentation: encode golden principles in `docs/conventions.md` and enforce them via linters with remediation-rich error messages.

---

## Quality Grades

**Source**: OpenAI Harness team

OpenAI maintains quality grades for product domains — structured assessments of how well each area of the codebase meets standards. This gives agents (and humans) a quick read on where attention is needed.

Track in `docs/quality-grades.json`:
```json
{
  "_meta": {
    "last_updated": "YYYY-MM-DD",
    "grading_scale": "A (exemplary) | B (solid) | C (needs work) | D (significant issues)"
  },
  "domains": [
    {
      "name": "Authentication",
      "grade": "B",
      "notes": "Good coverage, needs better error messages",
      "last_reviewed": "YYYY-MM-DD"
    }
  ]
}
```

Quality grades should be:
- Updated when a domain undergoes significant changes
- Referenced during audit (Mode A) to prioritize remediation
- Maintained by doc-gardening agents

---

## Brownfield Project Adaptation

**Source**: Birgitta Boeckeler (Martin Fowler site), ignorance.ai playbook

All published Harness success stories involve greenfield projects or teams that built harnesses from scratch. Applying these techniques to existing codebases is explicitly acknowledged as an open problem.

The retrofit challenge: running a Harness audit on a ten-year-old codebase with no architectural constraints, inconsistent testing, and patchy documentation produces overwhelming results. You'd "drown in alerts."

Recommended approach for brownfield projects:
1. **Start with AGENTS.md only** — don't try to generate the full `docs/` structure on day one
2. **Focus on the feedback loop first** — each time the agent makes a mistake, add one rule. This builds the documentation organically from actual failures, not from templates
3. **Adopt incrementally by domain** — pick one module/domain, apply full Harness treatment, prove the value, expand
4. **Don't fight existing conventions** — if the project uses a different structure, adapt the Harness patterns to fit, not the other way around
5. **Set realistic scoring expectations** — a brownfield project scoring 12/24 is normal and not a failure. Track improvement over time, not absolute score

---

## Custom Lint Errors as Teaching

**Source**: OpenAI Harness team

The cleverest idea from the OpenAI writeup: custom linter error messages that double as remediation instructions.

When an agent violates an architectural constraint, the error message doesn't just flag the violation — it tells the agent how to fix it. The tooling **teaches** the agent while it works.

Example:
```
ERROR: Cross-layer import detected.
  File: src/ui/Dashboard.tsx
  Import: src/services/UserService

  FIX: UI layer cannot import from Services directly.
  Dependency direction: Types → Config → Repo → Service → Runtime → UI.
  Instead, create a bridge function in src/runtime/bridges/ that
  wraps the service call. See src/runtime/bridges/auth-bridge.ts
  for an example.
```

This pattern should be documented in `docs/architecture.md` alongside the dependency rules. When the linter teaches the agent, you need fewer rules in AGENTS.md.
