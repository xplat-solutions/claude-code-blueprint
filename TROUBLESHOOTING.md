# Troubleshooting

> Common failure modes and how to recover from them. If you're stuck, start here.

---

## Ralph Loop Fails After 10 Retries

The Ralph Loop (lint → typecheck → test → build) retries up to 10 times. If it still fails:

**Diagnose:** Read the exact error from the last retry. Common causes: flaky test (passes locally, fails in loop), missing dependency, environment variable not set, database not running.

**Recover:**
- Resume from the failing phase: `/implement-track {id} --phase N` (skips completed phases)
- Debug locally in the worktree: `cd .worktrees/ai-track-{id}` and run the failing command manually
- Use `--no-worktree` to implement on the current branch (easier to debug, but loses isolation): `/implement-track {id} --no-worktree`
- If the issue is a single flaky test, fix it in the worktree, commit, and re-run the Ralph Loop manually

---

## Worktree in Bad State

Symptoms: worktree exists but checkout is corrupted, or a previous run left an orphaned worktree.

**List worktrees:**
```bash
git worktree list
```

**Remove a broken worktree:**
```bash
git worktree remove .worktrees/ai-track-{id} --force
```

**Recover uncommitted work:** If the worktree has uncommitted changes you want to keep:
```bash
cd .worktrees/ai-track-{id}
git stash
cd ../..
git worktree remove .worktrees/ai-track-{id}
# Create a new branch and apply the stash
git checkout -b recovery/track-{id}
git stash pop
```

**Clean up all worktrees:**
```bash
git worktree prune
```

---

## `/accept` Created a Bad Spec

If the generated `spec.md` or `plan.md` doesn't match your intent:

**Option A — Edit directly:** The spec and plan are plain Markdown files in `conductor/tracks/{track-id}/`. Edit them manually, then run `/review-specs {track-id}` to validate.

**Option B — Regenerate:** Delete the track directory and re-run `/accept` with a more detailed GitHub Issue. The more specific your issue's acceptance criteria (Gherkin scenarios), the better the generated spec.

**Option C — Resolve clarifications:** If `/accept` flagged `[NEEDS CLARIFICATION]` markers, resolve them in `spec.md` before proceeding. These mark genuinely ambiguous requirements — don't just delete them.

---

## `/review-specs` Fails with CRITICAL

Severity levels and what they mean:

| Severity | Meaning | Action |
|----------|---------|--------|
| CRITICAL | Blocks implementation — principles violation, unresolved `[NEEDS CLARIFICATION]`, spec↔plan mismatch | Must fix before `/implement-track` will proceed |
| HIGH | Likely to cause implementation problems — missing test tasks, coverage gaps | Should fix, but won't block implementation |
| MEDIUM | Quality concern — vague requirements, weak acceptance criteria | Fix when convenient |
| LOW | Suggestion — could be more specific, minor inconsistency | Optional |

**Common CRITICAL causes:**
- `[NEEDS CLARIFICATION]` markers still in spec.md → resolve the ambiguity
- Spec lists a requirement but plan has no corresponding task → add the task to plan.md
- Architectural principle violation (MUST level) → redesign the approach or request an exception in the review
- Gherkin scenario has no matching test task → add a test task to the appropriate phase

**To re-run after fixes:** Edit spec.md/plan.md, then run `/review-specs {track-id}` again. The old `review.md` is overwritten.

---

## Plugins Not Loading

```bash
# Clear plugin cache and reinstall
rm -rf ~/.claude/plugins/cache/claude-code-workflows
rm ~/.claude/plugins/installed_plugins.json

# Re-register marketplace and reinstall
/plugin marketplace add wshobson/agents
/plugin install {plugin-names}
```

If plugins are installed but `/prime` shows them as missing, check that `enabledPlugins` in `.claude/settings.json` includes them. `/prime` will self-heal if `enabledPlugins` is missing entirely.

---

## Agent Teams Not Spawning

1. Check env var: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in `.claude/settings.json`
2. Check display mode: `teammateMode` in `~/.claude/settings.json` (user-level)
3. If using tmux mode, ensure tmux is installed: `which tmux`
4. Try `in-process` mode first to rule out terminal issues

---

## GitHub Sync Fails

If `/implement-track` fails to create a PR or update the GitHub Issue:

**PR creation failed:**
```bash
# Verify gh CLI is authenticated
gh auth status

# Create PR manually from the worktree branch
git push -u origin ai/track-{id}
gh pr create --base main --head ai/track-{id} --title "Track: {id}" --body "Implements track {id}"
```

**Issue status not updated:**
```bash
# Manually update the GitHub Issue
gh issue edit {NNN} --remove-label "accepted" --add-label "implementing"
```

---

## Conductor Artifacts Out of Sync

If `conductor/tracks.md` shows a track as "complete" but the code is broken, or shows "in progress" but the track was already merged:

**Reset track status:** Edit `conductor/tracks.md` directly. Track status markers: `[ ]` = not started, `[~]` = in progress, `[x]` = complete.

**Re-check track state:**
```bash
/conductor:status
```

Conductor reads `tracks.md` as the source of truth. If you fix the file, the status commands will reflect the update immediately.

---

## Context Compaction Lost Critical State

If auto-compaction happened mid-implementation and Claude lost track of where it was:

**If compaction already happened once (recommended):** Don't compact again — run `/handoff` to create a structured handoff document, then start a fresh session. The handoff captures track state, decisions, failed approaches, and exact error messages — everything the fresh session needs to resume without repeating work.

```bash
# In the degrading session
/handoff --track {track-id}

# In the fresh session
/prime
Read conductor/handoffs/{filename} and continue from where the previous session left off.
```

Check `conductor/handoffs/` for recent handoff files.

**If this is the first compaction (lighter recovery):** Re-read track state from disk:

```bash
/prime                                    # Reload all context
/conductor:status                        # See which tracks are in progress
```

Then read the track's `plan.md` — it contains phase/task completion status. Claude can pick up where it left off.

**Prevent this:** Compact at natural breakpoints only. See CLAUDE.md Compact Instructions for the rules. Run `/compact focus on track {id}` to explicitly tell Claude what to preserve. The `PreCompact` hook in `.claude/settings.json` fires before every compaction as a reminder to consider `/handoff` instead. Run `/context-audit` proactively to check whether the context window is getting tight before it triggers auto-compaction.

---

## Port Conflicts

```bash
# Find what's using a port
lsof -i :{port}

# Kill the process
kill -9 {PID}
```

---

## Missing Context Files

`/prime` reports exactly which files are missing. Create them following the descriptions in SETUP.md Step 1.2. The minimum set to start building is `context/tech-stack.md` + `CLAUDE.md` validation commands.

---

*Last updated: {DATE}*
