# Context Budget Audit

Analyze token consumption across all context sources and report estimated budget usage. Helps identify bloat and optimize session headroom.

## Usage

```
/context-audit
/context-audit --track {track-id}
```

## Execution Steps

### Step 1: Scan Always-Loaded Context

Read and measure these files (loaded every session by `/prime`):

- `CLAUDE.md`
- `AGENTS.md`
- All files in `context/` (non-recursive): `context/*.md`
- All files in `context/decisions/`: `context/decisions/*.md`

For each file, report: filename, line count, estimated token count (lines × ~20 tokens/line as rough heuristic).

### Step 2: Scan Guides (Loaded vs Scouted)

List all files in `context/guides/`. For each:

- If it was loaded in full during `/prime` → mark as **loaded**
- If it was only scouted (previewed) → mark as **scouted** and note the token savings
- Report line count and estimated tokens for each

### Step 3: Scan Plugin Registry

Read `.claude/settings.json`. For each plugin in `enabledPlugins`:

- List the plugin name
- Estimate token weight: light (~0.5k), medium (~1.5k), heavy (~4k+) based on plugin type:
  - Single-skill plugins → light
  - Multi-command plugins (conductor, agent-teams, developer-essentials) → heavy
  - Domain plugins (javascript-typescript, backend-development) → medium
- Flag any plugins not referenced by `context/tech-stack.md` as potentially unnecessary

### Step 4: Scan MCP Servers

Read `.claude/settings.json` for `mcpServers`. For each configured server:

- List server name and estimated tool-definition token cost
- Flag servers with many tool definitions as high-cost

### Step 5: Scan Track Context (if --track flag provided)

If `--track {track-id}` was passed:

- Read `conductor/tracks/{track-id}/spec.md` — report line count and estimated tokens
- Read `conductor/tracks/{track-id}/plan.md` — report line count and estimated tokens
- Check if spec.md references other context files and note them
- This is **estimation only** — do not load the track into the active context

### Step 6: Generate Report

Output the report in this format:

```
## Context Budget Report

### Always-Loaded Context
  {filename}                         ~{lines} lines  (~{tokens} tokens)
  ...
  ─────────────────────────────────────────────
  Subtotal                                       ~{N}k tokens

### Guides ({N} loaded, {N} scouted)
  {filename}  [loaded]               ~{lines} lines  (~{tokens} tokens)
  {filename}  [scouted — not loaded] ~{lines} lines  (saving ~{tokens} tokens)
  ...
  ─────────────────────────────────────────────
  Loaded subtotal                                ~{N}k tokens
  Scouted savings                                ~{N}k tokens

### Plugins ({N} enabled)
  {plugin-name}                      ~{tokens} tokens ({weight class})
  ...
  ─────────────────────────────────────────────
  Subtotal                                       ~{N}k tokens

### MCP Servers ({N} configured)
  {server-name}                      ~{tokens} tokens (tool definitions)
  ...
  ─────────────────────────────────────────────
  Subtotal                                       ~{N}k tokens

### Track: {track-id} (estimated, if --track flag used)
  spec.md                            ~{lines} lines  (~{tokens} tokens)
  plan.md                            ~{lines} lines  (~{tokens} tokens)
  ─────────────────────────────────────────────
  Subtotal                                       ~{N}k tokens

### Summary
  Total estimated context load:      ~{N}k tokens
  Approximate % of 200k window:      ~{N}%
  Headroom for work:                 ~{N}k tokens

### Recommendations
  - {Actionable recommendations based on findings}
  - {e.g., "⚠ {plugin} is your heaviest plugin — consider context-mode compression"}
  - {e.g., "✓ Guide scouting is saving ~{N}k tokens ({N} guides scouted)"}
  - {e.g., "⚠ {mcp-server} has {N} tool definitions — consider disabling if unused"}
  - {e.g., "✓ Context budget is healthy — {N}% headroom available"}
```

## Notes

- Token estimates use a rough heuristic (~20 tokens per line, ~4 chars per token). Actual consumption varies by content density. The goal is directional awareness, not precision.
- Plugin token weights are estimates based on typical plugin sizes. Actual weights depend on how many skills, commands, and rules each plugin injects.
- Run this command after `/prime` for the most accurate picture of what's actually loaded.
- The `--track` flag provides estimation without loading — useful for gauging how much runway you'll have before starting implementation.
