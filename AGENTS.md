# Multi-Agent Coordination Guide

> How agents collaborate on the {PROJECT_NAME} project. Uses `wshobson/agents` plugins for orchestration, skills, and team coordination. Loaded by all sessions and subagents.

---

## Agent Architecture

{PROJECT_NAME} uses three agent modes. Choose per-track in the track's `plan.md`:

| Mode | When | How | Token Cost |
|------|------|-----|------------|
| **Single** | Sequential work, one feature layer at a time | Default `/implement-track` or `/conductor:implement` session | Low |
| **Subagents** | Parallel independent work within one session | Task tool spawns focused workers (default for most tracks) | Medium |
| **Agent Teams** | Cross-layer coordination requiring discussion | `/team-feature`, `/team-spawn` — full independent sessions | High (100-300k) |

The default is **Subagents**. Most tracks can split frontend and backend into parallel subagents. Use **Agent Teams** only when teammates need to communicate mid-task.

---

## Hobson Agents: Available Teams & Commands

### Agent Teams Plugin (requires `agent-teams` in enabledPlugins)

| Command | Purpose | Example |
|---------|---------|---------|
| `/team-spawn` | Spawn pre-configured or custom team | `/team-spawn review`, `/team-spawn feature`, `/team-spawn security` |
| `/team-review` | Multi-reviewer parallel code review | `/team-review src/ --reviewers security,performance,architecture` |
| `/team-feature` | Parallel feature development | `/team-feature "Add auth" --team-size 3 --plan-first` |
| `/team-debug` | Competing hypothesis debugging | `/team-debug "API 500 on POST /users" --hypotheses 3` |
| `/team-delegate` | Task delegation dashboard | `/team-delegate --rebalance` |
| `/team-status` | Team progress overview | `/team-status` |
| `/team-shutdown` | Graceful team shutdown | `/team-shutdown` |

### Preset Teams

| Preset | Team Composition | Use Case |
|--------|-----------------|----------|
| `review` | 3+ reviewers (security, performance, architecture) | Parallel code review across dimensions |
| `debug` | 3+ debuggers with competing hypotheses | Complex bug investigation |
| `feature` | Lead + 2-3 implementers | Parallel feature development with file ownership |
| `fullstack` | Frontend + Backend + Integration | Full-stack feature orchestration |
| `research` | 3 explore agents | Parallel codebase/web research |
| `security` | 4 reviewers (OWASP, auth, deps, config) | Comprehensive security audit |
| `migration` | Lead + 2 implementers + reviewer | Coordinated codebase migration |

### Agent Teams Agents (from `agent-teams` plugin)

| Agent | Role | Color |
|-------|------|-------|
| `team-lead` | Decomposes work, manages lifecycle, synthesizes results | Blue |
| `team-reviewer` | Multi-dimensional code reviewer | Green |
| `team-debugger` | Hypothesis investigator, evidence collector | Red |
| `team-implementer` | Parallel builder with strict file ownership | Yellow |

### Agent Teams Setup

```json
// ~/.claude/settings.json (user-level)
{
  "teammateMode": "tmux"
}
```

Display modes: `tmux` (recommended — each teammate in tmux pane), `iterm2` (macOS — each teammate in iTerm2 tab), `in-process` (default — same process)

The project-level `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` env var is already set in `.claude/settings.json`.

---

## Subagent Roles

When `/implement-track` or `/conductor:implement` spawns subagents via the Task tool, each subagent receives context through its task description. The main agent should include references to relevant `context/` files so subagents can read project-specific context directly.

### Module Subagents

Subagents are scoped to ownership zones — non-overlapping directory boundaries that prevent merge conflicts. Define zones based on your architecture. Examples:

**Web app:** Frontend subagent (owns `apps/web/`), Backend subagent (owns `apps/api/`), Shared Types subagent (owns `packages/shared-types/` — runs first when new types are needed).

**Monolith:** Domain subagents scoped by module (owns `src/auth/`, `src/billing/`, etc.).

**CLI / library:** Core subagent (owns `src/`), Test subagent (owns `tests/`).

Task descriptions should reference `context/conventions.md` for code style and `context/tech-stack.md` for framework details. Skills auto-activate from installed plugins based on file context.

### Code Review (via Hobson)

Use Hobson's review commands — no custom agent needed:

- `/full-review` — Single-agent multi-perspective review
- `/team-review src/ --reviewers security,performance,architecture` — Parallel team review
- `/team-spawn review` — Pre-configured review team

### Security Review (via Hobson)

Use Hobson's security commands — no custom agent needed:

- `/security-sast` — SAST security scan (mandatory in `/implement-track` workflow)
- `/security-hardening` — Security hardening recommendations
- `/team-spawn security` — 4-reviewer parallel security audit (OWASP, auth, deps, config)

---

## File Ownership Rules

Subagents and Agent Team implementers must respect file boundaries:

```
[module-a dirs]    → Module A subagent / implementer ONLY
[module-b dirs]    → Module B subagent / implementer ONLY
[shared dirs]      → Shared subagent (or whichever runs first)
[config files]     → Either (read-only during track implementation)
```

Define your actual ownership zones in each track's `plan.md` (see Track-Specific Agent Strategies below). For a web app this typically maps to frontend/backend/shared-types. For a monolith it maps to domain modules.

**Critical:** Two agents must NEVER edit the same file. If a task requires changes across boundaries, run it sequentially (not in parallel) or coordinate via the shared package.

---

## Coordination Patterns

### Pattern 1: Contracts-First (Default for Subagents)

Define shared contracts (types, interfaces, schemas) first, then implement modules in parallel.

```
1. Shared subagent → defines contracts (types, validation schemas, API shapes)
2. Module A        → implements using those contracts     } parallel
3. Module B        → implements using those contracts     }
```

### Pattern 2: Parallel Independent (for Agent Teams)

```
/team-feature "Add user dashboard" --team-size 3 --plan-first
→ Lead decomposes into work streams with file ownership
→ Implementers build in parallel
→ Lead integrates and validates
```

### Pattern 3: Sequential Integration

When modules must integrate (real-time flows, cross-module transactions):

```
1. Shared contracts → types + schemas
2. Module A         → core implementation
3. Module B         → consuming implementation
4. Integration      → end-to-end flow (sequential, must be single agent)
```

---

## Validation Protocol

Every subagent and team implementer must run validation before reporting completion:

```bash
# Minimum (every commit)
# [your lint + typecheck commands]

# After implementation (every phase)
# [your test command]

# Final (track completion)
# [your full test + build commands]
```

Define your actual validation commands in `CLAUDE.md` under Validation Commands.

If validation fails, the agent must fix the issue before returning results. Do NOT return with failing tests.

---

## Commit Conventions for Agents

Subagents and Agent Teams commit with the track and phase prefix:

```
feat(track-{id}/phase-{M}): {description}

# Examples:
feat(user-auth_20250115/phase-1): add API endpoints
feat(user-auth_20250115/phase-2): build UI components
fix(user-auth_20250115): resolve merge conflict in shared types
```

All commits within worktree branch. PR created at track completion via `gh pr create`.

### AI-Context Footer

When a commit modifies any file in `.claude/`, `context/`, or `conductor/`, agents **must** append an `AI-context:` trailer to the commit message. This tracks how the AI layer evolves over time and feeds into `/prime`'s git log review.

```
feat(track-user-auth_20250115/phase-1): add API endpoints

AI-context: added auth middleware pattern to context/guides/backend-conventions.md
```

See `context/conventions.md` → AI-Context Footer for the full convention. The footer is only required when AI layer files were actually modified — most commits won't need it.

---

## Context Loading

**Main session** (after `/prime`) loads: CLAUDE.md, AGENTS.md, all `context/**/*.md` files, `prd/*.md` files, conductor artifacts. Plugin skills auto-activate based on file context.

**Track creation:** Tracks can originate from GitHub Issues via `/accept` (story/bug → 1 track, epic → N tracks with child issues) or from `/conductor:new-track` and `/implement-prompt`. Tracks created by `/accept` include a `Source: gh#NNN` reference in their spec.md, enabling GitHub Issue status sync during implementation.

**Subagents are isolated.** They receive only:

- The task description from the main agent (should reference specific `context/` files for project context)
- Skills from installed plugins (auto-activate based on file context)
- Access to project files (can read context files directly)

**Agent Teams are independent sessions.** Each teammate:

- Has full Claude Code capabilities
- Can read project files and context independently
- Communicates via structured messages (not shared memory)
- Has its own tool access and can make commits

Task descriptions (for subagents) must include: which track/phase, files to create/modify, acceptance criteria, relevant API contracts.

### Dynamic System Prompt for Subagents

When spawning a subagent via CLI (outside the Task tool), use `--system-prompt` to inject surgical context without loading the full context layer:

```bash
claude --system-prompt "$(cat context/conventions.md context/tech-stack.md)" "Implement the auth middleware in src/auth/"
```

This is useful when a subagent needs specific context files but loading everything via `/prime` would waste tokens. The Task tool handles context injection through task descriptions; this technique is for CLI-spawned agents and scripts.

---

## Plugin Skill Injection

Subagents and Agent Team members get domain expertise from `wshobson/agents` plugins.

**How it works:**

1. Plugins installed via `/plugin install {name}` (from `wshobson/agents` marketplace)
2. For the main thread: skills auto-load based on file context when plugins are enabled
3. For subagents spawned via Task tool: include references to relevant `context/` files in the task description so the subagent reads project-specific context
4. For Agent Teams: each teammate is a full Claude Code session that auto-loads plugins and can read context files independently

**If plugins are not installed:** Agents still function but without framework-specific expertise.

---

## Error Escalation

If a subagent or team member encounters a blocker:

1. **Try to fix it** (up to 3 attempts for test failures)
2. **Check conventions** — re-read `context/conventions.md` for patterns
3. **Return with details** — report the error, what was tried, and what's needed
4. **Never silently skip** — failing tests or lint errors must be reported
5. **CI breaks are P0** — if work breaks CI on main, fix CI before starting anything new

The main agent or team lead decides whether to retry, switch to sequential, or escalate.

---

## Track-Specific Agent Strategies

Define per-track strategies in your track's `plan.md`:

```markdown
## Agent Strategy

**Mode:** Subagents (or Single, or Agent Teams)
**Justification:** {why this mode for this track}

| Phase | Agent | Parallel | Notes |
|-------|-------|----------|-------|
| 1. Shared contracts | Single | No | Define types/schemas first |
| 2. Module A | Module A subagent | Yes (with Phase 3) | Implements core logic |
| 3. Module B | Module B subagent | Yes (with Phase 2) | Implements consuming logic |
| 4. Integration | Single | No | Connect modules end-to-end |
```

---

## Reusable Patterns

As you build tracks, document reusable patterns here:

- Shared utilities and abstractions that multiple features use
- Integration patterns (API clients, event handling, message queues)
- Data access patterns (repositories, query builders, caching strategies)
- Testing patterns (fixture generation, mocking strategies, test helpers)
- Error handling patterns (retry logic, circuit breakers, fallbacks)

Before creating a new component or service, check if a similar pattern exists in an earlier track.

---

*Last updated: {DATE}*
