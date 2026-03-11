# Technology Stack

> Technology decisions and rationale. `/prime` reads this file to auto-detect which plugins to suggest.
>
> **How to use:** Fill in each table with your chosen technologies. The technology names should match entries in `guides/plugin-registry.md` for automatic plugin detection.

---

## Core Stack

### Frontend

| Component | Technology | Version | Notes |
|-----------|------------|---------|-------|
| Framework | [e.g., React, Vue, Svelte, Angular] | | |
| Build Tool | [e.g., Vite, Webpack, esbuild] | | |
| Language | [e.g., TypeScript, JavaScript] | | |
| State | [e.g., Zustand, Redux, Pinia] | | |
| Styling | [e.g., Tailwind CSS, CSS Modules, styled-components] | | |
| Components | [e.g., shadcn/ui, Material UI, Ant Design] | | |
| Routing | [e.g., React Router, Vue Router] | | |
| HTTP Client | [e.g., Axios, fetch, ky] | | |

### Backend

| Component | Technology | Version | Notes |
|-----------|------------|---------|-------|
| Runtime | [e.g., Node.js, Python, Go, Rust] | | |
| Framework | [e.g., Express, FastAPI, Gin, Actix] | | |
| Language | [e.g., TypeScript, Python, Go] | | |
| Database | [e.g., PostgreSQL, MySQL, MongoDB] | | |
| ORM | [e.g., Drizzle, Prisma, SQLAlchemy, GORM] | | |
| Validation | [e.g., Zod, Joi, Pydantic] | | |
| Auth | [e.g., Clerk, Auth0, NextAuth, custom JWT] | | |
| Queue | [e.g., BullMQ, Celery, custom] | | Optional |
| Cache | [e.g., Redis, Memcached] | | Optional |

### Infrastructure

| Component | Technology | Notes |
|-----------|------------|-------|
| Hosting | [e.g., AWS, GCP, DigitalOcean, Vercel] | |
| Containers | [e.g., Docker + Docker Compose] | Optional |
| CI/CD | [e.g., GitHub Actions, GitLab CI] | |
| Monorepo | [e.g., pnpm + Turborepo, Nx, Lerna] | If applicable |

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
