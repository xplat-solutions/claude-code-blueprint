# {PROJECT_NAME}

<!-- SIZE DISCIPLINE: This file should contain ~500 tokens of actual rules after setup.
     Delete every placeholder section you don't use. Remove anything Claude already knows
     (standard language conventions, common patterns). Keep ONLY: project-specific rules,
     validation commands, and learned corrections.

     Boris Cherny (Claude Code creator) keeps his CLAUDE.md to ~150 lines of terse rules.
     If Claude keeps ignoring a rule, the file is probably too long and the rule is getting lost.

     WHAT BELONGS HERE: One-liner rules, actual commands, project-specific corrections.
     WHAT BELONGS ELSEWHERE:
       - Code conventions → context/conventions.md
       - Architecture principles → context/principles.md
       - Product vision → context/product-vision.md
       - Tech stack details → context/tech-stack.md
       - Plugin mapping → context/guides/plugin-registry.md
       - Workflow details → README.md
       - Setup instructions → SETUP.md
       - Agent coordination → AGENTS.md
     All context/ files are loaded by /prime. Don't duplicate their content here. -->

> Session configuration for Claude Code. Run `/prime` at the start of every session.

**{PROJECT_NAME}** — {DESCRIPTION}

---

## Validation Commands

<!-- Replace with your project's actual commands. This is the most important section. -->

```bash
# Lint + typecheck
{your lint command}
{your typecheck command}

# Tests
{your test command}

# Build
{your build command}

# E2E (full stack required)
{your e2e command}

# Full CI
{your ci script}
```

---

## Design System

<!-- Delete this section if your project has no UI. Fill in tokens if it does. -->

| Token | Value |
|-------|-------|
| Background (dark) | {color} |
| Background (light) | {color} |
| Primary accent | {color} |
| Font (body) | {font} |
| Font (code) | {font} |

---

## Project Rules

<!-- This is where mistake-to-rule corrections accumulate. Start empty.
     After every correction, end your prompt with:
     "update CLAUDE.md so you don't make that mistake again."
     Each entry should be a terse one-liner. Examples:

     - Always use named exports, never default exports
     - Test files must use the async wrapper from test/setup.ts
     - Import from @/lib, not relative paths
     - API routes return { data, error } shape, never raw values
     - Use Zod for all request validation, not manual checks
-->

---

## Compact Instructions

<!-- Re-read from disk after every compaction. Tells Claude what to preserve. -->

When context compacts, preserve:

- **Active track context** — current track ID, phase, task number, and what was just completed
- **Source issue numbers** — any `gh#NNN` references needed for status sync
- **Dependency chain** — which tracks depend on which, and `--base` branch for chained worktrees
- **Failing test details** — if in a Ralph Loop, keep exact error messages and iteration count
- **Architecture decisions** — decisions not yet written to `context/decisions/`
- **Scouted guides loaded** — which `context/guides/` files were loaded mid-session (beyond what `/prime` previewed)

Summarize aggressively: file contents read for reference (re-read from disk), completed phases (plan.md has status), plugin detection output (re-run /prime).

Compact at natural breakpoints only — after `/accept`, between tracks, after Ralph Loop passes, after a failed approach. Never mid-phase or mid-debug.

```
/compact focus on track {track-id} implementation — preserve current phase, failing tests, and dependency chain
```

**PreCompact hook:** A `PreCompact` hook in `.claude/settings.json` fires before every compaction, reminding the agent to run `/handoff` first. When compaction triggers, evaluate whether a handoff would preserve more state than compaction — especially if compaction has already happened once.

**When to use `/handoff` instead of `/compact`:** If compaction has already happened once in this session and the agent is starting to lose track of context, run `/handoff` to create a structured handoff document and start a fresh session. A second compaction rarely preserves enough detail.

```
/handoff --track {track-id}
```

---

*Last updated: {DATE}*
