# {PROJECT_NAME}

<!-- TEMPLATE: Replace {PROJECT_NAME} with your project's name throughout this file.
     Search for {placeholder} markers to find all customization points. -->

> Session configuration for Claude Code. Loaded by `/prime` at session start.

---

## Quick Reference

| Command | Purpose | Options |
|---------|---------|---------|
| `/prime` | Load project context, detect plugins | — |
| `/setup-project` | One-time GitHub project setup (board, labels, templates) | — |
| `/accept {input}` | Accept GitHub Issue or PRD file into Conductor tracks | `#123`, `prd/file.md` |
| `/implement-track {id}` | Implement a Conductor track (worktree + validation + review + PR) | `--supervised`, `--phase N`, `--no-worktree`, `--base {branch}`, `--dry-run` |
| `/implement-prompt {desc}` | Create track from description, then implement | `--supervised`, `--skip-questions`, `--no-worktree`, `--dry-run` |
| `/implement-prd {slug}` | Batch implement all tracks from an epic/PRD | `--supervised`, `--dry-run`, `--skip {id}`, `--start {id}` |
| `/conductor:setup` | Initialize project with Conductor | — |
| `/conductor:new-track` | Create track with spec.md + plan.md | — |
| `/conductor:implement` | Execute tasks from plan (TDD workflow) | `[track-id]`, `--task X.Y`, `--phase N` |
| `/conductor:status` | Display project progress | — |
| `/conductor:revert` | Git-aware undo by track/phase/task | — |
| `/conductor:manage` | Archive, restore, delete, rename tracks | — |
| `/team-spawn` | Spawn agent team (preset or custom) | `review`, `debug`, `feature`, `research`, `security`, `migration` |
| `/team-review` | Multi-reviewer parallel code review | `--reviewers security,performance,architecture` |
| `/team-feature` | Parallel feature development | `--team-size N`, `--plan-first` |
| `/team-debug` | Hypothesis-driven debugging | `--hypotheses N` |
| `/team-status` | Display team members and progress | — |
| `/team-shutdown` | Gracefully shut down a team | — |
| `/team-delegate` | Task delegation and workload management | `--rebalance` |
| `/full-review` | Multi-perspective code review | — |
| `/tdd-cycle` | Start TDD red-green-refactor cycle | — |
| `/security-sast` | Run SAST security scan | — |
| `/full-stack-feature` | End-to-end feature orchestration | — |

---

## Context Layer Status

| File | Status | Purpose |
|------|--------|---------|
| `CLAUDE.md` | ✅ | Session config, project overview, tech stack |
| `AGENTS.md` | ✅ | Multi-agent coordination rules |
| `context/product-vision.md` | ✅ | Problem, users, success criteria |
| `context/tech-stack.md` | ✅ | Technology decisions + plugin mapping |
| `context/conventions.md` | ✅ | Code style, commits, API patterns, validation |
| `context/guides/plugin-registry.md` | ✅ | Technology → plugin mapping (verified v1.5.5) |

---

## Agent & Plugin Configuration

| Component | Location | Status |
|-----------|----------|--------|
| Custom commands | `.claude/commands/` | ✅ prime, setup-project, accept, implement-track, implement-prompt, implement-prd |
| Conductor commands | Via plugin | ✅ setup, new-track, implement, status, revert, manage |
| Agent Teams commands | Via plugin | ✅ team-spawn, team-review, team-feature, team-debug, etc. |
| Settings | `.claude/settings.json` | ✅ Plugins, permissions, hooks |
| Conventions | `context/conventions.md` | ✅ Code style, validation, commits |
| Ignore patterns | `.claudeignore` | ✅ Build artifacts, secrets, node_modules |

---

## Project Overview

<!-- TEMPLATE: Fill in your project details -->

**{PROJECT_NAME}** — {DESCRIPTION}

- **Problem:** {what problem this solves}
- **Users:** {target users}
- **Approach:** {high-level technical approach}

---

## Tech Stack

<!-- TEMPLATE: Fill in your actual tech stack. Technology names must match
     entries in context/guides/plugin-registry.md for plugin auto-detection. -->

| Layer | Technology | Version |
|-------|-----------|---------|
| Frontend | {framework} | {version} |
| Backend | {framework} | {version} |
| Database | {database} | {version} |
| Auth | {provider} | {version} |
| Testing | {framework} | {version} |
| CI/CD | {platform} | — |
| Infrastructure | {platform} | — |
| Package Manager | {tool} | {version} |

---

## Project Structure

```
{PROJECT_NAME}/
├── .claude/                    # Claude Code config (commands, settings)
├── .github/                    # GitHub Actions CI/CD + issue templates
├── conductor/                  # Conductor artifacts (created by /conductor:setup)
│   ├── product.md              # Product vision & goals
│   ├── tech-stack.md           # Technology preferences
│   ├── workflow.md             # Development practices (TDD, commits)
│   ├── tracks.md               # Master track registry
│   └── tracks/                 # Track specs and plans
│       └── {track-id}/         # e.g., user-auth_20250115/
│           ├── spec.md         # WHAT (requirements, acceptance criteria)
│           └── plan.md         # HOW (phases, tasks, checkpoints)
├── context/                    # Project context layer (for /prime)
│   ├── product-vision.md       # Problem, users, success criteria
│   ├── tech-stack.md           # Technology decisions + plugin mapping
│   ├── conventions.md          # Code style, commits, API patterns
│   ├── decisions/              # Architecture Decision Records
│   └── guides/                 # Plugin registry + project-specific guides
│       └── plugin-registry.md  # Technology → plugin mapping
├── prd/                        # Committed PRD files (from /accept epic intake)
├── apps/                       # Application code
├── packages/                   # Shared libraries
└── [config files]              # Docker, CI, package manager, etc.
```

---

## Agent Strategy

Choose per-track in the track's `plan.md`:

- [ ] **Single** — Sequential work, one agent session
- [x] **Subagents** — Parallel independent work via Task tool (default)
- [ ] **Agent Teams** — Cross-layer coordination with inter-agent messaging

### When to Use Agent Teams

Use Agent Teams (via `/team-feature` or `/team-spawn`) when:
- Work requires mid-task coordination between agents (not just independent parallel work)
- Multiple agents need to negotiate contracts or APIs
- Complex cross-cutting features spanning frontend + backend + infrastructure

For most tracks, Subagents (Task tool) is sufficient and more token-efficient.

### Agent Teams Setup

```json
// ~/.claude/settings.json (user-level, not project-level)
{
  "teammateMode": "tmux"
}
```

Display modes: `tmux` (recommended), `iterm2` (macOS), `in-process` (default)

---

## Validation Commands

<!-- TEMPLATE: Replace with your project's actual commands -->

```bash
# Level 1: Static Analysis
{your lint command}
{your typecheck command}

# Level 2: Unit Tests
{your test command}

# Level 3: Test Coverage
{your coverage command}

# Level 4: Build
{your build command}

# Level 5: E2E Tests (full stack required)
{your e2e command}

# Full CI (single orchestration script)
{your ci script}
```

---

## Workflow: How Commands Work Together

### Initial Project Setup (Once)

```
1. /conductor:setup          → Initialize Conductor artifacts (product, tech-stack, workflow, styleguides)
2. /setup-project            → Create GitHub Project board, labels, issue templates, prd/ directory
3. /prime                    → Load context layer, detect plugins, verify installation
4. /plugin install {names}   → Install any missing plugins flagged by /prime
5. /prime                    → Re-run to confirm all plugins enabled
```

### Issue Intake (Stories, Bugs, Epics)

```
1. Create GitHub Issue        → Use the story, bug, or epic template
2. /accept #NNN              → Parse issue → create Conductor track(s) + update GitHub status
3. Review specs               → Check conductor/tracks/{id}/spec.md and plan.md
4. /implement-track {id}     → Implement a single track
   OR /implement-prd {slug}  → Implement all tracks from an epic (dependency-aware)
```

### Creating & Implementing Features

```
Option A: Issue Intake Workflow (recommended)
1. Create GitHub Issue        → Story, bug, or epic template
2. /accept #NNN              → Create track(s) from issue
3. /implement-track {id}     → Worktree → Conductor implement → Ralph Loop → Review → PR
   OR /implement-prd {slug}  → Batch implement all epic tracks with chained worktrees

Option B: Structured Track Workflow
1. /conductor:new-track      → Create spec.md + plan.md via interactive Q&A
2. /implement-track {id}     → Worktree → Conductor implement → Ralph Loop → Review → PR

Option C: Ad-Hoc from Description
1. /implement-prompt {desc}  → Gap analysis → create track → implement-track flow

Option D: Agent Teams (for complex cross-cutting work)
1. /team-feature "desc"      → Decompose → plan → spawn parallel implementers
   /team-review src/          → Multi-reviewer parallel code review
   /team-debug "error desc"  → Competing hypothesis investigation
```

### Monitoring & Management

```
/conductor:status            → Project-wide progress overview
/conductor:manage            → Archive, restore, delete, rename tracks
/conductor:revert            → Git-aware undo by track/phase/task
/team-status                 → Agent team progress (when teams are active)
```

---

## Conductor Integration

This project uses custom commands that wrap Hobson's Conductor plugin:

| Need | Use | Why |
|------|-----|-----|
| Load project context | `/prime` | Reads context/ + prd/ layers + detects plugins (project-specific) |
| Set up GitHub project | `/setup-project` | Creates Project board, labels, issue templates, prd/ directory |
| Accept issues into tracks | `/accept` | Routes by label: story/bug → 1 track, epic → N tracks + child issues |
| Initialize new project | `/conductor:setup` | Interactive project setup with product, tech-stack, workflow |
| Create a track | `/conductor:new-track` | Interactive Q&A → spec.md + plan.md |
| Implement with full workflow | `/implement-track` | Wraps `/conductor:implement` with worktree, Ralph Loop, review, PR, GitHub sync |
| Implement from description | `/implement-prompt` | Creates a track from free-form input, then calls `/implement-track` |
| Implement an entire epic | `/implement-prd` | Dependency-aware batch implementation with chained worktrees |
| View progress | `/conductor:status` | Project-wide track overview |
| Undo work | `/conductor:revert` | Git-aware undo by track/phase/task |
| Manage track lifecycle | `/conductor:manage` | Archive, restore, delete, rename |

**Key distinction:** `/implement-track` adds worktree isolation, whole-track validation (Ralph Loop), mandatory code review + security scanning, automated PR creation, and GitHub Issue status sync on top of Conductor's TDD-focused implementation engine. `/implement-prd` chains worktrees for dependent tracks — each branches from its parent's remote branch.

---

## Conventions Summary

See `context/conventions.md` for the full guide. Populate it with your project's specific conventions during setup (SETUP.md Step 1.2). Key sections to fill in: file naming, directory structure, code style, git conventions, API patterns, testing strategy, and track completion gates.

---

## Plugin Dependencies

### Core (pre-configured in settings.json)

| Plugin | Commands | Purpose |
|--------|----------|---------|
| `conductor` | setup, new-track, implement, status, revert, manage | Context-Driven Development workflow |
| `agent-teams` | team-spawn, team-review, team-feature, team-debug, team-status, team-shutdown, team-delegate | Multi-agent team orchestration |
| `developer-essentials` | — | Git, SQL, debugging, E2E, monorepo skills |
| `comprehensive-review` | full-review, pr-enhance | Multi-perspective code review |
| `security-scanning` | security-sast, security-hardening, security-dependencies | SAST, threat modeling |
| `full-stack-orchestration` | full-stack-feature | End-to-end feature orchestration |
| `tdd-workflows` | tdd-cycle, tdd-red, tdd-green, tdd-refactor | TDD methodology |
| `unit-testing` | test-generate | Test automation |

### Tech-Stack-Specific (detected by /prime)

Populated automatically based on `context/tech-stack.md`. See `context/guides/plugin-registry.md` for the full mapping.

```bash
# Install missing plugins after /prime reports them
/plugin marketplace add wshobson/agents    # (if not already added)
/plugin install {plugin-names}
```

---

## MCP Servers (Optional)

<!-- TEMPLATE: Add project-specific MCP servers to .mcp.json at the repo root.
     This file is checked into git so every team member gets the same integrations. -->

Configure MCP servers in `.mcp.json` (repo root). Examples: Slack, Figma, Sentry, BigQuery, databases. See the [MCP documentation](https://modelcontextprotocol.io/) for available servers.

---

## Design System Reference

<!-- TEMPLATE: Fill in your design system details -->

| Token | Value |
|-------|-------|
| Background (dark) | {color} |
| Background (light) | {color} |
| Primary accent | {color} |
| Font (body) | {font} |
| Font (code) | {font} |

---

## Architecture Notes

<!-- TEMPLATE: Add project-specific architecture notes -->

---

## Getting Started

```bash
# 1. Clone and setup
git clone {repo-url}
cd {PROJECT_NAME}
{install dependencies command}

# 2. Initialize with Claude Code
/conductor:setup                         # First time: create Conductor artifacts
/setup-project                           # First time: create GitHub Project, labels, templates
/prime                                    # Every session: load context + detect plugins
/plugin marketplace add wshobson/agents   # First time: register marketplace
/plugin install {missing-plugins}         # Install any plugins flagged by /prime
/prime                                    # Verify all plugins enabled

# 3. Start building (issue intake workflow)
# Create a GitHub Issue using the story/bug/epic template
/accept #1                               # Accept issue into Conductor track(s)
# Review specs in conductor/tracks/
/implement-track {track-id}              # Implement a single track
# OR
/implement-prd {epic-slug}              # Implement all tracks from an epic

# Alternative approaches
/conductor:new-track                     # Create a track interactively (no GitHub Issue)
/implement-prompt {description}          # Ad-hoc implementation from description
/team-feature "description" --plan-first # Agent Teams for complex features
```

---

## Compact Instructions

<!-- This section is re-read from disk after every compaction. Use it to tell Claude
     what to preserve when context is summarized. -->

When context compacts, preserve:

- **Active track context** — current track ID, phase, task number, and what was just completed
- **Source issue numbers** — any `gh#NNN` references needed for status sync
- **Dependency chain** — which tracks depend on which, and the current `--base` branch for chained worktrees
- **Failing test details** — if in a Ralph Loop, keep the exact error messages and iteration count
- **Architecture decisions** — any decisions made during this session that aren't yet in `context/decisions/`

Summarize aggressively:

- File contents that were read for reference (re-read from disk if needed)
- Completed phases and their validation output (the plan.md has the status)
- Plugin detection output from `/prime` (re-run if needed)

### When to Compact Manually

Use `/compact` at natural breakpoints — not mid-implementation:

- **After `/accept`** completes (specs created, before implementation begins)
- **Between tracks** in `/implement-prd` (one track done, next not started)
- **After Ralph Loop passes** (validation done, before review step)
- **After a failed approach** (before trying a different strategy)
- **After research/exploration** (before writing code)

Never compact mid-phase, mid-debug, or while holding uncommitted context that isn't written to files.

### Compact with Focus

When running `/compact` manually, specify what to keep:

```
/compact focus on track {track-id} implementation — preserve current phase, failing tests, and dependency chain
```

---

*Last updated: {DATE}*
