# Architectural Principles

<!-- PRIORITY: Fill after 2-3 tracks — principles emerge from real implementation experience. -->

> Immutable development principles for {PROJECT_NAME}. These act as governance gates — every spec and plan is validated against them by `/review-specs`. Violations are flagged as **CRITICAL** and block implementation until resolved or explicitly justified.
>
> **This file must be maintained and updated** as the project evolves. Add new principles when the team discovers recurring quality issues. Amend existing principles when they no longer serve the project. Every change should be reviewed and merged via PR, just like code.
>
> **For LLM agents populating this file:** Each principle must have a clear MUST or SHOULD normative statement, a rationale, and at least one concrete gate check. Vague principles like "code should be clean" are useless — make them specific and testable.

---

## How Principles Work

1. **`/accept`** generates `spec.md` and `plan.md` — principles are not enforced yet.
2. **`/review-specs`** loads this file and checks every requirement and plan element against each MUST principle. Violations become CRITICAL findings.
3. **`/implement-track`** checks for a passing `review.md` before starting — so principles are enforced before any code is written.

Principles use two normative levels:

| Level | Meaning | Review Behavior |
|-------|---------|-----------------|
| **MUST** | Non-negotiable. Violation blocks implementation. | CRITICAL finding in `/review-specs` |
| **SHOULD** | Strongly recommended. Deviation requires documented justification. | HIGH finding in `/review-specs` |

---

## Principle Template

Use this structure for each principle. Copy it when adding new ones.

```markdown
### Principle {N}: {Short Name}

**Level:** MUST | SHOULD

**Statement:** {One sentence describing what must or should be true.}

**Rationale:** {Why this principle exists — what problem it prevents or what quality it ensures.}

**Gate Checks:**

- [ ] {Concrete, verifiable check that /review-specs can evaluate against spec.md or plan.md}
- [ ] {Another check if needed}

**Exceptions:** {When this principle does not apply, or "None."}
```

---

## Principles

<!--
  ADD YOUR PROJECT'S PRINCIPLES BELOW.

  Start with 3-5 principles that reflect your team's strongest opinions
  about how software should be built. Common categories:

  - Simplicity & complexity management
  - Testing strategy (TDD, integration-first, etc.)
  - Security posture (auth, data protection, input validation)
  - Data model integrity (single source of truth, no duplication)
  - API design (contract-first, versioning, backwards compatibility)
  - Dependency management (minimal deps, no wrappers, framework-native)
  - Observability (logging, metrics, tracing requirements)
  - Performance (latency budgets, resource limits)
  - Accessibility (WCAG compliance level)

  DELETE the examples below and replace with your own.
  The examples are illustrative — they may not fit your project.
-->

### Principle 1: {Simplicity First}

**Level:** MUST

**Statement:** {Every feature must start with the simplest viable implementation. Additional complexity requires documented justification in the spec or plan.}

**Rationale:** {Over-engineering is the most common failure mode in AI-assisted development. LLMs naturally gravitate toward elaborate abstractions. Constraining complexity at the spec level prevents it from reaching the code.}

**Gate Checks:**

- [ ] {Spec does not introduce abstractions beyond what the acceptance criteria require}
- [ ] {Plan uses ≤ N projects/packages for initial implementation (define your limit)}
- [ ] {No "future-proofing" language in the spec (e.g., "in case we later need...")}

**Exceptions:** {Infrastructure or platform work where layering is inherent to the domain.}

---

### Principle 2: {Test-First Development}

**Level:** MUST

**Statement:** {Every plan must define test tasks before implementation tasks within each phase. Tests are written first; implementation makes them pass.}

**Rationale:** {TDD ensures that acceptance criteria are mechanically verified, not just hoped for. It also prevents scope creep — if there's no test for it, it doesn't get built.}

**Gate Checks:**

- [ ] {Each phase in the plan has test tasks listed before implementation tasks}
- [ ] {Every Gherkin scenario in the spec maps to at least one test task in the plan}
- [ ] {No implementation task exists without a corresponding test task}

**Exceptions:** {Documentation-only tasks, configuration changes with no runtime behavior.}

---

### Principle 3: {Security by Default}

**Level:** MUST

**Statement:** {Every spec involving user input, authentication, authorization, or data storage must include explicit security requirements as NFRs.}

**Rationale:** {Security cannot be bolted on after implementation. If it's not in the spec, it won't be in the code — or worse, it will be implemented inconsistently.}

**Gate Checks:**

- [ ] {Specs with user input include input validation NFRs}
- [ ] {Specs with authentication include session management and token handling NFRs}
- [ ] {Specs with data storage include encryption-at-rest and access control NFRs}

**Exceptions:** {Internal tooling with no user-facing surface and no sensitive data.}

---

### Principle 4: {Contract-First Integration}

**Level:** SHOULD

**Statement:** {When a track produces an API or data interface consumed by another track, the contract (endpoint shape, request/response types, error codes) should be defined in the spec before implementation begins.}

**Rationale:** {Undefined contracts between tracks cause integration failures. Defining them in the spec enables `/review-specs --epic` to check cross-track consistency.}

**Gate Checks:**

- [ ] {Specs for API-producing tracks include endpoint contracts in Technical Notes or a dedicated section}
- [ ] {Dependent tracks reference the parent track's contract in their Dependencies section}

**Exceptions:** {Tracks with no external consumers (purely internal implementation).}

---

<!-- ADD MORE PRINCIPLES HERE AS YOUR PROJECT EVOLVES -->

---

## Amendment Log

<!-- Record changes to principles here for traceability.
     Format: YYYY-MM-DD | Added/Modified/Removed | Principle N | Rationale -->

| Date | Action | Principle | Rationale |
|------|--------|-----------|-----------|
| {DATE} | Created | All | Initial principles established |

---

*Last updated: {DATE}*
