# Code Conventions

> Style guides and patterns for {PROJECT_NAME}. Loaded every session by `/prime` to maintain consistency across all agents and sessions.
>
> **For LLM agents populating this file:** Be specific and concrete. Include actual directory paths, real command names, code examples in your chosen language/framework, and measurable quality gates. Vague conventions lead to inconsistent implementations. Every section below should have enough detail that an agent reading only this file can write code that passes review.

---

## File Naming

<!-- Define your naming conventions per file type.
     Include the actual extensions and patterns used in your project. -->

| Type | Convention | Example |
|------|------------|---------|
| Components | {convention} | {example} |
| Pages/Views | {convention} | {example} |
| Hooks/Composables | {convention} | {example} |
| Utilities | {convention} | {example} |
| Services | {convention} | {example} |
| Types/Interfaces | {convention} | {example} |
| Tests | {convention} | {example} |
| Config | {convention} | {example} |

---

## Directory Structure

<!-- Define your project's directory layout. Include both frontend and backend
     if applicable. Be specific — agents use these paths for file ownership. -->

### Frontend

```
{your frontend directory structure}
```

### Backend

```
{your backend directory structure}
```

### Shared Packages (if monorepo)

```
{your shared packages structure}
```

---

## Architectural Principles

<!-- Describe the high-level architecture patterns your project follows.
     Examples: DDD bounded contexts, clean architecture, feature-scoped modules,
     hexagonal architecture, microservices, etc.
     Include a checklist agents can reference when creating new services/modules. -->

{Describe your architectural approach here.}

### Service/Module Design Checklist

When creating a new service or module:

- [ ] {Principle 1 — e.g., belongs to one bounded context}
- [ ] {Principle 2 — e.g., single responsibility}
- [ ] {Principle 3 — e.g., dependencies injected via interfaces}
- [ ] {Principle 4 — e.g., no cross-context coupling}

---

## Code Style

<!-- Define import ordering, function style, type conventions, error handling patterns.
     Include code examples in your project's language(s). -->

### Imports Order

1. {Category 1 — e.g., framework/standard library}
2. {Category 2 — e.g., external packages}
3. {Category 3 — e.g., internal packages}
4. {Category 4 — e.g., relative imports}

### Type Conventions

<!-- How you define types, interfaces, enums, validation schemas, etc. -->

### Error Handling

<!-- Frontend and backend error handling patterns. Include code examples. -->

---

## Git Conventions

### Branch Naming

```
{your branch naming pattern}

Examples:
{example-1}
{example-2}
```

### Commit Messages

Format: `{your commit format}`

Types: {your commit types}

```
{example commits}
```

### Track Commits

When working on a Conductor track:
```
{type}(track-{id}/phase-{n}): {description}
```

### Worktree Workflow

Each track is implemented in an isolated git worktree. Branch naming: `ai/track-{track-id}`. Worktrees live in `.worktrees/` (git-ignored). Always merge with `--no-ff` to preserve commit history.

---

## API Conventions

<!-- Define your API patterns: REST, GraphQL, RPC, etc.
     Include endpoint format, response envelope, status codes, pagination. -->

### Endpoint Format

```
{your endpoint pattern — e.g., /api/v1/{resource}}
```

### Response Format (Success)

```json
{your success response shape}
```

### Response Format (Error)

```json
{your error response shape}
```

### HTTP Status Codes

| Code | Usage |
|------|-------|
| 200 | {when to use} |
| 201 | {when to use} |
| 400 | {when to use} |
| 401 | {when to use} |
| 403 | {when to use} |
| 404 | {when to use} |
| 500 | {when to use} |

---

## Testing Conventions

### Test File Location

<!-- Where unit, integration, and E2E tests live. -->

- Unit tests: {location — e.g., next to source file}
- Integration tests: {location}
- E2E tests: {location}

### Test Naming

<!-- Your describe/it naming pattern with an example. -->

### Coverage Targets

| Type | Target |
|------|--------|
| Unit | {percentage} |
| Component | {percentage} |
| Integration | {percentage} |
| E2E | {strategy — e.g., key user flows} |

### E2E Testing Strategy

<!-- Define which features need E2E tests and your tier system.
     Include quality standards: banned patterns, required assertions, etc. -->

| Tier | Requirement | Rationale |
|------|------------|-----------|
| Required | Must have E2E | {when} |
| Recommended | Should have E2E | {when} |
| Optional | E2E not expected | {when} |

---

## Component/UI Conventions

<!-- If your project has a frontend: design system usage, theming approach,
     component library patterns. Skip this section for backend-only projects. -->

---

## Track Completion Gates

Every track must pass these gates before being marked complete in `conductor/tracks.md`:

### Gate 1: Static Analysis
- [ ] Lint passes (`{your lint command}`)
- [ ] Type checking passes (`{your typecheck command}`)

### Gate 2: Tests
- [ ] Track-specific tests pass
- [ ] Full test suite passes (no regressions)

### Gate 3: Coverage
- [ ] Coverage thresholds met (`{your coverage command}`)

### Gate 4: Build
- [ ] Build succeeds (`{your build command}`)

### Gate 5: E2E (if applicable)
- [ ] E2E tests pass against full stack
- [ ] E2E quality standards met (no banned patterns)

### Gate 6: CI
- [ ] CI passes on branch before PR merge

### Gate 7: Registry
- [ ] `conductor/tracks.md` updated only after all gates pass

---

*Last updated: {DATE}*
