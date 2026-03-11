# Code Conventions

<!-- PRIORITY: Fill incrementally — start with git conventions and testing, add the rest as patterns emerge. -->

> Style guides and patterns for {PROJECT_NAME}. Loaded every session by `/prime` to maintain consistency across all agents and sessions.
>
> **For LLM agents populating this file:** Be specific and concrete. Include actual directory paths, real command names, code examples in your chosen language/framework, and measurable quality gates. Vague conventions lead to inconsistent implementations. Every section below should have enough detail that an agent reading only this file can write code that passes review.
>
> **Defaults are provided below.** Override them to match your team's preferences. Delete sections that don't apply to your project (e.g., API Conventions for a CLI tool, or UI Conventions for a backend service).

---

## File Naming

<!-- Override these conventions to match your project's language and framework. -->

| Type | Convention | Example |
|------|------------|---------|
| Source files | `kebab-case` | `user-service.ts`, `auth_handler.py`, `order_repo.go` |
| Test files | `{source}.test.{ext}` or `test_{source}.{ext}` | `user-service.test.ts`, `test_auth_handler.py` |
| Config files | Framework convention | `tsconfig.json`, `pyproject.toml`, `Cargo.toml` |
| Types / interfaces | `PascalCase` | `UserProfile`, `OrderItem` |

---

## Directory Structure

<!-- Define your project's directory layout. The principle: group by feature/domain,
     not by technical role (unless your framework dictates otherwise).
     Delete the examples and fill in your actual structure. -->

```
{your directory structure here}
```

---

## Architectural Approach

<!-- Describe the high-level patterns your project follows.
     Examples: feature-scoped modules, clean architecture, hexagonal,
     DDD bounded contexts, microservices, layered monolith, etc. -->

{Describe your architectural approach here.}

### Module/Service Design Checklist

When creating a new module or service:

- [ ] Belongs to a single domain / bounded context
- [ ] Has a single clear responsibility
- [ ] Dependencies are injected, not imported directly
- [ ] No direct coupling to other domains (communicate via contracts)

---

## Code Style

### Imports Order

1. Standard library / language built-ins
2. External packages / third-party dependencies
3. Internal packages / project modules
4. Relative imports within the same module

### Error Handling

Use explicit error handling over silent failures. Return errors rather than throwing where the language supports it. Log at the boundary (API handler, CLI entrypoint), not deep in business logic.

<!-- Add language-specific error handling patterns and examples here:
     TypeScript: Result<T, E> pattern, typed exceptions, or try/catch boundaries
     Python: custom exception classes, raise/except at boundaries
     Go: return err, wrap with fmt.Errorf or errors.Wrap
     Rust: Result<T, E> with ? operator, custom error enums -->

---

## Git Conventions

### Branch Naming

```
{type}/{short-description}

Examples:
feat/user-auth
fix/upload-timeout
refactor/db-queries
```

### Commit Messages

Format: `type(scope): description`

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `ci`, `perf`

```
feat(auth): add JWT token refresh endpoint
fix(upload): handle timeout for files over 50MB
test(billing): add integration tests for invoice generation
```

Change the format if your team uses a different convention (e.g., Conventional Commits with footers, Angular style, or plain imperative sentences).

### Track Commits

When working on a Conductor track:
```
{type}(track-{id}/phase-{n}): {description}
```

### Worktree Workflow

Each track is implemented in an isolated git worktree. Branch naming: `ai/track-{track-id}`. Worktrees live in `.worktrees/` (git-ignored). Always merge with `--no-ff` to preserve commit history.

---

## API Conventions

<!-- Delete this section if your project has no API.
     Fill in the patterns that match your API style (REST, GraphQL, RPC, etc.). -->

### Endpoint Format

```
/api/v1/{resource}
```

### Response Format

```json
{
  "data": { ... },
  "error": null
}
```

On error:
```json
{
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required"
  }
}
```

Adjust the response envelope to match your API style. The key principle: use a consistent shape for all endpoints so clients can handle responses uniformly.

### HTTP Status Codes

| Code | Usage |
|------|-------|
| 200 | Successful read or update |
| 201 | Successful creation |
| 400 | Client error — invalid input, validation failure |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 500 | Unexpected server error (never intentional) |

---

## Testing Conventions

### Test File Location

Co-locate tests with source files by default. Override if your framework or team prefers a separate `tests/` directory.

- Unit tests: next to the source file (e.g., `user-service.test.ts` beside `user-service.ts`)
- Integration tests: `tests/integration/` or `tests/` at project root
- E2E tests: `tests/e2e/` or `e2e/` at project root

### Test Naming

Use descriptive names that read as sentences: `{unit} → {scenario} → {expected outcome}`.

```
describe("UserService")
  it("creates a user with valid input")
  it("rejects duplicate email addresses")
  it("returns null for non-existent user ID")
```

### Coverage Targets

| Type | Target |
|------|--------|
| Unit | 80% line coverage |
| Integration | Critical paths (auth, payments, data mutations) |
| E2E | Core user flows (signup, login, primary action) |

Adjust thresholds to your project. 80% is a sensible starting point — high enough to catch regressions, low enough to avoid testing trivial code.

### E2E Testing Tiers

<!-- Delete this section if your project has no E2E tests. -->

| Tier | Requirement | When |
|------|------------|------|
| Required | Must have E2E | Auth flows, payment flows, data-critical paths |
| Recommended | Should have E2E | Core user journeys, multi-step workflows |
| Optional | E2E not expected | Admin pages, settings, low-risk reads |

---

## UI Conventions

<!-- Delete this section if your project has no UI.
     Add: design system usage, theming approach, component library patterns,
     accessibility requirements. Consider adding a separate
     context/guides/frontend-conventions.md for detailed UI guidance. -->

---

## Track Completion Gates

Every track must pass these gates before being marked complete in `conductor/tracks.md`:

### Gate 1: Static Analysis
- [ ] Lint passes
- [ ] Type checking passes (if applicable)

### Gate 2: Tests
- [ ] Track-specific tests pass
- [ ] Full test suite passes (no regressions)

### Gate 3: Coverage
- [ ] Coverage thresholds met

### Gate 4: Build
- [ ] Build succeeds

### Gate 5: E2E (if applicable)
- [ ] E2E tests pass against full stack

### Gate 6: CI
- [ ] CI passes on branch before PR merge

### Gate 7: Registry
- [ ] `conductor/tracks.md` updated only after all gates pass

---

*Last updated: {DATE}*
