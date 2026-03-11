# Setup Guide

> Step-by-step instructions for initializing a new project from this blueprint. This guide is written for both human developers and LLM agents — any agent reading this file should be able to set up a fully functional project.

---

## Quick Start (15 minutes)

If you want to get building fast, do only these 4 steps. Everything else can be filled in incrementally.

```bash
# 1. Replace placeholders in all files
#    Find-and-replace: {PROJECT_NAME}, {DESCRIPTION}, {DATE}

# 2. Fill in your validation commands in CLAUDE.md
#    This is the single most important thing — without real commands, the Ralph Loop can't validate.

# 3. Fill in context/tech-stack.md with your actual technologies
#    Technology names must match plugin-registry.md for /prime to detect plugins.

# 4. Initialize and verify
/plugin marketplace add wshobson/agents
/conductor:setup                         # Create Conductor artifacts
/prime                                    # Load context + detect plugins
/plugin install {missing-plugins}         # Install plugins flagged by /prime
/prime                                    # Verify all plugins enabled
```

After these 4 steps you can run `/accept`, `/implement-prompt`, or `/conductor:new-track` to start building. Fill in conventions, principles, GitHub project setup, and guides as you go — they improve quality but aren't blockers.

---

## Complete Setup

The sections below cover the full setup process. If you did the Quick Start above, skip to Step 2 (Hobson Agents) and work through the rest when ready.

---

## Step 1: Customize Project Identity

### 1.1 Find and replace placeholders

Search for these placeholders across all files and replace them:

| Placeholder | Replace With | Example |
|-------------|-------------|---------|
| `{PROJECT_NAME}` | Your project name | `AtlasClaims` |
| `{DESCRIPTION}` | One-line description | `Healthcare AI claims processing platform` |
| `{DATE}` | Current date | `2026-02-28` |

Files to update: `CLAUDE.md`, `AGENTS.md`, `SETUP.md`, `README.md`, `context/product-vision.md`

### 1.2 Fill in the context layer

These files define your project for all agents and sessions. Each file has a specific audience and purpose:

| File | What to Fill In | Who Reads It |
|------|----------------|--------------|
| `context/product-vision.md` | Problem statement, target users, success criteria, product scope, key differentiators | All agents (via `/prime`), LLM agents writing specs |
| `context/tech-stack.md` | Technology choices with versions. **Technology names must match keywords in `context/guides/plugin-registry.md`** for plugin auto-detection | All agents (via `/prime`), `/prime` plugin detection |
| `context/conventions.md` | Code style, naming conventions, commit format, API patterns, validation gates, testing tiers | All agents, subagents (referenced in task descriptions) |
| `context/principles.md` | Architectural principles with MUST/SHOULD levels, rationale, and gate checks. Start with 3-5 principles. | `/review-specs` (governance gates), all agents (via `/prime`) |
| `CLAUDE.md` | Validation commands (actual lint/test/build commands), design system tokens, project-specific rules | Main session (loaded first by `/prime`) |

**For LLM agents creating context files:** Be thorough. These files are the primary source of truth for all implementation decisions. Include specific technology versions, directory paths, API patterns with examples, and concrete validation commands. Other LLM agents will read these files to write specs and implement features — vague or incomplete context leads to poor implementation quality.

---

## Step 2: Set Up Hobson Agents

### 2.1 Register the marketplace

```bash
# One-time setup — register the wshobson/agents marketplace
/plugin marketplace add wshobson/agents
```

### 2.2 Install core plugins

These are always recommended regardless of tech stack:

```bash
/plugin install conductor
/plugin install agent-teams
/plugin install developer-essentials
/plugin install comprehensive-review
/plugin install security-scanning
/plugin install full-stack-orchestration
/plugin install tdd-workflows
/plugin install unit-testing
```

### 2.3 Install tech-stack-specific plugins

Based on your `context/tech-stack.md`, install additional plugins. Examples:

```bash
# JavaScript/TypeScript project
/plugin install javascript-typescript

# React + Tailwind frontend
/plugin install frontend-mobile-development

# Node.js backend
/plugin install backend-development

# PostgreSQL database
/plugin install database-design

# CI/CD with GitHub Actions
/plugin install cicd-automation

# Docker/Kubernetes
/plugin install kubernetes-operations
```

Full mapping: `context/guides/plugin-registry.md`

### 2.4 Configure Agent Teams (optional but recommended)

Add to your `~/.claude/settings.json` (user-level, not project-level):

```json
{
  "teammateMode": "tmux"
}
```

Options: `tmux` (recommended — each teammate in tmux pane), `iterm2` (macOS), `in-process` (default)

The project-level `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is already set in `.claude/settings.json`.

### 2.5 Update CLAUDE.md

Fill in the remaining sections of `CLAUDE.md`:

- Validation commands (your actual lint, test, build, E2E commands)
- Design system tokens (delete this section if no UI)
- Project name and description

---

## Step 3: Initialize Conductor

Run Conductor's interactive setup to create project-level documentation:

```bash
/conductor:setup
```

This creates the `conductor/` directory with:
- `product.md` — Product vision & goals
- `tech-stack.md` — Technology preferences
- `workflow.md` — Development practices (TDD, commits)
- `code_styleguides/` — Language-specific conventions
- `tracks.md` — Master track registry
- `tracks/` — Directory for track specs and plans

Conductor's setup detects whether this is a greenfield or brownfield project and asks sequential questions about your product, tech stack, and workflow preferences.

---

## Step 4: Set Up GitHub Project

Run the one-time GitHub project setup command:

```bash
/setup-project
```

This creates:
- **GitHub Project board** with status columns: Backlog → Accepted → Implementing → In Review → Done
- **Labels** — `story`, `bug`, `epic`, `priority:high`, `priority:medium`, `priority:low`, `accepted`, `implementing`
- **Issue templates** — `.github/ISSUE_TEMPLATE/story.yml`, `bug.yml`, `epic.yml` (with Gherkin acceptance criteria)
- **`prd/` directory** — For committed PRD files saved by `/accept` from epic issues

### Issue Intake Workflow

After setup, the standard workflow for accepting work into the project is:

```
1. Create GitHub Issue        → Use the story, bug, or epic template
2. /accept #NNN              → Parse issue → create Conductor track(s) → flag [NEEDS CLARIFICATION] markers
3. Resolve clarifications     → Fix any [NEEDS CLARIFICATION] markers in spec.md
4. /review-specs {id}        → Validate spec↔plan consistency, coverage, principles → writes review.md
5. Review specs               → Check conductor/tracks/{id}/spec.md, plan.md, and review.md
6. /implement-track {id}     → Verifies review.md exists → implement a single track
   OR /implement-prd {slug}  → Implement all tracks from an epic (dependency-aware chained worktrees)
```

For epics, `/accept` decomposes the PRD into multiple tracks and creates child GitHub Issues for each story. `/implement-prd` then implements all tracks respecting the dependency graph — each dependent track branches from its parent track's remote branch.

---

## Step 5: Add Project-Specific Guides (Optional)

The `context/guides/` directory ships with only `plugin-registry.md` (universal technology → plugin mapping). You can add project-specific guides as needed:

```
context/guides/
├── plugin-registry.md           # Ships with blueprint (do not delete)
├── frontend-conventions.md      # Add: frontend domain boundaries, design system, quality gates
├── backend-conventions.md       # Add: backend domain, API format, architecture patterns
├── e2e-durability.md            # Add: E2E test reliability rules, banned patterns
├── ci-integration.md            # Add: CI/CD pipeline patterns, Docker Compose setup
└── {your-guide}.md              # Add: any project-specific architectural guidance
```

**For LLM agents creating guides:** These files are read by `/prime` automatically (all `context/**/*.md` files are loaded). Write them as actionable instructions, not abstract principles. Include specific directory paths, file patterns, code examples, and quality gate criteria. Subagents receive context through task descriptions that reference these files, so make each guide self-contained.

---

## Step 6: Set Up Application Code

Create your project's source directories. Examples:

```bash
# Monorepo (web app)
mkdir -p apps/api/src apps/web/src packages/shared-types/src

# Single app
mkdir -p src/ tests/

# CLI / library
mkdir -p src/ tests/ examples/

# Microservices
mkdir -p services/auth/src services/billing/src packages/shared/
```

---

## Step 7: Create Your First Track

Tracks are the unit of work in this blueprint. A track has a spec (WHAT to build) and a plan (HOW to build it), stored in `conductor/tracks/{track-id}/`.

### Track ID Format

Conductor uses the format: `{shortname}_{YYYYMMDD}`

Examples: `user-auth_20250115`, `dashboard_20250220`, `fix-upload-timeout_20260228`

### Option A: Create via Conductor (recommended for interactive sessions)

```bash
/conductor:new-track
```

Follow the interactive prompts to define requirements. Conductor generates `spec.md` and `plan.md` in the correct directory.

### Option B: Create via /implement-prompt (recommended for LLM agents)

```bash
/implement-prompt Build a notification system for the dashboard
```

This runs gap analysis, generates spec + plan as a Conductor track, and then implements it. Best when an LLM agent has a clear description of what to build.

### Option C: Create manually (for LLM agents or developers scripting track creation)

LLM agents that need to create tracks programmatically (e.g., from a product roadmap or issue list) can create them directly without using `/conductor:new-track`:

```bash
# 1. Create the track directory
mkdir -p conductor/tracks/{track-id}

# 2. Create spec.md (WHAT — requirements)
# 3. Create plan.md (HOW — phases and tasks)
# 4. Register in conductor/tracks.md
```

**spec.md template** (follow Conductor's format):

```markdown
# {Track Title}

## Description
{What this track builds and why}

## Scope
{Classification: new feature / enhancement / bug fix / refactor / infrastructure}

## Functional Requirements

- **FR-1:** {Requirement description}
- **FR-2:** {Requirement description}
- **FR-3:** {Requirement description}

## Non-Functional Requirements

- **NFR-1:** {Performance, security, or quality requirement}

## MoSCoW Prioritization

- **Must:** {Critical requirements}
- **Should:** {Important but not blocking}
- **Could:** {Nice-to-have}
- **Won't:** {Explicitly out of scope}

## Dependencies

- {Other tracks or services this depends on}

## Risks

- {Known risks and mitigations}
```

**plan.md template** (follow Conductor's format):

```markdown
# Implementation Plan: {Track Title}

## Progress Summary

| Phase | Status | Description |
|-------|--------|-------------|
| 1 | Not Started | {Phase 1 description} |
| 2 | Not Started | {Phase 2 description} |
| 3 | Not Started | {Phase 3 description} |

## Phase 1: {Phase Title}

- [ ] Task 1.1: {Description} → {files to create/modify}
- [ ] Task 1.2: {Description} → {files to create/modify}
- [ ] Task 1.3: Write tests for Phase 1

### Verification
- [ ] All Phase 1 tests pass
- [ ] Lint + typecheck pass

## Phase 2: {Phase Title}

- [ ] Task 2.1: {Description} → {files to create/modify}
- [ ] Task 2.2: {Description} → {files to create/modify}
- [ ] Task 2.3: Write tests for Phase 2

### Verification
- [ ] All Phase 2 tests pass
- [ ] Lint + typecheck pass

## Phase 3: {Phase Title}

- [ ] Task 3.1: {Description} → {files to create/modify}
- [ ] Task 3.2: Integration tests
- [ ] Task 3.3: Final verification

### Final Verification
- [ ] All tests pass (unit + integration)
- [ ] Lint + typecheck pass
- [ ] Build succeeds
```

**Register in conductor/tracks.md:**

Add a line to `conductor/tracks.md`:

```markdown
- [ ] {track-id} — {Track Title}
```

Track status markers: `[ ]` = not started, `[~]` = in progress, `[x]` = complete

---

## Step 8: Start Building with Claude Code

### Every session:

```bash
/prime                                    # Load context + verify plugins
```

### Implementation options:

```bash
# Option A: Issue Intake (recommended)
# Create a GitHub Issue using story/bug/epic template
/accept #1                               # Accept issue → create track(s)
# Resolve any [NEEDS CLARIFICATION] markers in spec.md
/review-specs {track-id}                 # Validate spec/plan → writes review.md
# Review specs in conductor/tracks/
/implement-track {track-id}              # Implement a single track (requires passing review)
/implement-prd {epic-slug}              # Or implement all tracks from an epic

# Option B: Track-based (structured)
/conductor:new-track                     # Create track interactively
/implement-track {track-id}              # Full workflow: worktree → implement → validate → review → PR

# Option C: Ad-hoc from description
/implement-prompt Build a user dashboard  # Gap analysis → create track → implement

# Option D: Agent Teams (complex features)
/team-feature "Add OAuth2 authentication" --plan-first
```

### Monitoring:

```bash
/conductor:status                        # Project progress overview
/team-status                             # Agent team progress (when teams active)
```

### Undoing work:

```bash
/conductor:revert                        # Git-aware undo by track/phase/task
```

---

## Step 9: Verification Checklist

Before starting feature work, verify:

- [ ] All `{PLACEHOLDER}` tokens replaced in all files
- [ ] `context/product-vision.md` filled in with thorough product definition
- [ ] `context/tech-stack.md` filled in (technology names match plugin-registry keywords)
- [ ] `context/conventions.md` reviewed and customized
- [ ] `context/principles.md` populated with 3-5 project-specific architectural principles
- [ ] `CLAUDE.md` validation commands filled in (actual lint/test/build commands)
- [ ] `wshobson/agents` marketplace registered
- [ ] Core plugins installed (conductor, agent-teams, developer-essentials, etc.)
- [ ] Tech-stack plugins installed (e.g., javascript-typescript, backend-development)
- [ ] `/conductor:setup` run successfully
- [ ] `/setup-project` run successfully (GitHub Project, labels, issue templates)
- [ ] `/prime` succeeds with all plugins showing ✅ Enabled
- [ ] First track created via `/accept #NNN`, `/conductor:new-track`, `/implement-prompt`, or manually

---

## Troubleshooting

See `TROUBLESHOOTING.md` for the complete guide covering: Ralph Loop failures, worktree recovery, bad specs, plugin issues, GitHub sync failures, context compaction recovery, and more.

---

*Last updated: {DATE}*
