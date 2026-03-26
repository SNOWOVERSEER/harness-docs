# Eight Harness Engineering Principles

This file is the basis for audit scoring. Each principle corresponds to a scoring dimension (0-3 points).

For extended Harness concepts beyond these core principles (plans, tool accessibility, doc-gardening, entropy management, brownfield adaptation), see `references/harness-concepts.md`.

---

## Principle 1: AGENTS.md is a directory, not an encyclopedia

**Source**: OpenAI Codex team

A monolithic instruction file rots quickly. Agents cannot tell which rules are still valid, and humans stop maintaining it.

**Scoring criteria**:
- 3: AGENTS.md ~100 lines, contains only an index, key commands, and the most critical rules; points to organized `docs/` with clear topics
- 2: AGENTS.md 100-200 lines, good structure but could be tighter; has `docs/` directory
- 1: AGENTS.md 200-400 lines, contains too many specific rules; partial `docs/` usage
- 0: AGENTS.md > 400 lines, or does not exist

**Remediation**: Move specific rules into the `docs/` directory. AGENTS.md retains only pointers and high-frequency information.

---

## Principle 2: Progressive Disclosure

**Source**: OpenAI Harness Post, Anthropic Skill architecture

An agent's context window is a scarce resource. Information must be layered:

- Layer 1: AGENTS.md (index + most critical rules, ~100 lines)
- Layer 2: Topic docs under docs/ (loaded on demand)
- Layer 3: Deep reference materials (loaded only for specific tasks)

**Scoring criteria**:
- 3: Clear three-layer structure, AGENTS.md has pointers to each layer
- 2: Two layers (AGENTS.md + some docs), but not systematic
- 1: All information in one file, but file is well-structured
- 0: Information scattered everywhere, no structure

---

## Principle 3: Machine readability first

**Source**: Anthropic engineering team, LangChain

Documentation for agents differs fundamentally from documentation for humans:

- Track state with JSON, not Markdown checklists (agents are less likely to accidentally overwrite structured data)
- Use exact invocation syntax for commands, not natural language descriptions
- Embed fix guides in error messages (linter error = fix guide)
- Prefer API/CLI interfaces over dashboard/visualization descriptions

**Scoring criteria**:
- 3: State tracked in JSON, commands have exact syntax, error messages include fixes
- 2: Partially structured, but some commands or state still described in natural language
- 1: Almost entirely Markdown narrative
- 0: No structured content

---

## Principle 4: Actionability

**Source**: Mitchell Hashimoto, Boris Tane

Every instruction must be a directly executable action, not advice or background knowledge.

**Bad**: "It is recommended to follow REST conventions when designing APIs"
**Good**: "All API endpoints MUST use the `/api/v1/` prefix. Run `make lint-api` to check compliance when creating new endpoints."

**Scoring criteria**:
- 3: 90%+ of instructions are executable (with specific commands, paths, formats)
- 2: 60-90% executable
- 1: 30-60% executable, heavy use of "suggest", "consider" language
- 0: Primarily descriptive/explanatory content

---

## Principle 5: Freshness signals

**Source**: OpenAI Codex team

Stale documentation is more dangerous than no documentation — agents will follow outdated rules, producing bugs that are hard to trace.

**Mechanisms**:
- `last_verified` date at the top of each file
- Version numbers or associated commit hashes
- Background agents that periodically scan for stale docs and open cleanup PRs
- Rules linked to specific code paths (path disappears = rule may be stale)

**Scoring criteria**:
- 3: Date/version markers present, with a mechanism to detect stale content
- 2: Date markers present but no automated detection
- 1: Some files have dates
- 0: No freshness signals at all

---

## Principle 6: Feedback loop

**Source**: Greg Brockman, Mitchell Hashimoto

Documentation is a feedback loop, not a static artifact. Every time an agent makes a mistake, the documentation should be updated to prevent recurrence.

Key question: Is there evidence in the project that documentation was iteratively refined through actual agent usage?

**Scoring criteria**:
- 3: Clear error-to-doc-update workflow, visible history
- 2: Documentation occasionally updated due to agent errors, but no systematic process
- 1: Documentation occasionally updated but not driven by agent feedback
- 0: Documentation has not been touched since initial creation

---

## Principle 7: Architecture constraint documentation

**Source**: OpenAI Codex team

Architecture constraints (dependency direction, module boundaries, forbidden imports) must be explicitly documented and enforced through mechanical means (linters, structural tests).

Architecture rules in documentation should:
- Declare dependency direction (e.g., Types → Config → Repo → Service → Runtime → UI)
- List forbidden dependencies
- Link to specific linter rules or tests
- Include fix guidance in violation error messages

**Scoring criteria**:
- 3: Architecture constraints documented + mechanically enforced + error messages include fixes
- 2: Architecture constraints documented but no mechanical enforcement
- 1: Architecture implied in code structure, no explicit documentation
- 0: No architecture constraint documentation

---

## Principle 8: Observability

**Source**: OpenAI Codex team, Harrison Chase (LangChain)

Harrison Chase's core insight: **"In traditional software, code documents what the app does; in AI, traces document what the app does."** An agent's actual decisions happen inside the model runtime — code is just scaffolding. If an agent cannot query logs, metrics, or traces, it is flying blind — guessing failure points and proposing changes based on incomplete information.

OpenAI equips each Codex agent worktree with an independent observability pipeline (logs via LogQL, metrics via PromQL, distributed traces), queryable through a DSL. This makes instructions like "ensure service startup completes within 800ms" executable — the agent uses runtime evidence to verify its own work.

**Key shift**: The primary consumer of observability interfaces has shifted from humans to agents. Dashboards and charts are not enough — agents need structured data returned via API/CLI. Machine readability, structured output, and composability are first-class citizens.

**Scoring criteria**:
- 3: Documentation includes log/metrics/trace query commands usable by agents; agent can self-verify with runtime data
- 2: Partial observability documentation (e.g., log paths), but agents cannot query directly
- 1: Human-facing dashboards/monitoring exist, but no machine-readable interface for agents
- 0: No observability documentation; agent has zero access to runtime feedback
