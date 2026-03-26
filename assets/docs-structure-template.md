# docs/ Directory Standard Structure

## Standard files

```
docs/
├── architecture.md        — Module boundaries, dependency direction, data flow
├── conventions.md         — Naming conventions, file organization, code style
├── commands.md            — Exact invocation syntax for all available tools/scripts
├── decisions.md           — Architecture Decision Records (ADR format)
├── troubleshooting.md     — Known failure modes and fix steps
├── observability.md       — Log/metric/trace query methods and self-verification checklists
└── feature-tracker.json   — Feature tracking (JSON, not Markdown)
```

## Optional extensions (add as needed for larger projects)

```
docs/
├── api-reference.md       — API endpoint list with request/response formats
├── deployment.md          — Deployment process and environment configuration
├── testing-guide.md       — Testing strategy, fixture docs, mock conventions
├── migration-guide.md     — Database/API version migration steps
└── references/            — Deep reference materials directory
    ├── schema-definitions.md
    └── third-party-apis.md
```

---

## Structure template for each file

### architecture.md

```markdown
<!-- last_verified: YYYY-MM-DD -->
<!-- related_paths: src/ -->

# Architecture

## Module Map

[List all top-level modules and their responsibilities, 1-2 lines each]

## Dependency Direction

[Declare dependency flow rules, e.g.:]
Types → Config → Repo → Service → Runtime → UI

Reverse imports are forbidden. Violation detection: `[lint command]`

## Data Flow

[Describe how data flows through modules from input to output]

## Module Boundaries

| Module | Responsibility | Allowed imports | Forbidden imports |
|---|---|---|---|
| types/ | Type definitions | (none) | everything else |
| service/ | Business logic | types, config, repo | runtime, ui |
```

### conventions.md

```markdown
<!-- last_verified: YYYY-MM-DD -->
<!-- related_paths: src/ -->

# Conventions

## File Naming
[Exact rules + examples]

## Function/Variable Naming
[Exact rules + examples]

## File Organization
[Directory structure rules]

## Import Order
[Import ordering rules + auto-format command]

## Error Handling
[Unified error handling pattern + example code]
```

### commands.md

```markdown
<!-- last_verified: YYYY-MM-DD -->

# Commands

Each command includes: exact invocation syntax, working directory, expected output, common failure causes.

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
- **Context**: [Why this decision was needed, 1-2 sentences]
- **Decision**: [What was decided]
- **Consequences**: [Impact of this decision]
- **Alternatives considered**: [Other options and why they were rejected]
```

### troubleshooting.md

```markdown
<!-- last_verified: YYYY-MM-DD -->

# Troubleshooting

Each entry includes: symptom, cause, fix steps. Format follows the "error message = fix guide" principle.

## [Error symptom or message]
**Cause**: [root cause]
**Fix**:
\`\`\`bash
[exact fix commands]
\`\`\`
**Prevention**: [how to avoid this in future]
```
