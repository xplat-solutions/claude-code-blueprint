# /implement-track

> Thin orchestration wrapper around `/conductor:implement`. Adds worktree isolation, whole-track validation (Ralph Loop), Hobson code review + security scanning, automated PR creation, and GitHub Issue status sync.

## Usage

```
/implement-track {track-id}
/implement-track {track-id} --supervised
/implement-track {track-id} --phase 2
/implement-track {track-id} --no-worktree
/implement-track {track-id} --base ai/track-{parent-id}
```

---

## Prerequisites

1. **`/prime` ran this session** — The agent must have a prime session UUID in conversation memory. If not, stop and instruct: `Run /prime first.`
2. **Conductor track exists** — `conductor/tracks/{track-id}/spec.md` and `conductor/tracks/{track-id}/plan.md` must exist.
3. **Track status is ready** — The track should be in an implementable state in `conductor/tracks.md`.

---

## Arguments

```
/implement-track {track-id} [options]

track-id:  The Conductor track ID (e.g., user-auth_20250115, dashboard_20250220)
           Format: {shortname}_{YYYYMMDD}
```

## Options

| Flag | Effect |
|------|--------|
| `--supervised` | Passed through to `/conductor:implement` — pause after each phase |
| `--phase N` | Passed through to `/conductor:implement` — start from phase N |
| `--no-worktree` | Skip worktree isolation, work directly on current branch (hot-fixes only) |
| `--base {branch}` | Branch the worktree from this branch instead of `main`. Used by `/implement-prd` for chained worktrees. Also sets the PR base branch. |
| `--dry-run` | Show what would happen without executing |

---

## Process Overview

```
┌────────────────────────────────────────────────────────────────────┐
│              /implement-track {track-id}                           │
│                                                                    │
│   Step 1: PREFLIGHT                                               │
│   ├── Verify prime session UUID in conversation memory            │
│   ├── Verify conductor/tracks/{track-id}/spec.md exists           │
│   ├── Verify conductor/tracks/{track-id}/plan.md exists           │
│   ├── Check spec.md for "Source: gh#NNN"                          │
│   │   ├── If found: extract issue number, store for Step 5        │
│   │   └── Update GitHub Projects status → "Implementing"          │
│   ├── Display track summary (title, phases, tasks)                │
│   └── Create git worktree (unless --no-worktree)                  │
│       └── Base: --base {branch} or main (default)                 │
│                                                                    │
│   Step 2: IMPLEMENT (Conductor pass-through)                      │
│   └── /conductor:implement {track-id} [--supervised] [--phase N]  │
│       └── Conductor handles: TDD, task execution, commits,        │
│           phase checkpoints, plan.md progress updates              │
│                                                                    │
│   Step 3: VALIDATE (Ralph Loop)                                   │
│   ├── Run: lint → typecheck → tests → build                      │
│   ├── If fail: analyze → fix → commit fix → retry                 │
│   └── Loop until pass (max 10 iterations)                         │
│                                                                    │
│   Step 4: REVIEW                                                  │
│   ├── /full-review (multi-perspective code review)                │
│   ├── /security-sast (SAST security scan — ALWAYS)                │
│   ├── Fix any critical/high findings                              │
│   └── Re-run Ralph Loop if fixes were made                        │
│                                                                    │
│   Step 5: PUSH & PR                                               │
│   ├── git push -u origin {branch}                                 │
│   ├── gh pr create --base {base-branch} --head {branch}           │
│   │   └── base-branch: --base flag value, or main                 │
│   ├── If source issue exists:                                     │
│   │   ├── PR body includes "Closes #{issue-number}"               │
│   │   ├── Update GitHub Projects status → "In Review"             │
│   │   └── Comment on issue: "PR created: {pr-url}"                │
│   ├── Update conductor/tracks.md status                           │
│   ├── Clean up worktree                                           │
│   └── Display summary                                             │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Preflight

### Verify Prime Session

Check that a prime session UUID exists in conversation memory (set by `/prime` earlier in this session).

If not found:

```
❌ Prime session not detected

Run /prime first to load project context and verify plugins.
```

**Stop here.** Do not proceed without a primed session.

### Verify Track Exists

Check that the Conductor track directory and required files exist:

```bash
conductor/tracks/{track-id}/spec.md    # WHAT — requirements
conductor/tracks/{track-id}/plan.md    # HOW — phases, tasks
```

If missing:

```
❌ Track not found: conductor/tracks/{track-id}/

Check:
- Directory exists: ls conductor/tracks/
- Both spec.md and plan.md exist in the track directory

Create a track first:
  /conductor:new-track                    (interactive)
  /accept #NNN                            (from GitHub Issue)
  Or manually create conductor/tracks/{track-id}/ with spec.md + plan.md
```

### Check for Source GitHub Issue

Read `spec.md` and look for a `Source: gh#NNN` line. If found:

1. Extract the issue number
2. Store it for use in Step 5 (PR creation and status sync)
3. Update the GitHub Issue status to "Implementing":

```bash
# Add "implementing" label (remove "accepted" if present)
gh issue edit {number} --add-label "implementing" --remove-label "accepted"
```

If the `gh` command fails or the issue doesn't exist, log a warning but continue — GitHub sync is best-effort.

### Display Track Summary

Read `spec.md` and `plan.md`, then display:

```
📋 Track: {track-id} — {Title}

Source: gh#{issue-number}          (if linked)
Base: {base-branch}                (main or --base value)
Phases: {N}
Tasks: {N} total
Files to create/modify: {estimated}

Proceed? [Y/n]
```

### Create Git Worktree

Unless `--no-worktree` is specified:

```bash
# Determine base branch
base_branch="${base_flag:-main}"

# Create worktree from the base branch
git worktree add -b "ai/track-{track-id}" ".worktrees/track-{track-id}" "$base_branch"
cd .worktrees/track-{track-id}
# [run your package manager install command]
```

If `--no-worktree`, work on the current branch directly.

---

## Step 2: Implement (Conductor Pass-Through)

Delegate entirely to Conductor's implementation engine:

```
/conductor:implement {track-id}
```

Pass through any flags:
- `--supervised` → Conductor pauses after each phase for approval
- `--phase N` → Conductor starts from phase N

**What Conductor handles (do NOT reimplement):**
- TDD red-green-refactor workflow per task
- Task execution order and dependencies
- Commit creation with semantic messages
- Phase checkpoints and progress tracking
- `plan.md` task status updates (`- [ ]` → `- [x]`)

**What this wrapper does NOT do:**
- Zero custom implementation logic
- Zero file-reading duplication (Conductor reads spec/plan)
- Zero custom task execution

Wait for Conductor to complete all phases before proceeding.

---

## Step 3: Validate (Ralph Loop)

After Conductor finishes implementation, run whole-track validation. This catches cross-phase regressions that per-phase checks might miss (e.g., Phase 3 code breaking Phase 1 tests).

### Run Validation

```bash
# Level 1: Static Analysis
{your lint command}
{your typecheck command}

# Level 2: Tests
{your test command}

# Level 3: Build
{your build command}
```

Replace `{your ... command}` with the actual commands from `CLAUDE.md` Validation Commands section.

### On Success

```
Ralph Loop — track {track-id}

  Level 1: lint + typecheck .... ✅
  Level 2: tests ............... ✅ ({N} passed)
  Level 3: build ............... ✅

✅ All levels passed (iteration 1/10)
```

### On Failure

```
Level 2: tests ............... ❌ {N} passed, {M} failed

Analyzing failures...
  - {test-file}: {error description}

Fixing... (iteration {i}/10)
```

Fix → Commit fix (`fix(track-{id}): {description}`) → Retry from Level 1 → Loop.

**Max 10 iterations.** If validation still fails after 10 attempts:

```
⚠️ Validation failed after 10 iterations

Track status: blocked
Please review the failing tests and provide guidance.
```

Stop here. Do not proceed to Review or Push.

---

## Step 4: Review

After Ralph Loop passes, run Hobson's review and security scanning.

### Code Review

```
/full-review
```

This runs a multi-perspective code review (architecture, best practices, security surface).

### Security Scan (ALWAYS)

```
/security-sast
```

**This is mandatory for every track.** Not conditional on project settings or track type. Security scanning runs on every implementation.

### Handle Findings

If `/full-review` or `/security-sast` report critical or high severity findings:

1. Fix the issues
2. Commit fixes: `fix(track-{id}): address review/security findings`
3. Re-run Ralph Loop (Step 3) to ensure fixes don't introduce regressions
4. Re-run the review tool that flagged the issue to verify resolution

If only low/medium findings remain, note them in the PR description but proceed.

---

## Step 5: Push & PR

### Determine Base Branch for PR

```
base_branch = "--base flag value" or "main" (default)
```

This is critical for chained worktrees: dependent tracks target their parent track's branch, not main.

### Push to Remote

```bash
git push -u origin "ai/track-{track-id}"
```

### Create Pull Request

```bash
# Build PR body
pr_body="## Track: {track-id}

### Summary
{brief description from spec.md}

### Phases Completed
{list of phases with status}

### Validation
- Lint + Typecheck: ✅
- Tests: ✅ ({N} passed)
- Build: ✅
- Code Review: ✅ /full-review
- Security Scan: ✅ /security-sast
- Ralph Loop iterations: {N}

### Review Notes
{any low/medium findings noted}"

# Add "Closes #NNN" if source issue exists
if [source_issue]; then
  pr_body+="

Closes #{source-issue-number}"
fi

gh pr create \
  --base "{base-branch}" \
  --head "ai/track-{track-id}" \
  --title "feat: {track title}" \
  --body "$pr_body"
```

If `gh` is not available:

```
⚠️ gh CLI not available — push completed but PR must be created manually.

Branch pushed: ai/track-{track-id}
Create PR: https://github.com/{org}/{repo}/compare/ai/track-{track-id}
```

### Update GitHub Issue (if linked)

If `spec.md` contained `Source: gh#NNN`:

```bash
# Comment on the issue with PR link
gh issue comment {number} --body "🚀 PR created: {pr-url}

Track: \`{track-id}\`
Branch: \`ai/track-{track-id}\`
Base: \`{base-branch}\`"

# Update labels
gh issue edit {number} --remove-label "implementing"
# Note: "Closes #NNN" in PR body auto-closes the issue on merge
```

Update GitHub Projects status to "In Review" (best-effort, warn if fails).

### Update Conductor Registry

Update `conductor/tracks.md` to reflect track status (e.g., mark as `[~]` for in-review or `[x]` for complete depending on Conductor's convention).

### Clean Up Worktree

```bash
cd <repo-root>
git worktree remove ".worktrees/track-{track-id}"
```

### Display Summary

```
✅ Track {track-id} — PR created

Summary:
  Phases completed: {N}/{N}
  Commits: {N}
  Tests: {N} passed
  Ralph Loop iterations: {N}
  Review: ✅ /full-review + /security-sast
  PR: {pr-url}
  Base: {base-branch}
  Source issue: gh#{number}         (if linked)
  Duration: ~{N} minutes

The PR is ready for human review and merge.
Next suggested track: {next-track-id}
```

---

## Error Handling

### Prime Not Detected

```
❌ Prime session not detected
Run /prime first to load project context and verify plugins.
```

### Track Not Found

```
❌ Track not found: conductor/tracks/{track-id}/
Create a track first via /conductor:new-track, /accept, or manually.
```

### Base Branch Not Found

```
❌ Base branch not found: {base-branch}

The --base branch must exist locally or on the remote.
For chained worktrees, ensure the parent track was implemented and pushed first.
```

### Conductor Failure

If `/conductor:implement` fails or errors out:

```
❌ Conductor implementation failed

Error: {error details}

Options:
  1. Fix the issue and retry: /implement-track {track-id} --phase {last-phase}
  2. Check Conductor status: /conductor:status
  3. Revert Conductor changes: /conductor:revert
```

### Validation Exhausted

```
⚠️ Validation failed after 10 iterations
Track status: blocked
Resume with: /implement-track {track-id} --phase {last-phase}
```

### GitHub Sync Failures

GitHub Issue status updates are best-effort. If `gh` commands fail:

```
⚠️ Could not update GitHub Issue #{number} — {error}
Continuing without GitHub sync. Update the issue manually.
```

---

## Related Commands

- `/prime` — Load context (prerequisite)
- `/accept {input}` — Accept GitHub Issues/PRDs into tracks
- `/implement-prompt {desc}` — Create a track from free-form input, then implement it
- `/implement-prd {slug}` — Batch implement all tracks from an epic (calls this command)
- `/conductor:new-track` — Create track interactively (spec.md + plan.md)
- `/conductor:implement` — TDD implementation engine (called by this wrapper)
- `/conductor:status` — View all track progress
- `/conductor:revert` — Git-aware undo by track/phase/task
- `/conductor:manage` — Archive, restore, delete, rename tracks
- `/full-review` — Multi-perspective code review
- `/security-sast` — SAST security scan
