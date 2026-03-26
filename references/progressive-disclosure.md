# Progressive Disclosure

An agent's context window is its most valuable resource. Dumping all information at once is the same as having no focus. The core idea of progressive disclosure: load on demand, go deeper layer by layer.

---

## Three-layer architecture

### Layer 1: AGENTS.md (always in context)

**Budget**: ~100 lines (matching OpenAI's practice, hard ceiling 200 lines)
**Content**: Project identity, key commands, architecture overview pointer, top 5-10 common mistakes, documentation map
**Analogy**: A book's table of contents + quick reference card

The agent reads this file at every startup. Every line must be high-frequency information. Pure background knowledge and low-frequency rules go in docs/.

**Decision criterion**: If a rule is not "something most tasks need to know," it does not belong in AGENTS.md.

### Layer 2: docs/ topic documents (loaded on demand)

**Budget**: 100-300 lines per file
**Content**: Complete rule set for a given topic (architecture, conventions, commands, troubleshooting)
**Analogy**: Book chapters

The agent reads the relevant file when executing a specific type of task. Pointers in AGENTS.md tell it when to read which file.

**Pointer format**:
```markdown
## Architecture
→ `docs/architecture.md` — Module boundaries, dependency direction, data flow
  When to read: creating new modules, modifying cross-module calls, adding dependencies
```

### Layer 3: Deep references (loaded only in specific scenarios)

**Budget**: Unlimited
**Content**: API docs, schema definitions, historical decision records, large config examples
**Analogy**: Appendices and bibliography

Linked from Layer 2 documents. The agent loads these only when it needs specific details.

---

## How to decide which layer information belongs in

```
Q: Is this information needed in 80%+ of tasks?
  Yes → Layer 1 (AGENTS.md)
  No ↓

Q: Is this information frequently needed for a certain type of task?
  Yes → Layer 2 (docs/topic.md)
  No ↓

→ Layer 3 (docs/references/ or specific reference files)
```

---

## How to write pointers

A good pointer is not just a file path — it also tells the agent **when** to read that file.

**Bad pointer**:
```
See docs/architecture.md for details
```

**Good pointer**:
```
## Architecture
→ `docs/architecture.md`
When you need to: create new modules, modify cross-module calls, add dependencies
Contains: module boundary diagram, dependency direction rules, data flow description
```

---

## Documentation map template

AGENTS.md should end with a complete documentation map:

```markdown
## Documentation Map

| File | When to read | Contains |
|---|---|---|
| `docs/architecture.md` | Creating/modifying module structure | Dependency direction, module boundaries, data flow |
| `docs/conventions.md` | Writing new code | Naming conventions, file organization, code style |
| `docs/commands.md` | Running build/test/deploy commands | All available commands with exact syntax |
| `docs/decisions.md` | Understanding design rationale | Architecture Decision Records (ADRs) |
| `docs/troubleshooting.md` | Encountering errors or build failures | Known issues and fix steps |
| `docs/observability.md` | Verifying changes work in production | Log/metric/trace query commands and self-verification checklists |
| `docs/feature-tracker.json` | Checking feature status/progress | Feature list and current state (JSON) |
```

---

## Anti-patterns

1. **All API endpoint docs crammed into AGENTS.md** → Move to `docs/api-reference.md`
2. **Only one giant README.md in docs/** → Split by topic
3. **Pointers without read-timing guidance** → Add "When to read" descriptions
4. **Deep documents not linked from shallow layers** → Agent will never find them
5. **AGENTS.md contains conditional logic (if/else)** → Simplify; move conditional branches to the relevant topic doc
