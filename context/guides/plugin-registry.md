# Plugin & Skill Registry

> Maps technologies to `wshobson/agents` plugins. Used by `/prime` to auto-detect and suggest plugins based on `context/tech-stack.md`. All plugin names are verified against the marketplace as of v1.5.5.

---

## How It Works

1. `/prime` reads `context/tech-stack.md` and extracts technology names
2. Technologies are matched against the mapping table below
3. Matching plugins are suggested for installation via `/plugin install {name}`
4. For the main thread, skills auto-load based on file context — no manual invocation needed
5. For subagents, skills auto-activate based on file context when the subagent reads files in matching directories

---

## Plugin Marketplace Setup

All plugins come from the `wshobson/agents` marketplace.

```bash
# One-time setup — register the marketplace
/plugin marketplace add wshobson/agents

# Install a plugin
/plugin install {plugin-name}

# List available plugins
/plugin

# Cache troubleshooting (if plugins fail to load)
rm -rf ~/.claude/plugins/cache/claude-code-workflows
rm ~/.claude/plugins/installed_plugins.json
```

Full marketplace: https://github.com/wshobson/agents

---

## Core Plugins (Always Recommended)

These plugins are tech-stack-independent and should be enabled for all projects:

| Plugin | Version | Purpose |
|--------|---------|---------|
| `conductor` | 1.2.1 | Context-Driven Development — project setup, track creation, implementation workflow, status, revert, manage |
| `agent-teams` | 1.0.2 | Multi-agent orchestration — parallel review, debugging, feature development, team coordination |
| `developer-essentials` | 1.0.1 | Git workflows, SQL optimization, error handling, code review, E2E testing, monorepo management |
| `comprehensive-review` | 1.3.0 | Multi-perspective code review — architecture, security, best practices |
| `security-scanning` | 1.3.0 | SAST analysis, dependency scanning, OWASP compliance, threat modeling |
| `full-stack-orchestration` | 1.3.0 | End-to-end feature orchestration with testing, security, performance, deployment |
| `tdd-workflows` | 1.3.0 | Test-driven development red-green-refactor cycles |
| `unit-testing` | 1.2.0 | Unit and integration test automation with debugging support |

These are pre-configured in `.claude/settings.json` `enabledPlugins`.

---

## Context Management (Recommended with MCP Servers)

If your project uses 3+ MCP servers, install context-mode to compress tool outputs and preserve context window capacity:

| Plugin | Marketplace | Purpose |
|--------|-------------|---------|
| `context-mode` | `mksglu/claude-context-mode` | Compresses MCP tool outputs by 95-99% via sandboxed execution + FTS5 search. Extends sessions from ~30 min to ~3 hours. |

```bash
/plugin marketplace add mksglu/claude-context-mode
/plugin install context-mode
```

This is a separate marketplace from `wshobson/agents`. It's additive — context-mode sits between Claude Code and your MCP servers, compressing outputs before they enter the context window. No conflicts with Hobson plugins.

---

## Technology → Plugin Mapping

Use this table to determine which additional plugins to install based on your `context/tech-stack.md`.

### Languages

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| TypeScript, JavaScript, JS, TS, `.ts`, `.tsx`, `.js`, Node.js, React | `javascript-typescript` | JS/TS patterns, Node.js backend, React, async, testing |
| Python, `.py`, Django, Flask, FastAPI | `python-development` | Python 3.12+, Django, FastAPI, async, uv/ruff |
| Rust, `.rs`, Cargo | `systems-programming` | Rust, Go, C/C++ for performance-critical development |
| Go, Golang, `.go` | `systems-programming` | Go idioms, concurrency, module management |
| Java, `.java`, Spring, Kotlin, `.kt`, Scala | `jvm-languages` | Java, Kotlin, Scala with enterprise patterns |
| C#, `.cs`, .NET, ASP.NET | `dotnet-contribution` | C#, ASP.NET Core, Entity Framework, Dapper |
| Ruby, `.rb`, PHP, `.php` | `web-scripting` | PHP and Ruby web frameworks, CMS development |
| Elixir, OTP, Phoenix | `functional-programming` | Elixir, OTP patterns, Phoenix framework |
| Julia | `julia-development` | Julia 1.10+, scientific computing, package management |
| Shell, Bash, `.sh` | `shell-scripting` | Production Bash scripting, POSIX compliance |
| ARM, Cortex-M, embedded | `arm-cortex-microcontrollers` | ARM firmware, Teensy, STM32, nRF52, SAMD |

### Frontend Frameworks

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| React, React Native, Next.js, Vue, Angular, Svelte, Tailwind | `frontend-mobile-development` | Frontend UI + mobile development across platforms |
| UI/UX, design system, iOS, Android | `ui-design` | UI/UX design for mobile and web applications |

### Backend Frameworks

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| Express, Fastify, NestJS, API, REST, GraphQL, DI, DDD | `backend-development` | API architecture, DDD, event sourcing, CQRS, Temporal |
| API scaffolding, endpoint generation | `api-scaffolding` | REST and GraphQL API scaffolding and generation |

### Infrastructure & DevOps

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| Docker, Kubernetes, K8s, Helm | `kubernetes-operations` | K8s manifests, Helm charts, networking, security, GitOps |
| CI/CD, GitHub Actions, GitLab CI | `cicd-automation` | Pipeline config, GitHub Actions, GitLab CI, deployment |
| AWS, Azure, GCP, Terraform, cloud | `cloud-infrastructure` | Multi-cloud architecture, Terraform, service mesh |
| Deployment, rollback, blue-green, canary | `deployment-strategies` | Deployment patterns, rollback automation |
| Pre-deploy validation | `deployment-validation` | Configuration validation, readiness assessment |
| Incident, PagerDuty, on-call | `incident-response` | Production incident management and triage |
| Monitoring, metrics, traces, logs | `observability-monitoring` | Metrics, logging, distributed tracing, SLOs |

### Database & Data

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| PostgreSQL, MySQL, SQL, schema, migration | `database-design` | Schema design, indexing, PostgreSQL patterns |
| Database migration, Drizzle, Prisma, Knex | `database-migrations` | Migration automation, cross-database strategies |
| Database optimization, query performance | `database-cloud-optimization` | Query optimization, cloud cost optimization |
| ETL, data pipeline, warehouse | `data-engineering` | ETL pipelines, data warehouse, batch processing |
| Schema validation, data quality | `data-validation-suite` | Schema validation, data quality monitoring |

### Testing

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| Unit test, Jest, Vitest, pytest | `unit-testing` | Unit and integration test automation |
| TDD, red-green-refactor | `tdd-workflows` | TDD methodology with cycle management |
| Playwright, Cypress, E2E | `developer-essentials` | E2E testing patterns (via `e2e-testing-patterns` skill) |
| Performance test, load test | `performance-testing-review` | Performance analysis, test coverage review |
| API testing, OpenAPI, mock | `api-testing-observability` | API test automation, request mocking, OpenAPI docs |

### Security

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| SAST, OWASP, vulnerability | `security-scanning` | SAST, dependency scanning, threat modeling |
| SOC2, HIPAA, GDPR, compliance | `security-compliance` | Compliance validation, secrets scanning |
| API security, auth, rate limiting | `backend-api-security` | API hardening, authentication, authorization |
| Frontend security, XSS, CSRF | `frontend-mobile-security` | XSS/CSRF prevention, CSP, secure storage |

### Payments & Business

| Technology Keywords | Plugin Name | What It Provides |
|--------------------|-------------|-----------------|
| Stripe, PayPal, payments, billing | `payment-processing` | Payment gateway integration, subscription billing |
| Blockchain, Solidity, Web3, DeFi | `blockchain-web3` | Smart contracts, DeFi, NFT platforms |
| Trading, quantitative, financial | `quantitative-trading` | Algorithmic trading, financial modeling |
| Business metrics, KPIs | `business-analytics` | Business metrics analysis, financial reporting |
| LLM, RAG, vector search, AI agents | `llm-application-dev` | LLM apps, LangGraph, RAG, vector search |
| ML, model training, MLOps | `machine-learning-ops` | ML pipelines, model deployment, experiment tracking |

---

## Verified Skill Names by Plugin

These are the actual skill names available from each plugin. Skills auto-activate based on file context.

### `javascript-typescript`
- `nodejs-backend-patterns` — Node.js servers, REST APIs, middleware, auth
- `modern-javascript-patterns` — ES6+, async/await, functional patterns
- `typescript-advanced-types` — Generics, conditional types, utility types
- `javascript-testing-patterns` — Jest, Vitest, Testing Library, TDD

### `frontend-mobile-development`
- `react-state-management` — Redux Toolkit, Zustand, Jotai, React Query
- `tailwind-design-system` — Tailwind v4, design tokens, component libraries
- `nextjs-app-router-patterns` — Next.js 14+, Server Components, streaming
- `react-native-architecture` — Expo, navigation, native modules

### `backend-development`
- `api-design-principles` — REST/GraphQL API design, scalability
- `architecture-patterns` — Clean Architecture, Hexagonal, DDD
- `microservices-patterns` — Service boundaries, event-driven communication
- `cqrs-implementation` — Command Query Responsibility Segregation
- `event-store-design` — Event sourcing infrastructure
- `saga-orchestration` — Distributed transactions, compensating workflows
- `workflow-orchestration-patterns` — Temporal workflows, durable execution
- `temporal-python-testing` — Temporal workflow testing with pytest
- `projection-patterns` — Read models, materialized views

### `python-development`
- `async-python-patterns` — asyncio, concurrent programming
- `python-testing-patterns` — pytest, fixtures, mocking, TDD
- `python-type-safety` — Type hints, generics, protocols, mypy
- `python-design-patterns` — KISS, SRP, composition over inheritance
- `python-error-handling` — Validation, exception hierarchies
- `python-project-structure` — Module architecture, public API design
- `python-code-style` — Linting, formatting, naming conventions
- `python-performance-optimization` — Profiling, bottleneck optimization
- `python-observability` — Structured logging, metrics, tracing
- `python-resilience` — Retries, backoff, timeouts, fault tolerance
- `python-background-jobs` — Task queues, workers, async processing
- `python-configuration` — Environment vars, pydantic-settings
- `python-resource-management` — Context managers, cleanup, streaming
- `python-packaging` — pyproject.toml, distributable packages
- `python-anti-patterns` — Common anti-patterns to avoid
- `uv-package-manager` — uv for fast dependency management

### `developer-essentials`
- `code-review-excellence` — Code review practices, PR feedback
- `git-advanced-workflows` — Rebasing, cherry-picking, bisect, worktrees
- `e2e-testing-patterns` — Playwright, Cypress, reliability patterns
- `sql-optimization-patterns` — Query optimization, indexing, EXPLAIN
- `auth-implementation-patterns` — JWT, OAuth2, sessions, RBAC
- `error-handling-patterns` — Exceptions, Result types, graceful degradation
- `debugging-strategies` — Systematic debugging, profiling, root cause analysis
- `monorepo-management` — Turborepo, Nx, pnpm workspaces
- `turborepo-caching` — Turborepo local and remote caching
- `nx-workspace-patterns` — Nx workspace configuration
- `bazel-build-optimization` — Bazel builds for large monorepos

### `security-scanning`
- `sast-configuration` — SAST tools setup, DevSecOps
- `stride-analysis-patterns` — STRIDE threat identification
- `attack-tree-construction` — Threat path visualization
- `threat-mitigation-mapping` — Threats to security controls
- `security-requirement-extraction` — Security requirements from threat models

### `database-design`
- `postgresql` — PostgreSQL schema, indexing, advanced features

### `cicd-automation`
- `github-actions-templates` — Production GitHub Actions workflows
- `gitlab-ci-patterns` — GitLab CI/CD pipelines
- `deployment-pipeline-design` — Multi-stage pipelines, approval gates
- `secrets-management` — Vault, AWS Secrets Manager, CI secrets

### `cloud-infrastructure`
- `terraform-module-library` — Reusable Terraform modules
- `multi-cloud-architecture` — AWS/Azure/GCP decision framework
- `cost-optimization` — Resource rightsizing, reserved instances
- `istio-traffic-management` — Istio routing, circuit breakers
- `linkerd-patterns` — Linkerd service mesh
- `hybrid-cloud-networking` — VPN, dedicated connections
- `mtls-configuration` — Zero-trust mTLS
- `service-mesh-observability` — Distributed tracing, mesh monitoring

### `agent-teams`
- `team-composition-patterns` — Team sizing, preset configurations
- `task-coordination-strategies` — Task decomposition, dependency graphs
- `parallel-debugging` — Competing hypotheses, evidence collection
- `multi-reviewer-patterns` — Review dimensions, severity calibration
- `parallel-feature-development` — File ownership, conflict avoidance
- `team-communication-protocols` — Message types, approval workflow

---

## Auto-Loading Behavior

**For the main thread:** Skills auto-load based on file and directory context when plugins are installed and enabled.

| Working In | Skills That Activate |
|-----------|---------------------|
| `*.ts`, `*.tsx`, `*.js` files | `javascript-typescript` skills |
| React/Next.js/Vue components | `frontend-mobile-development` skills |
| Backend/API directories | `backend-development` + language-specific skills |
| Database migration files | `database-design` / `database-migrations` skills |
| `Dockerfile`, `docker-compose.yml` | `kubernetes-operations` skills |
| Test files (`*.test.*`, `*.spec.*`) | `unit-testing` / `tdd-workflows` skills |
| CI/CD files (`.github/workflows/`) | `cicd-automation` skills |
| Terraform files (`*.tf`) | `cloud-infrastructure` skills |

**For subagents:** Skills auto-activate when the subagent reads files in matching directories. Include references to relevant `context/` files in the task description so subagents have project-specific context.

---

## Skill Activation

For the **main thread**, skills auto-activate based on file context when plugins are installed — no manual configuration needed.

For **subagents** spawned via Task tool, include references to relevant `context/` files in the task description so the subagent reads project-specific context directly.

For **Agent Teams**, each teammate is a full Claude Code session that auto-loads plugins and can read context files independently.

**Code review and security** are handled entirely by Hobson commands (`/full-review`, `/team-review`, `/security-sast`, `/team-spawn security`) — no custom agents needed.

**Example skill recommendations by tech stack:**

| Stack | Recommended Skills |
|-------|--------------------|
| React + TypeScript | `react-state-management`, `typescript-advanced-types`, `tailwind-design-system`, `javascript-testing-patterns`, `e2e-testing-patterns` |
| Next.js + TypeScript | `nextjs-app-router-patterns`, `react-state-management`, `typescript-advanced-types`, `tailwind-design-system`, `javascript-testing-patterns` |
| Node.js + Express | `api-design-principles`, `architecture-patterns`, `nodejs-backend-patterns`, `typescript-advanced-types`, `javascript-testing-patterns`, `sql-optimization-patterns` |
| Python + FastAPI | `api-design-principles`, `architecture-patterns`, `async-python-patterns`, `python-testing-patterns`, `python-type-safety`, `sql-optimization-patterns` |
| Go backend | `api-design-principles`, `architecture-patterns`, `sql-optimization-patterns` |

---

## Updating This Registry

When wshobson/agents adds new plugins:

1. Check available plugins: `/plugin` or browse the marketplace
2. Clone the repo to see all skills: `git clone https://github.com/wshobson/agents.git`
3. Add new entries to the appropriate section above
4. Update `tech-stack.md` if new technologies are adopted

This registry is a reference — not a dynamic API. It maps known technologies to known plugins and is maintained alongside the project.

---

*Last updated: {DATE} — verified against wshobson/agents marketplace v1.5.5*
