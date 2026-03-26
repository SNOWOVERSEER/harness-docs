# Agent Documentation Writing Guide

This guide teaches you how to rewrite "human-readable documentation" into "documentation agents can efficiently consume." Read this file in full before writing any documentation content.

---

## Core mental model

An agent is a new employee with zero background knowledge, but one who can precisely execute clear instructions. It will not "understand" hints, but will faithfully follow rules. Based on this mental model, follow these principles when writing.

---

## Principle 1: Zero-context assumption

Assume the reader knows nothing about the project. Never write phrases like "as we usually do" or "following convention."

**Before**: Handle errors following project conventions
**After**: All error handling uses the `AppError` class exported from `src/lib/errors.ts`. Do not use native `throw new Error()`. Example: `throw new AppError('NOT_FOUND', 'User not found')`

---

## Principle 2: Imperative voice, no suggestions

Every sentence must be an executable command. Do not use vague words like "suggest", "consider", "try to", or "should."

**Forbidden words**:
- suggest / recommend
- consider
- try to
- usually / generally
- you can / you may
- should (use "MUST" or direct imperative instead)

**Before**: It is recommended to enable TypeScript strict mode
**After**: `tsconfig.json` MUST have `strict` set to `true`. Run `npx tsc --noEmit` to verify.

---

## Principle 3: One fact per line

Do not pack multiple instructions into a single sentence. Each line conveys one piece of information. This makes it easier for agents to parse, and easier to locate and update individual rules.

**Before**: Before running tests, make sure the database is migrated and the env file is copied, then run the full test suite with pytest.
**After**:
```
1. Copy environment variables: `cp .env.example .env`
2. Run database migration: `make db-migrate`
3. Run all tests: `pytest`
```

---

## Principle 4: Exact command format

Every command must be given in a complete, copy-paste-ready format. Include the working directory, required environment variables, and expected output.

**Before**: Start the service with Docker
**After**:
```
# Working directory: project root
docker compose up -d

# Verify successful startup (expected: 3 running containers):
docker compose ps
```

---

## Principle 5: Error messages are fix guides

When writing linter rules, CI checks, or "don't do X" rules in documentation, always state what to do instead and how to do it.

**Before**: Cross-layer imports are forbidden.
**After**: Cross-layer imports are forbidden. Dependency direction: Types → Config → Repo → Service → Runtime → UI. If you need Service-layer data in the UI layer, create a bridge function in the Runtime layer. Example: see `src/runtime/bridges/user-bridge.ts`.

---

## Principle 6: JSON for state, Markdown for prose

Use JSON for mutable state (feature lists, task progress, config switches). Agents are less likely to accidentally modify other fields when working with JSON.

**Use JSON for**: feature tracker, task status, config checklists, API endpoint registries
**Use Markdown for**: architecture docs, decision records, concept explanations, instruction sets

---

## Principle 7: Freshness markers

Include at the top of every documentation file:

```markdown
<!-- last_verified: 2025-01-15 -->
<!-- related_paths: src/api/, src/models/ -->
```

`last_verified` is the date the content was last confirmed accurate. `related_paths` lists code paths described by this document — when these paths undergo significant changes, the document may need updating.

---

## Format reference

| Element | Format |
|---|---|
| File paths | Wrapped in backticks: \`src/api/routes.ts\` |
| Commands | Code block + comment noting working directory |
| Environment variables | Uppercase + backticks: \`DATABASE_URL\` |
| Rules | Numbered list, one rule per line |
| State/tracking | JSON file |
| Options/decisions | Table (option + pros/cons + conclusion) |

---

## Self-check checklist

Use this checklist after writing documentation:

1. Does any part assume the reader has background knowledge? → Fill in the context
2. Are there any "suggest", "consider" type words? → Rewrite as imperatives
3. Does any line contain multiple instructions? → Split them
4. Can every command be directly copy-pasted and executed? → Add missing paths and arguments
5. Is there any "don't do X" without saying "do Y instead"? → Add the alternative
6. Is mutable state using JSON? → Convert the format
7. Does the file have `last_verified` at the top? → Add it
