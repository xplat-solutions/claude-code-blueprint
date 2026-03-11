# /implement-prompt

> Create a Conductor track from a free-form description, then implement it via `/implement-track`. This is a convenience wrapper — it performs gap analysis, generates spec + plan as a Conductor track, and delegates implementation to the standard track workflow.

## Usage

```
/implement-prompt Build a notification system for the dashboard
/implement-prompt Add rate limiting to all API endpoints --supervised
/implement-prompt Fix file upload timeout --no-worktree
/implement-prompt Refactor background jobs to use queues --skip-questions
/implement-prompt Add dark mode toggle --dry-run
```

---

## Prerequisites

1. **`/prime` ran this session** — The agent must have a prime session UUID in conversation memory. If not, stop and instruct: `Run /prime first.`
2. **Clean working tree** — No uncommitted changes on current branch (unless `--no-worktree`).

---

## Arguments

```
/implement-prompt {description} [options]

description:  Free-form description of what to build, fix, or refactor.
              Can be a single sentence or multiple sentences.
```

## Options

| Flag | Effect |
|------|--------|
| `--skip-questions` | Skip gap analysis interview; agent uses best judgment for all gaps |
| `--dry-run` | Generate spec + plan only, do not implement |
| `--supervised` | Passed through to `/implement-track` → pauses after each phase |
| `--no-worktree` | Passed through to `/implement-track` → skip worktree isolation |
| `--phase N` | Passed through to `/implement-track` → start from phase N |

---

## Process Overview

```
┌────────────────────────────────────────────────────────────────────┐
│              /implement-prompt {description}                       │
│                                                                    │
│   Step 1: ANALYZE (Gap Analysis)                                  │
│   ├── Parse description                                           │
│   ├── Cross-reference with loaded project context                 │
│   ├── Identify affected layers (FE/BE/DB/types/infra)             │
│   ├── Generate up to 10 clarifying questions                      │
│   ├── Present questions ranked by impact                          │
│   └── Await answers (or skip with --skip-questions)               │
│                                                                    │
│   Step 2: CREATE TRACK                                            │
│   ├── Derive track ID: {slug}_{YYYYMMDD}                         │
│   ├── Create conductor/tracks/{track-id}/spec.md                  │
│   ├── Create conductor/tracks/{track-id}/plan.md                  │
│   ├── Register in conductor/tracks.md                             │
│   ├── Display summary                                             │
│   └── If --dry-run: stop here                                     │
│                                                                    │
│   Step 3: IMPLEMENT                                               │
│   └── /implement-track {track-id} [flags pass-through]            │
│       └── (Preflight → Conductor → Ralph Loop → Review → PR)     │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Analyze (Gap Analysis)

### Parse & Cross-Reference

Read the description and cross-reference with loaded project context:

1. **Identify affected areas** — Scan for keywords mapping to project layers:
   - Frontend: UI, component, page, form, modal, dashboard
   - Backend: API, endpoint, service, middleware, route
   - Database: table, column, schema, migration
   - Shared types: interface, type, validation, contract
   - Infrastructure: Docker, CI, script, deployment

2. **Check existing code** — Search the codebase for related files, APIs, types, schemas.

3. **Detect scope** — Classify as: new feature, enhancement, bug fix, refactor, or infrastructure.

### Gap Analysis

Identify genuine gaps in the description. Generate questions only for things the description doesn't already specify clearly.

**Dimensions to evaluate:**

| Dimension | What to Check |
|-----------|--------------|
| Scope | Which packages/directories are affected? |
| Data model | New tables, columns, migrations? |
| API surface | New endpoints, changes to existing? |
| Types | New shared types or contracts? |
| Auth/RBAC | Access control requirements? |
| UI/UX | Components, layout, interactions? |
| Error handling | Failure modes, edge cases? |
| Dependencies | External services or feature dependencies? |

### Present Questions

Up to **10 questions maximum**, ranked by impact:

```
Gap Analysis — {N} questions:

 1. {Highest-impact unknown}
 2. {Second-highest}
 ...

Answer each question, or type "skip" to let the agent decide.
```

**If `--skip-questions`:** Skip the interview. Agent makes best-judgment decisions and documents them in the spec under a "Design Decisions (Agent-Selected)" section.

---

## Step 2: Create Track

### Derive Track ID

Generate a Conductor-compatible track ID from the description:

```
{slug}_{YYYYMMDD}

slug:  kebab-case, 3-5 meaningful words
date:  today's date
```

| Prompt | Track ID |
|--------|----------|
| "Build a notification system for the dashboard" | `notification-system_20260228` |
| "Add dark mode toggle to settings" | `dark-mode-toggle_20260228` |
| "Fix file upload timeout" | `fix-upload-timeout_20260228` |

### Create Track Files

Create the Conductor track directory and files:

```bash
mkdir -p conductor/tracks/{track-id}
```

**`conductor/tracks/{track-id}/spec.md`** — Generate from description + gap analysis answers:

Follow Conductor's spec template format:
- Title, description, scope classification
- Functional Requirements (FR-1, FR-2, ...)
- Non-Functional Requirements (if applicable)
- MoSCoW prioritization (Must/Should/Could/Won't)
- Design Decisions (include agent-selected decisions if `--skip-questions`)
- Dependencies and risks

**`conductor/tracks/{track-id}/plan.md`** — Generate implementation plan:

Follow Conductor's plan template format:
- Phase-based with `- [ ]` checkboxes per task
- Task descriptions with file paths and acceptance criteria
- Progress Summary table
- Verification checkpoints per phase

### Register Track

Add the new track to `conductor/tracks.md`:

```markdown
- [ ] {track-id} — {title}
```

### Display Summary & Confirm

```
📋 Track created: {track-id}

Title: {title}
Scope: {classification}
Phases: {N}
Tasks: {N} total

Spec: conductor/tracks/{track-id}/spec.md
Plan: conductor/tracks/{track-id}/plan.md

Proceed with implementation? [Y/n]
```

**If `--dry-run`:** Display the summary and stop. The spec and plan files remain for review.

---

## Step 3: Implement

Delegate to `/implement-track` with the newly created track ID:

```
/implement-track {track-id} [--supervised] [--no-worktree] [--phase N]
```

All flags from the original `/implement-prompt` call are passed through. From here, the standard implement-track flow takes over:

1. **Preflight** — Prime check (already passed), track verification (just created), worktree creation
2. **Implement** — `/conductor:implement {track-id}`
3. **Validate** — Ralph Loop (lint → typecheck → tests → build)
4. **Review** — `/full-review` + `/security-sast` (always)
5. **Push & PR** — `git push` + `gh pr create`

---

## Error Handling

### Prime Not Detected

```
❌ Prime session not detected
Run /prime first to load project context and verify plugins.
```

### Track Creation Fails

If `conductor/tracks/` directory doesn't exist:

```
❌ Conductor not initialized

Run /conductor:setup first to create the conductor/ directory structure.
```

### Implementation Fails

Errors during Step 3 are handled by `/implement-track`'s error handling (see implement-track.md).

---

## Related Commands

- `/prime` — Load context (prerequisite)
- `/implement-track {id}` — Implement from existing Conductor track (called by this wrapper)
- `/conductor:new-track` — Alternative: create track interactively via Conductor Q&A
- `/conductor:status` — View all track progress
- `/conductor:revert` — Git-aware undo by track/phase/task
