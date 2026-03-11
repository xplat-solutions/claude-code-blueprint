# Technology Stack

<!-- PRIORITY: Fill before first track — /prime needs this for plugin detection. -->

> Technology decisions and rationale. `/prime` reads this file to auto-detect which plugins to suggest.
>
> **How to use:** Fill in each table with your chosen technologies. Technology names should match entries in `guides/plugin-registry.md` for automatic plugin detection.
>
> **Not every section applies.** Delete categories that don't fit your project. A CLI tool might only need Language, Testing, and CI/CD. A mobile app might replace Frontend/Backend with Platform/API. A data pipeline might need Language, Storage, and Orchestration. Adapt the structure to match your architecture.

---

## Core Stack

<!-- Delete or rename sections that don't apply to your project. -->

### Language & Runtime

| Component | Technology | Version | Notes |
|-----------|------------|---------|-------|
| Language | [e.g., TypeScript, Python, Go, Rust, Java] | | |
| Runtime | [e.g., Node.js, CPython, Go toolchain] | | |

### Frontend (delete if no UI)

| Component | Technology | Version | Notes |
|-----------|------------|---------|-------|
| Framework | [e.g., React, Vue, Svelte, Angular, SwiftUI] | | |
| Build Tool | [e.g., Vite, Webpack, esbuild] | | |
| State | [e.g., Zustand, Redux, Pinia] | | |
| Styling | [e.g., Tailwind CSS, CSS Modules] | | |
| Components | [e.g., shadcn/ui, Material UI] | | |
| Routing | [e.g., React Router, Vue Router] | | |

### Backend / API (delete if client-only)

| Component | Technology | Version | Notes |
|-----------|------------|---------|-------|
| Framework | [e.g., Express, FastAPI, Gin, Actix, Spring Boot] | | |
| Database | [e.g., PostgreSQL, MySQL, MongoDB, SQLite] | | |
| ORM / Query | [e.g., Drizzle, Prisma, SQLAlchemy, GORM, JOOQ] | | |
| Validation | [e.g., Zod, Pydantic, go-validator] | | |
| Auth | [e.g., Clerk, Auth0, custom JWT] | | |
| Queue | [e.g., BullMQ, Celery, custom] | | Optional |
| Cache | [e.g., Redis, Memcached] | | Optional |

### Infrastructure

| Component | Technology | Notes |
|-----------|------------|-------|
| Hosting | [e.g., AWS, GCP, DigitalOcean, Vercel, Fly.io] | |
| Containers | [e.g., Docker + Docker Compose] | Optional |
| CI/CD | [e.g., GitHub Actions, GitLab CI, CircleCI] | |
| Monorepo | [e.g., Turborepo, Nx, Cargo workspaces, Go modules] | If applicable |

---

## Development Tools

### Code Quality

| Tool | Purpose | Config File |
|------|---------|-------------|
| [Linter] | Code linting | [config path] |
| [Formatter] | Code formatting | [config path] |
| [Type checker] | Type checking | [config path] |

### Testing

| Type | Framework | Config |
|------|-----------|--------|
| Unit | [e.g., Vitest, Jest, pytest] | |
| Component | [e.g., Testing Library, Vue Test Utils] | |
| Integration | [e.g., Supertest, httpx] | |
| E2E | [e.g., Playwright, Cypress] | |

### Commands

```bash
# Development
# [your dev commands here]

# Build
# [your build commands here]

# Test
# [your test commands here]

# Lint & Format
# [your lint/format commands here]

# Database
# [your database commands here]
```

---

## Package Management

| Environment | Manager | Lock File |
|-------------|---------|-----------|
| [scope] | [e.g., pnpm, npm, yarn, pip, cargo] | [lock file] |

---

## Environment Variables

| Variable | Purpose | Required | Default |
|----------|---------|----------|---------|
| `DATABASE_URL` | Database connection string | Yes | - |
| [Add your variables] | | | |

---

## Third-Party Services

| Service | Purpose | API Docs |
|---------|---------|----------|
| [Add your service] | Brief description | Link to docs |

---

## Decisions Log

| Decision | Date | Rationale | ADR |
|----------|------|-----------|-----|
| [e.g., Chose React + Vite] | [date] | [reason] | [link to ADR] |

---

*Last updated: 2026-02-12*
