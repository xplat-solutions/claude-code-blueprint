# /handoff

> Create a structured handoff document that captures the current session's work, decisions, and next steps — designed to be the sole input to a fresh Claude Code session. Use this when context is bloated, compaction has already happened once, or you're at a natural stopping point and want to continue later.

## Usage

```
/handoff
/handoff --track {track-id}
/handoff --reason "hitting context limits after e2e testing"
```

---

## When to Use

| Situation | Recommended Action |
|-----------|-------------------|
| Context compacted once, still productive | Continue working |
| Context compacted twice or agent hallucinating | **`/handoff`** → fresh session |
| Natural stopping point, will resume later | **`/handoff`** → fresh session |
| Switching from planning to implementation | **`/handoff`** → fresh session (or use the spec directly) |
| Mid-debug with complex state to preserve | **`/handoff`** → fresh session |

**`/handoff` vs `/compact`:** `/compact` summarizes in-place — the same session continues with a compressed history. `/handoff` produces an external artifact for session transfer — you end the current session and start clean. Use `/compact` when you have one compaction left in you. Use `/handoff` when the session is beyond saving or you want a clean break.

---

## Process

### Step 1: Gather Session State

Read the following (silently — do not output raw content):

1. **Git state** — `git log --oneline -20` for recent commits this session
2. **Git diff** — `git diff --stat` for any uncommitted changes
3. **Active track** — If a track is in progress, read `conductor/tracks/{track-id}/plan.md` for phase/task status
4. **Track registry** — Read `conductor/tracks.md` for overall project status
5. **Conversation context** — Review the current conversation for decisions, failed approaches, and open questions

If `--track {track-id}` is provided, focus the handoff on that specific track. Otherwise, capture all work done in the session.

### Step 2: Build Handoff Document

Create the handoff file at:

```
conductor/handoffs/{YYYYMMDD}-{HHmm}-handoff.md
```

Create the `conductor/handoffs/` directory if it doesn't exist.

Use this structure:

```markdown
# Session Handoff — {date} {time}

> {One-line summary of what this session accomplished}

---

## Session Summary

{2-4 sentences describing what was done, what was attempted, and the overall outcome.}

**Reason for handoff:** {Why the session is ending — context limits, natural breakpoint, switching to implementation, etc. Use --reason value if provided.}

---

## Track State

<!-- Include this section only if working on a Conductor track -->

| Field | Value |
|-------|-------|
| Track ID | {track-id} |
| Source Issue | gh#{number} (if linked) |
| Current Phase | {N} of {total} |
| Current Task | {task description} |
| Phase Status | {completed tasks}/{total tasks} in current phase |
| Branch | ai/track-{track-id} |
| Worktree | .worktrees/track-{track-id} (if active) |
| Ralph Loop | {iteration count if in validation, or "not started"} |

### Completed Work

{Bulleted list of what was accomplished — phases completed, features built, tests passing.}

### In Progress

{What was actively being worked on when the session ended. Be specific: file names, function names, test names.}

### Failing Tests (if any)

{Exact error messages and file locations for any failing tests. This is critical for the next session to pick up debugging.}

```
{paste exact error output here}
```

---

## Key Decisions

<!-- Decisions made during this session that aren't yet captured in context/decisions/ or the spec/plan -->

1. **{Decision}** — {Rationale}. {Alternative that was rejected and why.}
2. **{Decision}** — {Rationale}.

---

## Failed Approaches

<!-- Approaches that were tried and didn't work — saves the next session from repeating them -->

1. **{What was tried}** — {Why it failed}. {What to do instead.}

---

## Unresolved Issues

<!-- Blockers, open questions, things that need human input -->

- {Issue or question}
- {Issue or question}

---

## Next Steps

<!-- Concrete actions for the next session, in priority order -->

1. {First thing to do}
2. {Second thing to do}
3. {Third thing to do}

---

## Files Modified

<!-- Files changed in this session — helps the next session understand the blast radius -->

| File | Change |
|------|--------|
| {path} | {brief description} |

---

## Context to Load

<!-- Which context files the next session should read beyond what /prime loads -->

- `context/guides/{file}` — {why it's needed}
- `conductor/tracks/{track-id}/spec.md` — {track requirements}
- `conductor/tracks/{track-id}/plan.md` — {implementation plan with progress}

---

## Resume Command

```bash
# Start a new session, then:
/prime
# Then paste:
Read conductor/handoffs/{this-file-name} and continue from where the previous session left off.
```
```

### Step 3: Commit the Handoff

```bash
# Create directory if needed
mkdir -p conductor/handoffs

# Stage and commit
git add conductor/handoffs/{filename}
git commit -m "chore: session handoff — {one-line summary}"
```

If there are uncommitted work-in-progress changes, commit those first with a `wip:` prefix before creating the handoff commit:

```bash
git add -A
git commit -m "wip(track-{id}): {description of uncommitted work}"
git add conductor/handoffs/{filename}
git commit -m "chore: session handoff — {one-line summary}"
```

### Step 4: Output Summary

```
📋 Handoff created: conductor/handoffs/{filename}

Summary: {one-line summary}
Track: {track-id} (phase {N}/{total})     ← if applicable
Commits this session: {N}
Uncommitted changes: {committed as wip / none}

Resume in a new session:
  /prime
  Read conductor/handoffs/{filename} and continue from where the previous session left off.
```

---

## Handoff Quality Checklist

Before finalizing, verify the handoff document includes:

- [ ] Specific file paths and function names (not vague references)
- [ ] Exact error messages for any failing tests (copy-pasted, not paraphrased)
- [ ] Decisions with rationale (not just "we decided X")
- [ ] Failed approaches (so the next session doesn't repeat them)
- [ ] Concrete next steps (not "continue implementation")
- [ ] Context files to load (beyond what `/prime` provides)

A good handoff document should let the next session start productive work within its first 2-3 turns.

---

## Arguments

| Flag | Effect |
|------|--------|
| `--track {track-id}` | Focus the handoff on a specific track |
| `--reason "..."` | Include a specific reason for the handoff |

---

## Error Handling

### No Work Done

If no commits were made and no track is in progress:

```
⚠️ No work detected in this session.

A handoff document is most useful when there's session state to transfer.
If you want to save planning context or decisions, proceed anyway? [Y/n]
```

### Uncommitted Changes

If `git diff` shows uncommitted changes, commit them as WIP before creating the handoff (see Step 3).

---

## Related Commands

- `/compact` — In-place context compression (same session continues)
- `/prime` — Load context at session start (run first in the new session)
- `/implement-track {id} --phase N` — Resume implementation from a specific phase
- `/conductor:status` — View track progress after resuming
