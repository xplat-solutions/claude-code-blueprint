# /accept

> Accept a GitHub Issue or PRD file into Conductor tracks. Routes by label: `story`/`bug` → one track, `epic` → decompose into multiple tracks + child GitHub Issues. Does NOT implement — use `/implement-track` or `/implement-prd` after reviewing the generated specs.

## Usage

```
/accept #123                    # Accept a GitHub Issue (story, bug, or epic)
/accept prd/my-feature.md       # Accept a local PRD file as an epic
```

---

## Prerequisites

1. **`/prime` ran this session** — Session UUID must be in conversation memory.
2. **`gh` CLI authenticated** — `gh auth status` must succeed.
3. **Conductor initialized** — `conductor/tracks.md` must exist.

---

## Arguments

```
/accept {input}

input:  One of:
  #123              — GitHub Issue number (prefixed with #)
  prd/{filename}.md — Path to a local PRD file (treated as epic)
```

---

## Process Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         /accept {input}                                  │
│                                                                          │
│   Step 1: RESOLVE INPUT                                                  │
│   ├── If #NNN: fetch issue via gh, extract labels and body              │
│   ├── If prd/*.md: read file, treat as epic                             │
│   └── Determine route: story | bug | epic                               │
│                                                                          │
│   Step 2: ROUTE                                                          │
│   ├── story/bug → Step 3A (single track)                                │
│   └── epic      → Step 3B (decompose to multiple tracks)                │
│                                                                          │
│   Step 3A: SINGLE TRACK (story or bug)                                   │
│   ├── Parse issue body: use case, Gherkin acceptance criteria           │
│   ├── Generate track ID: {slug}_{YYYYMMDD}                             │
│   ├── Create conductor/tracks/{track-id}/spec.md                        │
│   ├── Create conductor/tracks/{track-id}/plan.md                        │
│   ├── Register in conductor/tracks.md                                    │
│   ├── Label issue "accepted"                                             │
│   ├── Update GitHub Projects status → "Accepted"                        │
│   └── Comment on issue with track ID                                     │
│                                                                          │
│   Step 3B: DECOMPOSE EPIC                                                │
│   ├── Parse issue/file: user stories, requirements, dependencies        │
│   ├── For each user story:                                               │
│   │   ├── Generate track ID: {epic-slug}-{story-slug}_{YYYYMMDD}       │
│   │   ├── Create conductor/tracks/{track-id}/spec.md                    │
│   │   ├── Create conductor/tracks/{track-id}/plan.md                    │
│   │   ├── Register in conductor/tracks.md (with dependency notation)    │
│   │   ├── Create child GitHub Issue (labeled "story")                   │
│   │   └── Link child issue to parent epic                               │
│   ├── Save PRD to prd/{slug}.md (if from GitHub Issue)                  │
│   ├── Label epic issue "accepted"                                        │
│   ├── Update GitHub Projects status → "Accepted"                        │
│   └── Comment on epic with track list + dependency graph                │
│                                                                          │
│   Step 4: OUTPUT SUMMARY                                                 │
│   └── Display tracks created, ready for review                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Resolve Input

### GitHub Issue (`#NNN`)

```bash
gh issue view {number} --json title,body,labels,number,url
```

Extract the issue labels to determine the route:

| Label | Route |
|-------|-------|
| `story` | Single track (Step 3A) |
| `bug` | Single track (Step 3A) |
| `epic` | Decompose (Step 3B) |

If no type label is found:

```
❌ Issue #{number} has no type label (story, bug, or epic).

Add a label first:
  gh issue edit {number} --add-label "story"
  gh issue edit {number} --add-label "bug"
  gh issue edit {number} --add-label "epic"
```

**Stop here.**

### Local PRD File (`prd/*.md`)

Read the file. Treat it as an **epic** — always route to Step 3B (decompose).

If the file doesn't exist:

```
❌ File not found: {path}

Ensure the PRD file exists in the prd/ directory.
```

---

## Step 2: Route

Based on the resolved type, proceed to Step 3A or Step 3B.

---

## Step 3A: Single Track (Story or Bug)

### Parse Issue Body

Extract from the GitHub Issue body:

- **Summary** — from the Summary field
- **Use Case** — As a / I want / So that (for stories)
- **Steps to Reproduce** — (for bugs)
- **Acceptance Criteria** — Gherkin scenarios (required)
- **Technical Notes** — optional hints
- **Priority** — from the Priority dropdown

### Generate Track ID

```
{slug}_{YYYYMMDD}

slug: kebab-case from the issue summary (max 40 chars)
date: today's date

Example: password-reset_20260228
```

### Create spec.md

Write `conductor/tracks/{track-id}/spec.md`:

```markdown
# {Issue Title}

> Source: gh#{issue-number}

## Description

{Use case or bug description from the issue body}

## Scope

{story → "new feature" or "enhancement" | bug → "bug fix"}

## Acceptance Criteria

{Gherkin scenarios copied verbatim from the issue}

## Functional Requirements

{Derived from the acceptance criteria — one FR per scenario}

- **FR-1:** {requirement from Scenario 1}
- **FR-2:** {requirement from Scenario 2}

## Non-Functional Requirements

{From technical notes, or defaults if not specified}

- **NFR-1:** {requirement}

## Technical Notes

{From the Technical Notes field, if provided}

## Priority

{From the Priority dropdown}
```

### Create plan.md

Use Conductor's plan format. Derive phases from the acceptance criteria:

```markdown
# Implementation Plan: {Issue Title}

> Source: gh#{issue-number}

## Progress Summary

| Phase | Status | Description |
|-------|--------|-------------|
| 1 | Not Started | {Phase 1 description} |
| 2 | Not Started | {Phase 2 description} |
| 3 | Not Started | Tests + validation |

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

## Phase 3: Integration & Validation

- [ ] Task 3.1: Integration tests
- [ ] Task 3.2: Final verification against all Gherkin scenarios
- [ ] Task 3.3: Update documentation if needed

### Final Verification
- [ ] All tests pass (unit + integration)
- [ ] Lint + typecheck pass
- [ ] Build succeeds
```

### Register Track

Add to `conductor/tracks.md`:

```markdown
- [ ] {track-id} — {Issue Title} (gh#{issue-number})
```

### Update GitHub Issue

```bash
# Add "accepted" label
gh issue edit {number} --add-label "accepted"

# Comment with track info
gh issue comment {number} --body "🎯 Track created: \`{track-id}\`

Spec: \`conductor/tracks/{track-id}/spec.md\`
Plan: \`conductor/tracks/{track-id}/plan.md\`

Next: Review the spec and plan, then run \`/implement-track {track-id}\`"
```

If the issue is linked to a GitHub Project, update the project status to "Accepted":

```bash
# Get the project item ID and update status
gh project item-list {project-number} --owner {org} --format json | # find item for this issue
gh project item-edit --project-id {project-id} --id {item-id} --field-id {status-field-id} --single-select-option-id {accepted-option-id}
```

> **Note:** GitHub Projects v2 field updates via CLI can be complex. If automated status update fails, log a warning and continue — the "accepted" label still provides status visibility.

---

## Step 3B: Decompose Epic

### Parse Epic Content

Extract from the issue body or PRD file:

- **Feature Title** — the epic's name
- **Executive Summary** — what and why
- **User Stories** — list of stories with As a / I want / So that + Gherkin
- **Requirements** — FR-N and NFR-N items
- **Dependencies** — ordering constraints between stories
- **Technical Considerations** — architecture notes

### Generate Epic Slug

```
{slug} = kebab-case from the feature title (max 40 chars)
```

### For Each User Story

#### Generate Track ID

```
{epic-slug}-{story-slug}_{YYYYMMDD}

Example: multi-tenant-user-mgmt_20260228
         multi-tenant-rbac_20260228
         multi-tenant-dashboard_20260228
```

#### Create spec.md

Same format as Step 3A, but include:

- `Source: gh#{epic-issue-number}` (or `Source: prd/{filename}.md`)
- `Parent Epic: {epic-slug}`
- Dependencies section listing other tracks this depends on

```markdown
# {Story Title}

> Source: gh#{epic-issue-number}
> Parent Epic: {epic-slug}

## Description

{Story use case}

## Scope

new feature

## Acceptance Criteria

{Gherkin scenarios for this story}

## Functional Requirements

- **FR-1:** {derived from Gherkin}

## Non-Functional Requirements

- **NFR-1:** {from epic requirements relevant to this story}

## Dependencies

- {track-id of dependency} — {why}

## Technical Notes

{From epic technical considerations relevant to this story}

## Priority

{From epic priority}
```

#### Create plan.md

Same Conductor format as Step 3A.

#### Register in conductor/tracks.md

Add all tracks from this epic in a group, with dependency notation:

```markdown
## Epic: {Feature Title} (gh#{issue-number})

- [ ] {track-id-1} — {Story 1 Title}
- [ ] {track-id-2} — {Story 2 Title} (depends: {track-id-1})
- [ ] {track-id-3} — {Story 3 Title}
```

#### Create Child GitHub Issue

For each story, create a child GitHub Issue:

```bash
gh issue create \
  --title "{Story Title}" \
  --label "story" \
  --label "epic:{epic-slug}" \
  --body "## Source

Parent epic: #{epic-issue-number}
Track: \`{track-id}\`

## Use Case

{As a / I want / So that}

## Acceptance Criteria

{Gherkin scenarios}

## Dependencies

{dependency list}

---

*Auto-generated by /accept from epic #{epic-issue-number}*"
```

### Save PRD

If the source was a GitHub Issue (not a local file), save the epic content to `prd/`:

```bash
# Save the issue body as a PRD file
# File: prd/{epic-slug}.md
```

Write the PRD file with the full issue content, including a header:

```markdown
# {Feature Title}

> Source: gh#{epic-issue-number}
> Accepted: {YYYY-MM-DD}
> Tracks: {track-id-1}, {track-id-2}, {track-id-3}

{Full epic issue body}
```

### Update GitHub Epic Issue

```bash
# Add "accepted" label
gh issue edit {epic-issue-number} --add-label "accepted"

# Comment with decomposition summary
gh issue comment {epic-issue-number} --body "🎯 Epic decomposed into {N} tracks:

| Track | Story | Dependencies | Child Issue |
|-------|-------|-------------|-------------|
| \`{track-id-1}\` | {Story 1} | — | #{child-1} |
| \`{track-id-2}\` | {Story 2} | {track-id-1} | #{child-2} |
| \`{track-id-3}\` | {Story 3} | — | #{child-3} |

PRD saved: \`prd/{epic-slug}.md\`

Next: Review specs in \`conductor/tracks/\`, then run \`/implement-prd {epic-slug}\`"
```

---

## Step 4: Output Summary

### Single Track (Story/Bug)

```
✅ Track accepted: {track-id}

Source: gh#{issue-number} — {issue-title}
Type: {story | bug}
Priority: {priority}

Track files:
  conductor/tracks/{track-id}/spec.md
  conductor/tracks/{track-id}/plan.md

GitHub:
  Issue labeled: accepted
  Comment posted with track info

Next: Review the spec and plan, then:
  /implement-track {track-id}
```

### Epic Decomposition

```
✅ Epic decomposed: {epic-slug}

Source: gh#{issue-number} — {feature-title}
Tracks created: {N}

| Track | Story | Dependencies | Child Issue |
|-------|-------|-------------|-------------|
| {track-id-1} | {Story 1} | — | #{child-1} |
| {track-id-2} | {Story 2} | {track-id-1} | #{child-2} |
| {track-id-3} | {Story 3} | — | #{child-3} |

PRD saved: prd/{epic-slug}.md

GitHub:
  Epic labeled: accepted
  {N} child issues created
  Decomposition comment posted

Next: Review specs in conductor/tracks/, then:
  /implement-prd {epic-slug}
  Or implement individually: /implement-track {track-id}
```

---

## Error Handling

### Prime Not Detected

```
❌ Prime session not detected
Run /prime first to load project context and verify plugins.
```

### Issue Not Found

```
❌ Issue #{number} not found
Check: gh issue view {number}
```

### No Type Label

```
❌ Issue #{number} has no type label (story, bug, or epic).
Add a label: gh issue edit {number} --add-label "story"
```

### Conductor Not Initialized

```
❌ conductor/tracks.md not found
Run /conductor:setup first.
```

### PRD File Not Found

```
❌ File not found: {path}
Ensure the PRD file exists in the prd/ directory.
```

---

## Related Commands

- `/setup-project` — One-time GitHub project setup (labels, templates, board)
- `/implement-track {id}` — Implement a single track
- `/implement-prd {slug}` — Implement all tracks from an epic/PRD
- `/conductor:new-track` — Create track interactively (alternative to /accept)
- `/conductor:status` — View all track progress
- `/prime` — Load project context (prerequisite)
