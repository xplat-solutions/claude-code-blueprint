# /implement-prd

> Batch-implement all tracks from an epic/PRD using dependency-aware chained worktrees. Each track branches from its parent track's remote branch (not from main), preserving the dependency chain.

## Usage

```
/implement-prd {epic-slug}
/implement-prd {epic-slug} --supervised
/implement-prd {epic-slug} --dry-run
```

---

## Prerequisites

1. **`/prime` ran this session** — Session UUID must be in conversation memory.
2. **Tracks exist** — The epic's tracks must be registered in `conductor/tracks.md` and have `spec.md` + `plan.md`.
3. **Remote is accessible** — `git push` must work (for chaining worktrees from remote branches).

---

## Arguments

```
/implement-prd {epic-slug} [options]

epic-slug:  The epic identifier. Matches tracks in conductor/tracks.md
            that belong to this epic group.
            Can also be a GitHub Issue number (e.g., #42) — resolves to
            the epic slug from the PRD comment on that issue.
```

## Options

| Flag | Effect |
|------|--------|
| `--supervised` | Passed through to each `/implement-track` call |
| `--dry-run` | Show execution plan without implementing |
| `--skip {track-id}` | Skip a specific track (e.g., already implemented) |
| `--start {track-id}` | Start from a specific track (skip earlier ones) |

---

## Process Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    /implement-prd {epic-slug}                             │
│                                                                          │
│   Step 1: RESOLVE TRACKS                                                 │
│   ├── Find all tracks in conductor/tracks.md for this epic              │
│   ├── Read each track's spec.md for dependencies                        │
│   └── Build dependency graph                                             │
│                                                                          │
│   Step 2: PLAN EXECUTION ORDER                                           │
│   ├── Topological sort of dependency graph                              │
│   ├── Identify independent tracks (can run first, branch from main)     │
│   ├── Identify dependent tracks (branch from parent's remote branch)    │
│   └── Display execution plan                                             │
│                                                                          │
│   Step 3: IMPLEMENT SEQUENTIALLY                                         │
│   ├── For each track in order:                                           │
│   │   ├── Determine base branch:                                        │
│   │   │   ├── Independent → main                                        │
│   │   │   └── Dependent → ai/track-{parent-track-id} (remote)          │
│   │   ├── /implement-track {track-id} --base {base-branch}             │
│   │   ├── If success: continue to next track                            │
│   │   ├── If fail: mark blocked, skip dependents, continue independents │
│   │   └── Update GitHub Issue status if linked                          │
│   └── Final summary                                                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Resolve Tracks

### Find Epic Tracks

Read `conductor/tracks.md` and find the epic group:

```markdown
## Epic: {Feature Title} (gh#{issue-number})

- [ ] {track-id-1} — {Story 1 Title}
- [ ] {track-id-2} — {Story 2 Title} (depends: {track-id-1})
- [ ] {track-id-3} — {Story 3 Title}
```

Collect all track IDs belonging to this epic.

If no tracks found:

```
❌ No tracks found for epic: {epic-slug}

Check conductor/tracks.md for an epic group matching this slug.
Did you run /accept first?
```

### Build Dependency Graph

For each track, read `conductor/tracks/{track-id}/spec.md` and extract:

- **Dependencies** section — lists other track IDs this depends on
- **depends:** notation in `conductor/tracks.md`

Build a directed acyclic graph (DAG):

```
track-1 ──→ track-2 ──→ track-4
                    └──→ track-5
track-3 (independent)
```

### Detect Cycles

If the dependency graph has cycles:

```
❌ Circular dependency detected in epic {epic-slug}:

  {track-a} → {track-b} → {track-a}

Fix the dependencies in the track spec.md files before proceeding.
```

**Stop here.**

---

## Step 2: Plan Execution Order

### Topological Sort

Sort tracks in dependency order. Independent tracks come first, then dependent tracks in topological order.

### Determine Base Branches

| Track Type | Base Branch | Why |
|------------|-------------|-----|
| Independent (no dependencies) | `main` | Standard starting point |
| Dependent (has parent) | `ai/track-{parent-track-id}` | Branches from parent's remote branch to inherit its code |

### Display Execution Plan

```
📋 Implementation Plan: {epic-slug}

Tracks: {N} total

| Order | Track | Base Branch | Dependencies |
|-------|-------|-------------|-------------|
| 1 | {track-id-1} | main | — |
| 2 | {track-id-3} | main | — |
| 3 | {track-id-2} | ai/track-{track-id-1} | {track-id-1} |
| 4 | {track-id-4} | ai/track-{track-id-2} | {track-id-2} |

Proceed? [Y/n]
```

If `--dry-run`:

```
(dry run — no changes made)
```

**Stop here for dry-run.**

---

## Step 3: Implement Sequentially

### For Each Track (in order)

#### Determine Base Branch

```
base_branch = "main"                              # if no dependencies
base_branch = "ai/track-{parent-track-id}"       # if has parent dependency
```

For dependent tracks, the parent's branch must exist on the remote. If the parent track was just implemented in a prior step, its branch was pushed to remote by `/implement-track`.

#### Verify Base Branch Exists

```bash
git ls-remote --heads origin {base-branch}
```

If the base branch doesn't exist on the remote:

```
❌ Base branch not found: {base-branch}

This track depends on {parent-track-id}, but its branch hasn't been pushed.
Ensure /implement-track completed successfully for the parent track.
```

Mark this track and all its dependents as **blocked**. Continue with other independent tracks.

#### Call /implement-track

```
/implement-track {track-id} --base {base-branch}
```

The `--base` flag tells `/implement-track` to:

1. Create the worktree from `{base-branch}` instead of `main`
2. Create the PR with `--base {base-branch}` instead of `--base main`

#### Handle Results

**On success:**

```
✅ {track-id} — PR #{pr-number} created (base: {base-branch})
```

Continue to the next track.

**On failure (validation or implementation):**

```
⚠️ {track-id} — FAILED

Marking as blocked. Dependent tracks will be skipped:
  - {dependent-track-1}
  - {dependent-track-2}

Continuing with independent tracks...
```

Mark the failed track and all downstream dependents as **blocked** in `conductor/tracks.md`. Continue implementing any remaining independent tracks.

#### Update GitHub Issue

If the track has a source GitHub Issue:

```bash
# Update is handled by /implement-track's GitHub sync
# /implement-prd only needs to track overall progress
```

---

## Step 4: Final Summary

```
✅ Epic implementation complete: {epic-slug}

| Track | Status | PR | Base |
|-------|--------|----|------|
| {track-id-1} | ✅ Complete | #{pr-1} | main |
| {track-id-3} | ✅ Complete | #{pr-3} | main |
| {track-id-2} | ✅ Complete | #{pr-2} | ai/track-{track-id-1} |
| {track-id-4} | ⚠️ Blocked | — | — |

Completed: {N}/{total}
Blocked: {N} (due to {failed-track-id} failure)

PR merge order (respects dependencies):
  1. #{pr-1} (base: main)          ← merge first
  2. #{pr-3} (base: main)          ← merge anytime
  3. #{pr-2} (base: ai/track-{track-id-1})  ← merge after #{pr-1}
     └── GitHub auto-retargets to main after #{pr-1} merges

Next steps:
  - Review and merge PRs in the order above
  - After merging parent PRs, GitHub retargets child PRs to main
  - For blocked tracks, fix the issue and run:
    /implement-track {blocked-track-id} --base {appropriate-branch}
```

---

## Chained Worktree Strategy

This is the key architectural pattern for dependency-aware implementation:

```
main ─────────────────────────────────────────────────────→
  │
  ├── ai/track-{track-1} ────────────────────→ PR #1 (base: main)
  │     │
  │     └── ai/track-{track-2} ──────────→ PR #2 (base: ai/track-{track-1})
  │           │
  │           └── ai/track-{track-4} ──→ PR #4 (base: ai/track-{track-2})
  │
  └── ai/track-{track-3} ────────────────────→ PR #3 (base: main)
```

**How it works:**

1. Independent tracks branch from `main` (standard)
2. Dependent tracks branch from their parent's remote branch
3. Each PR targets its parent branch, not main
4. When a parent PR is merged into main, GitHub automatically retargets child PRs to main
5. This preserves the full commit history and makes code review natural

**Why not branch everything from main:**

- Dependent tracks need code from their parent track
- If track-2 depends on track-1's API endpoints, branching from main would mean track-2 can't see those endpoints
- Chaining ensures each dependent track starts with all prerequisite code

---

## Error Handling

### No Tracks Found

```
❌ No tracks found for epic: {epic-slug}
Run /accept first to create tracks from the epic.
```

### Circular Dependencies

```
❌ Circular dependency detected
Fix dependencies in spec.md files.
```

### Parent Branch Missing

```
❌ Base branch not found: ai/track-{parent-id}
Ensure parent track was implemented and pushed.
```

### Partial Failure

The command continues past individual track failures, implementing all independent tracks. Blocked tracks are clearly reported in the summary.

### Resume After Failure

```bash
# Fix the failed track
/implement-track {failed-track-id} --base {base-branch}

# Then continue with remaining tracks
/implement-prd {epic-slug} --start {next-track-id}
```

---

## Related Commands

- `/accept {input}` — Accept issues/PRDs into tracks (prerequisite)
- `/implement-track {id}` — Implement a single track (called by this command)
- `/conductor:status` — View all track progress
- `/setup-project` — One-time GitHub project setup
- `/prime` — Load project context (prerequisite)
