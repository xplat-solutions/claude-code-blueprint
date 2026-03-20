# {PROJECT_NAME}

> {DESCRIPTION}

---

## Quick Start

```bash
# 1. Clone and customize
git clone {repo-url}
cd {PROJECT_NAME}
# Replace all {PLACEHOLDER} tokens — see SETUP.md Step 1

# 2. Fill in context layer
# Edit: context/product-vision.md, context/tech-stack.md, CLAUDE.md

# 3. Set up Hobson agents + Claude Code
/plugin marketplace add wshobson/agents
/conductor:setup                         # Initialize project with Conductor
/setup-project                           # Create GitHub Project, labels, issue templates
/prime                                    # Load context + detect plugins
/plugin install {missing-plugins}         # Install plugins flagged by /prime
/prime                                    # Verify all plugins enabled

# 4. Start building (issue intake workflow)
# Create a GitHub Issue using story/bug/epic template
/accept #1                               # Accept issue → create Conductor track(s)
# Resolve any [NEEDS CLARIFICATION] markers in spec.md
/review-specs {track-id}                 # Validate spec/plan → writes review.md (required gate)
/implement-track {track-id}              # Implement a single track
/implement-prd {epic-slug}              # Or implement all tracks from an epic
```

---

## Project Structure

```
{PROJECT_NAME}/
├── .claude/                    # Claude Code config
│   ├── commands/               # Custom commands (prime, implement-track, implement-prompt)
│   └── settings.json           # Plugins, permissions, hooks
├── .github/                    # GitHub Actions CI/CD + issue templates
├── conductor/                  # Conductor artifacts (created by /conductor:setup)
│   ├── product.md              # Product vision & goals
│   ├── tech-stack.md           # Technology preferences
│   ├── workflow.md             # Development practices (TDD, commits)
│   ├── tracks.md               # Master track registry
│   └── tracks/                 # Track specs and plans
│       └── {track-id}/         # spec.md (WHAT) + plan.md (HOW)
├── context/                    # Project context layer (for /prime)
│   ├── product-vision.md       # Problem, users, success criteria
│   ├── tech-stack.md           # Technology decisions + plugin mapping
│   ├── conventions.md          # Code style, commits, API patterns
│   ├── principles.md           # Architectural principles & governance gates
│   ├── decisions/              # Architecture Decision Records
│   └── guides/                 # Plugin registry + project-specific guides
│       └── plugin-registry.md  # Technology → plugin mapping
├── prd/                        # Committed PRD files (from /accept epic intake)
├── src/ or apps/               # Application code (structure varies by project)
├── TROUBLESHOOTING.md          # Error recovery and common failure modes
└── [config files]              # Docker, CI, package manager, etc.
```

> **What's essential on day one?** `CLAUDE.md` (validation commands), `context/tech-stack.md`, and Conductor. Everything else — conventions, principles, GitHub templates, ADRs — adds value but can wait until your second or third track. See SETUP.md Quick Start for the 15-minute path.

---

## Using Claude Code

### Every Session

```bash
/prime                                    # Load context + verify plugins (required first)
```

### Issue Intake (Recommended)

| Step | Command | What Happens |
|------|---------|-------------|
| 1. Create issue | GitHub UI | Use story, bug, or epic template (with Gherkin acceptance criteria) |
| 2. Accept | `/accept #NNN` | Creates Conductor track(s), flags `[NEEDS CLARIFICATION]` markers, updates GitHub status |
| 3. Review specs | `/review-specs {id}` | Validates spec↔plan consistency, coverage, principles alignment. Writes `review.md` |
| 4. Review | Manual | Check `conductor/tracks/{id}/spec.md`, `plan.md`, and `review.md` |
| 5. Implement | `/implement-track {id}` or `/implement-prd {slug}` | Verifies review passed → worktree → implement → validate → review → PR |

### Alternative Approaches

| Approach | Command | When to Use |
|----------|---------|-------------|
| Structured track | `/conductor:new-track` → `/implement-track {id}` | Planned features with spec + plan (no GitHub Issue) |
| Ad-hoc | `/implement-prompt {description}` | Quick features from a description |
| Agent Teams | `/team-feature "description" --plan-first` | Complex cross-cutting features |

### Spec Review & Code Review

| Command | Purpose |
|---------|---------|
| `/review-specs {id}` | Pre-implementation spec/plan consistency analysis (required gate) |
| `/review-specs {id} --epic {slug}` | Batch review all tracks in an epic + cross-track consistency |
| `/full-review` | Multi-perspective code review |
| `/team-review src/ --reviewers security,performance,architecture` | Parallel team review |
| `/security-sast` | SAST security scan |
| `/team-spawn security` | Comprehensive parallel security audit |

### Session Management

| Command | Purpose |
|---------|---------|
| `/handoff` | Create structured handoff document for session transfer |
| `/compact focus on {what}` | In-place context compression (same session continues) |
| `/context-audit` | Analyze token budget across context, plugins, MCP servers |
| `/context-audit --track {id}` | Include track token estimation without loading it |

### Monitoring & Management

| Command | Purpose |
|---------|---------|
| `/conductor:status` | Project progress overview |
| `/team-status` | Agent team progress |
| `/conductor:revert` | Git-aware undo |
| `/conductor:manage` | Archive, restore, delete, rename tracks |

---

## Hobson Agents

This project uses the [wshobson/agents](https://github.com/wshobson/agents) plugin marketplace for Claude Code. Key plugins:

- **Conductor** — Context-Driven Development workflow (setup, tracks, implement, status, revert)
- **Agent Teams** — Multi-agent orchestration (parallel review, debugging, feature development)
- **Developer Essentials** — Git, SQL, debugging, E2E, code review, monorepo skills
- **Full-Stack Orchestration** — End-to-end feature coordination

See `context/guides/plugin-registry.md` for the full technology → plugin mapping and `SETUP.md` for installation instructions.

---

## Customization

See `SETUP.md` for the complete step-by-step setup guide, including:

1. Replacing placeholder tokens
2. Filling in the context layer (product vision, tech stack, conventions)
3. Defining architectural principles (see below)
4. Installing and configuring Hobson plugins
5. Initializing Conductor
6. Setting up GitHub Project (board, labels, issue templates)
7. Adding project-specific guides (optional)
8. Creating your first track

### Maintaining Architectural Principles

The `context/principles.md` file defines your project's immutable development principles. These are enforced by `/review-specs` as governance gates — violations are flagged as CRITICAL and block implementation.

**This file should be actively maintained throughout the project's lifecycle:**

- Start with 3-5 principles that reflect your team's strongest architectural opinions
- Add new principles when you discover recurring quality issues in code reviews or production incidents
- Amend principles via PR when they no longer serve the project — track changes in the Amendment Log
- Each principle must have a clear MUST/SHOULD level, a rationale, and concrete gate checks

The principles complement `context/conventions.md` (which covers code style and patterns) by focusing on *architectural governance* — the high-level decisions that shape how features are designed, not how code is formatted.

---

## Best Practices

Operational tips for humans and agents working with this blueprint.

### Run Parallel Sessions

Any `/implement-prd` or `/implement-track` call that doesn't share a dependency chain with another can run in its own parallel session. Each session is a separate Claude Code instance — a terminal tab, a [Conductor.build](https://conductor.build) workspace, or a cloud session on [claude.ai/code](https://claude.ai/code).

**Prerequisite.** Accept all work items first from a single session so specs and dependency graphs are committed before parallel sessions branch from them:

```bash
# Single session — accept everything
/prime
/accept #10                               # signup epic → 3 tracks
/accept #11                               # billing epic → 4 tracks
/accept #15                               # standalone story → 1 track
/accept #18                               # standalone bug → 1 track
# Review specs in conductor/tracks/, then launch parallel sessions
```

**Running independent pipelines and tracks.** Open one session per pipeline or standalone track. Every session starts with `/prime`:

```bash
# Session 1                               # Session 2
/prime                                     /prime
/implement-prd signup                      /implement-prd billing

# Session 3                               # Session 4
/prime                                     /prime
/implement-track cart-api_20260302         /implement-track fix-nav_20260302
```

Each `/implement-prd` handles its own epic's dependency graph internally — it runs tracks in topological order, chaining worktrees so dependent tracks branch from their parent's remote branch. Each `/implement-track` runs a single track end-to-end (worktree, implement, Ralph Loop, review, PR). No coordination between sessions is needed because the work items don't overlap.

**Parallelizing tracks within a single epic.** `/implement-prd` runs tracks sequentially within one session. To go faster, skip `/implement-prd` and dispatch independent tracks from the same epic to separate sessions:

```bash
# After /accept #10 created: signup-auth_20260302, signup-profile_20260302 (depends: auth), signup-onboard_20260302

# Session A — independent track           # Session B — independent track
/prime                                     /prime
/implement-track signup-auth_20260302      /implement-track signup-onboard_20260302

# Session C — starts after Session A pushes its branch
/prime
/implement-track signup-profile_20260302 --base ai/track-signup-auth_20260302
```

Check `conductor/tracks.md` for the dependency notation (`depends: {track-id}`). Tracks with no dependencies run immediately. Dependent tracks wait for their parent to push, then use `--base ai/track-{parent-id}` so the worktree branches from the parent's code and the PR targets the parent's branch. GitHub auto-retargets the PR to `main` when the parent merges.

**PR merge order.** After all sessions finish, merge PRs respecting the dependency chain. Independent PRs merge in any order. Chained PRs merge parent-first — GitHub retargets each child PR to `main` automatically after its parent merges.

### Use Plan Mode First

For any non-trivial work, start in Plan mode (`shift+tab` twice). Iterate on the plan until it's solid, then switch to auto-accept edits mode for execution. A good plan lets Claude one-shot the implementation. This applies to ad-hoc work — the issue intake workflow (`/accept` → `/implement-track`) already produces structured specs and plans.

### Compound CLAUDE.md with Mistake-to-Rule

This is the single highest-leverage habit for improving Claude's output over time. Every time Claude makes a mistake, turn the correction into a permanent rule:

**In-session:** After correcting Claude, end your prompt with: *"update CLAUDE.md so you don't make that mistake again."* Claude will add a terse one-liner rule.

**In code review:** Tag `@claude` on PRs to update `CLAUDE.md` as part of the PR. Example from Boris Cherny's team: `@claude add to CLAUDE.md to never use enums, always prefer literal unions`. Claude reads the CLAUDE.md, updates the relevant line, and commits — all in under a minute. Install the Claude Code GitHub Action (`/install-github-action`) to enable this.

**The compound effect:** A team contributing corrections multiple times per week will see Claude's mistake rate drop from ~1 correction every 3-4 interactions to ~1 every 8-10 within days. Over a month, CLAUDE.md becomes a high-density knowledge base of project-specific patterns.

**Size discipline:** Keep CLAUDE.md under ~500 tokens of actual rules. Remove anything Claude already knows from training (standard language conventions, common patterns). Only keep project-specific corrections and commands. If Claude ignores a rule, the file is too long and the rule is getting lost — prune aggressively. Detailed conventions belong in `context/conventions.md`, not here.

### Add a PostToolUse Formatter Hook

Add a `PostToolUse` hook to `.claude/settings.json` so every file Claude writes gets auto-formatted. This prevents CI formatting failures with zero ongoing effort:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "{your format command} || true"
          }
        ]
      }
    ]
  }
}
```

Replace `{your format command}` with your project's formatter (e.g., `bun run format`, `npx prettier --write $CLAUDE_FILE`, `black $CLAUDE_FILE`). The `|| true` ensures formatting failures don't block Claude's workflow.

### Add a PreToolUse Hook for Git Push Review

For ad-hoc work outside the `/implement-track` pipeline (which has built-in review via Ralph Loop + `/full-review`), add a `PreToolUse` hook that reminds you to review changes before pushing:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git push *)",
        "hooks": [
          {
            "type": "command",
            "command": "echo '⚠️ Review staged changes before pushing' && git diff --stat HEAD~1 >&2"
          }
        ]
      }
    ]
  }
}
```

This is a safety net for work that bypasses the track workflow. The hook shows a diff summary before every push so nothing slips through unreviewed.

### Manage MCP Context Costs

Every MCP server registers tool definitions that consume context window tokens — with 80+ active tools, up to 72% of context can be consumed before you type a single message. Tool *outputs* compound this further: a single Playwright snapshot costs 56 KB, 20 GitHub issues cost 59 KB. This directly reduces session length and implementation quality.

Mitigations:

- **Keep it lean** — Configure MCP servers in `.mcp.json` (checked into git). Aim for under 10 active servers. Disable unused ones via `disabledMcpServers` in `.claude/settings.json`.
- **Install context-mode** — The `mksglu/claude-context-mode` plugin compresses MCP tool outputs by 95-99%, extending sessions from ~30 minutes to ~3 hours. Install via `/plugin marketplace add mksglu/claude-context-mode`. Recommended for projects with 3+ MCP servers.
- **Batch operations** — When querying external tools, prefer batch calls over many small ones to reduce per-call overhead.

### Search Before Building

Before writing custom code for a non-trivial problem, search for existing solutions: project-internal patterns first, then installed dependencies, then well-maintained libraries. If an existing solution covers ≥80% of the requirement, prefer it over custom code. This is codified as a SHOULD-level principle in `context/principles.md` and documented in `context/conventions.md`. AI agents naturally gravitate toward writing new code — this convention counteracts that bias.

### Audit Your Context Budget

Run `/context-audit` after `/prime` to see where your tokens are going. The report breaks down consumption across always-loaded context, guides (loaded vs scouted), plugins, MCP servers, and optionally a specific track. Use it to identify bloat — an unused plugin consuming 4k tokens or a guide loaded in full that should be scouted. Add `--track {id}` to estimate how much runway you'll have before starting implementation on a specific track.

### Leverage Guide Scouting

`/prime` automatically scouts `context/guides/` — when there are 4+ guide files, it reads only the title and description of each guide rather than loading them in full. The agent then loads relevant guides just-in-time when it starts working in a specific area. This keeps the initial context window lean while still making all project knowledge discoverable.

To make scouting effective, every guide file in `context/guides/` should start with a clear title and one-line description:

```markdown
# Frontend Conventions

> React component patterns, design system usage, and accessibility requirements. Load when working on UI components.
```

If a guide doesn't have this opening format, the preview in `/prime` will be less useful and the agent may not recognize when to load it.

### Keep Permissions Explicit

Don't use `--dangerously-skip-permissions`. Instead, pre-allow safe commands in `.claude/settings.json` (already configured in this blueprint). When your project adds new tools or commands, add them to the `allow` list and commit — the whole team benefits.

### Use `/handoff` for Session Continuity

Long sessions degrade. When context has been compacted once and the agent starts losing track of decisions or repeating mistakes, run `/handoff` instead of compacting again. This creates a structured handoff document at `conductor/handoffs/` that captures track state, key decisions, failed approaches, and concrete next steps — then you start a fresh session with `/prime` and point it at the handoff file.

The rule of thumb: `/compact` once is fine. `/compact` twice means run `/handoff` and start fresh. A good handoff document lets the next session start productive work within 2-3 turns. A `PreCompact` hook in `.claude/settings.json` fires before every compaction as a reminder to consider `/handoff` first.

```bash
# When the session is getting bloated
/handoff --track {track-id}

# In the new session
/prime
Read conductor/handoffs/{filename} and continue from where the previous session left off.
```

### Compact at Natural Breakpoints

Claude Code auto-compacts at ~95% context capacity, but you get better results compacting manually at logical breakpoints: after `/accept` finishes (before implementation), between tracks in `/implement-prd`, or after a failed approach. Never compact mid-phase or mid-debug. Run `/compact focus on {what matters}` to tell Claude what to preserve. See the Compact Instructions section in `CLAUDE.md` for project-specific preservation rules.

---

### Troubleshooting

See `TROUBLESHOOTING.md` for common failure modes: Ralph Loop failures, worktree issues, bad specs, plugin problems, GitHub sync failures, and context compaction recovery.

---

*Built with the [Blueprint](https://github.com/{org}/blueprint) template for AI-assisted development.*
