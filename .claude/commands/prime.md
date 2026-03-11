# /prime

> Load the {PROJECT_NAME} project context layer and verify Hobson plugin setup. Run this first in every Claude Code session.

## Usage

```
/prime
```

---

## Process

### Step 1: Read Root Context Files

Read these files in order. **If any is missing, stop and report the error.**

```
1. CLAUDE.md                        — Session config, project overview, tech stack
2. AGENTS.md                        — Multi-agent coordination rules
```

### Step 2: Read All Context Files

Read every `.md` file in `context/` and all subdirectories (`context/**/*.md`). This includes:

```
context/
├── product-vision.md               — What {PROJECT_NAME} is, target users, success criteria
├── tech-stack.md                    — Technology decisions and rationale
├── conventions.md                   — Code style, naming, commit format, API patterns, validation gates
├── principles.md                    — Architectural principles & governance gates (used by /review-specs)
├── decisions/                       — Architecture Decision Records (if any)
└── guides/
    └── plugin-registry.md           — Technology → plugin mapping (verified v1.5.5)
```

**If `context/product-vision.md`, `context/tech-stack.md`, or `context/conventions.md` is missing, stop and report the error.** Other files in `context/` are optional — read them if present, skip if not.

**If `context/guides/plugin-registry.md` is missing, stop and report the error.** The registry is required for plugin detection.

Any additional `.md` files found in `context/` or its subdirectories (e.g., project-specific guides added by the team) are read automatically. No hardcoded file list — `/prime` reads everything recursively.

### Step 2b: Read PRD Files (Non-Blocking)

Read any `.md` files in the `prd/` directory (`prd/*.md`). These are committed PRD files saved by `/accept` from epic issues.

```
prd/
└── {epic-slug}.md                   — PRD for an accepted epic (source, tracks, requirements)
```

This is non-blocking — if `prd/` doesn't exist or is empty, continue without error. PRD files provide context about active epics and their track decomposition.

### Step 3: Check Conductor Artifacts

Check for Conductor artifacts (non-blocking — continue if missing):

```
conductor/product.md                — Conductor product definition
conductor/tech-stack.md             — Conductor tech stack preferences
conductor/workflow.md               — Conductor workflow rules (TDD, commits)
conductor/tracks.md                 — Conductor track registry
```

If missing:

```
💡 Conductor not initialized. Run /conductor:setup to create project artifacts.
```

If `conductor/tracks.md` exists, read it to get track inventory and status counts.

### Step 4: Verify Plugin Setup

#### 4a. Check Marketplace

Check if the `wshobson/agents` marketplace is registered (run `/plugin` to see available marketplaces).

If not registered:

```
❌ Plugin marketplace not registered

The wshobson/agents marketplace provides Conductor, Agent Teams, and domain expertise skills.
Run: /plugin marketplace add wshobson/agents
Then re-run: /prime
```

**Stop here.** Without the marketplace, plugins cannot be installed or verified.

#### 4b. Detect Recommended Plugins

1. Read `context/tech-stack.md` — extract all technology names
2. Read `context/guides/plugin-registry.md` — load the technology → plugin mapping
3. Match technologies against the registry to build the **recommended list**
4. Read `enabledPlugins` from `.claude/settings.json` (project scope) — this is the **installed list**
5. If not found in project scope, also check `~/.claude/settings.json` (user scope)
6. Cross-reference recommended list against installed list

**If `enabledPlugins` is not found in ANY settings.json — self-heal:**

1. Build an `enabledPlugins` object with every recommended plugin set to `true`
2. Write the `enabledPlugins` key into `.claude/settings.json` (project scope)
3. Log: `🔧 enabledPlugins was missing from settings — auto-populated from tech stack.`

**Plugin states:**

| `enabledPlugins` value | Meaning |
|------------------------|---------|
| `true` | Installed and enabled |
| `false` | Installed but disabled |
| Key absent but in recommended list | Not installed (needs installation) |

**Output:**

```
🔌 Plugin Status
┌──────────────────────────┬──────────────┬───────────────────────────────────────────────┐
│ Plugin                   │ Status       │ Activation Context                            │
├──────────────────────────┼──────────────┼───────────────────────────────────────────────┤
│ conductor                │ ✅ Enabled   │ Always active (workflow automation)            │
│ agent-teams              │ ✅ Enabled   │ /team-spawn, /team-feature, /team-review       │
│ developer-essentials     │ ✅ Enabled   │ Always active (git, debugging, E2E, monorepo) │
│ {plugin-name}            │ ✅ Enabled   │ {file patterns or directories}                │
│ {plugin-name}            │ ⚠️ Missing   │ {file patterns or directories}                │
│ {plugin-name}            │ ⛔ Disabled  │ {file patterns or directories}                │
└──────────────────────────┴──────────────┴───────────────────────────────────────────────┘

  {N} enabled · {N} disabled · {N} missing
```

If there are missing plugins:

```
Install missing plugins:
  /plugin install {plugin-names}
```

If all recommended plugins are enabled:

```
🔌 All {count} recommended plugins enabled ✅
  Skills auto-load when you work in matching files/directories.
```

#### 4c. Verify Agent Teams Configuration

Check Agent Teams setup (non-blocking):

1. Check `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` env var in `.claude/settings.json` — should be `"1"`
2. If missing:

```
⚠️ Agent Teams not enabled
  Add to .claude/settings.json env: "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
```

3. Optionally check `~/.claude/settings.json` for `teammateMode`. If not set:

```
💡 Agent Teams display mode not configured (using default: in-process)
  For better UX, set in ~/.claude/settings.json: "teammateMode": "tmux"
  Options: tmux (recommended), iterm2 (macOS), in-process (default)
```

### Step 5: Generate Session UUID & Output Summary

Generate a random UUID for this prime session. Display it in the output so it persists in conversation memory. This UUID is checked by `/implement-track` to verify prime ran in the current session.

```
✅ Context loaded: {PROJECT_NAME}
🔑 Session: {uuid}

Project: {one-line description from product-vision.md}
Stack: {tech stack summary from tech-stack.md}
Context files: {N} loaded from context/**/*.md
PRDs: {N} loaded from prd/*.md
Conductor: {Initialized — {N} tracks / Not initialized}
Agent Teams: {Enabled / Not enabled}
Plugins: {N} enabled, {N} missing

Ready. Use /implement-track {id}, /implement-prompt {desc}, or /team-feature {desc} to start building.
```

**Do NOT start implementing anything.** `/prime` only loads context. Wait for the next command.

---

## Error Handling

If any required file is missing, stop immediately and report:

```
❌ Context layer incomplete

Missing:
- {file path}
- {file path}

Please create the missing file(s) before retrying /prime.
See SETUP.md for the full setup guide.
```

Required files (stop if missing):

- `CLAUDE.md`
- `AGENTS.md`
- `context/product-vision.md`
- `context/tech-stack.md`
- `context/conventions.md`
- `context/guides/plugin-registry.md`

Optional files (warn if missing, continue):

- All other `context/**/*.md` files
- `conductor/*` artifacts

---

## Notes

- `/prime` is idempotent — running it twice is fine (generates a new UUID each time)
- Context is session-scoped — each new Claude Code session needs `/prime` again
- **Subagent context:** Subagents spawned via the Task tool do NOT inherit the main session's context. Task descriptions must include relevant context references so the subagent reads project files directly.
- **Agent Teams:** Each teammate is an independent Claude Code session. Teammates auto-load installed plugins and can read project files (including all `context/` files) independently. The team lead coordinates via structured messages.
- **Skill loading:** For the main thread, plugin skills auto-activate based on file context. No manual skill configuration is needed. Subagents also auto-activate skills when they read files in matching directories.
