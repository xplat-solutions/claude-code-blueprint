# /review-specs

> Non-destructive cross-artifact consistency and quality analysis of a Conductor track's spec.md and plan.md before implementation. Produces a structured analysis report and writes a `review.md` file to the track directory as proof of review. This file is required by `/implement-track` as a preflight gate.

## Usage

```
/review-specs {track-id}
/review-specs {track-id} --epic {epic-slug}     # Batch review all tracks in an epic
```

---

## Prerequisites

1. **`/prime` ran this session** — Session UUID must be in conversation memory.
2. **Track exists** — `conductor/tracks/{track-id}/spec.md` and `conductor/tracks/{track-id}/plan.md` must exist.

---

## Arguments

```
/review-specs {track-id} [options]

track-id:  The Conductor track ID (e.g., user-auth_20250115)
           Format: {shortname}_{YYYYMMDD}
```

## Options

| Flag | Effect |
|------|--------|
| `--epic {slug}` | Review all tracks belonging to this epic group in `conductor/tracks.md`. Runs analysis per-track, then produces a cross-track consistency summary. |

---

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify `spec.md` or `plan.md`. The only file written is `conductor/tracks/{track-id}/review.md` — the analysis report.

**Principles Authority**: The project principles file (`context/principles.md`), if it exists, is treated as authoritative. Principle conflicts are automatically CRITICAL severity. If a principle needs to change, that happens in a separate update outside this command.

---

## Process Overview

```
┌────────────────────────────────────────────────────────────────────┐
│              /review-specs {track-id}                               │
│                                                                    │
│   Step 1: INITIALIZE                                               │
│   ├── Verify prime session                                         │
│   ├── Verify track directory and files exist                       │
│   ├── Load spec.md, plan.md                                        │
│   ├── Load context/principles.md (if exists)                       │
│   ├── Load context/conventions.md (for convention alignment)       │
│   └── Load context/product-vision.md (for scope alignment)         │
│                                                                    │
│   Step 2: BUILD SEMANTIC MODELS                                    │
│   ├── Extract requirements inventory from spec.md                  │
│   ├── Extract task/phase inventory from plan.md                    │
│   ├── Map tasks → requirements                                     │
│   └── Extract principle rules (if principles.md exists)            │
│                                                                    │
│   Step 3: DETECTION PASSES                                         │
│   ├── A. Duplication Detection                                     │
│   ├── B. Ambiguity Detection                                       │
│   ├── C. Underspecification                                        │
│   ├── D. Principles Alignment                                      │
│   ├── E. Coverage Gaps                                             │
│   └── F. Inconsistency                                             │
│                                                                    │
│   Step 4: SEVERITY ASSIGNMENT                                      │
│   Step 5: PRODUCE ANALYSIS REPORT                                  │
│   Step 6: WRITE review.md                                          │
│   Step 7: NEXT ACTIONS                                             │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Initialize Analysis Context

### Verify Prime Session

Check that a prime session UUID exists in conversation memory. If not:

```
❌ Prime session not detected
Run /prime first to load project context and verify plugins.
```

### Load Artifacts

Load the following files:

**Required (abort if missing):**

- `conductor/tracks/{track-id}/spec.md` — requirements, acceptance criteria, FRs, NFRs
- `conductor/tracks/{track-id}/plan.md` — phases, tasks, verification checkpoints

**Optional (enhance analysis if present):**

- `context/principles.md` — architectural principles and gates
- `context/conventions.md` — code conventions, testing standards, track completion gates
- `context/product-vision.md` — product scope and success criteria

If required files are missing:

```
❌ Track artifacts incomplete: conductor/tracks/{track-id}/

Missing: {list of missing files}

Create the track first:
  /accept #NNN                            (from GitHub Issue)
  /conductor:new-track                    (interactive)
```

---

## Step 2: Build Semantic Models

Create internal representations (do not include raw artifacts in output):

### Requirements Inventory

From `spec.md`, extract:

- Each **FR-N** (functional requirement) with a stable slug key (e.g., "User can upload file" → `user-can-upload-file`)
- Each **NFR-N** (non-functional requirement) with a slug key
- Each **acceptance criterion** / Gherkin scenario mapped to its parent FR
- **Dependencies** declared on other tracks

### Task/Phase Inventory

From `plan.md`, extract:

- Phase names and descriptions
- Task descriptions with their checkbox status
- File paths referenced in tasks
- Verification checkpoints per phase

### Coverage Mapping

Map each task to one or more requirements using:

- Explicit references (FR-N, NFR-N identifiers)
- Keyword/phrase matching between task descriptions and requirement text
- Gherkin scenario ↔ test task alignment

### Principles Rule Set (if principles.md exists)

Extract principle names and their MUST/SHOULD normative statements for validation in Step 3D.

---

## Step 3: Detection Passes

Focus on high-signal findings. Limit to **50 findings total**; aggregate remainder in overflow summary.

### A. Duplication Detection

- Identify near-duplicate requirements in `spec.md` (similar phrasing, overlapping scope)
- Identify tasks in `plan.md` that duplicate work (same files, same operations)
- Mark lower-quality phrasing for consolidation

### B. Ambiguity Detection

- Flag vague adjectives lacking measurable criteria: "fast", "scalable", "secure", "intuitive", "robust", "flexible", "user-friendly", "efficient", "reliable"
- Flag unresolved placeholders: `TODO`, `TBD`, `TKTK`, `???`, `<placeholder>`, `{placeholder}`
- Flag **`[NEEDS CLARIFICATION]`** markers that were not resolved — these are CRITICAL (the `/accept` command marks uncertainties this way; they must be resolved before implementation)
- Flag acceptance criteria that are not testable (no concrete assertion possible)

### C. Underspecification

- Requirements with verbs but missing object or measurable outcome
- Acceptance criteria / Gherkin scenarios missing `Then` assertions
- Tasks referencing files or components not mentioned in `spec.md`
- FRs with no corresponding Gherkin scenario
- NFRs with no measurable threshold (e.g., "should be fast" with no latency target)

### D. Principles Alignment (if context/principles.md exists)

- Any requirement or plan element conflicting with a MUST principle
- Missing mandated quality gates from principles
- Complexity or abstraction violations (if principles define gates for these)

If `context/principles.md` does not exist, skip this pass and note:

```
ℹ️ Principles alignment skipped — no context/principles.md found.
   Consider creating one for architectural governance.
```

### E. Coverage Gaps

- **Uncovered requirements:** FRs or NFRs with zero associated tasks in `plan.md`
- **Orphan tasks:** Tasks with no mapped requirement or acceptance criterion
- **NFR coverage:** Non-functional requirements (performance, security, observability) not reflected in any task
- **Acceptance criteria coverage:** Gherkin scenarios with no corresponding test task

### F. Inconsistency

- **Terminology drift:** Same concept named differently across spec.md and plan.md (e.g., "user" vs "account" vs "member")
- **Entity mismatch:** Data entities referenced in plan.md but absent in spec.md (or vice versa)
- **Task ordering contradictions:** Integration tasks before foundational setup tasks without dependency notes
- **Conflicting requirements:** Contradictory statements (e.g., one FR says REST, another says GraphQL)
- **Scope creep:** Tasks in plan.md that address requirements not present in spec.md

---

## Step 4: Severity Assignment

| Severity | Criteria |
|----------|----------|
| **CRITICAL** | Violates a MUST principle; unresolved `[NEEDS CLARIFICATION]` marker; FR with zero task coverage that blocks baseline functionality; contradictory requirements |
| **HIGH** | Duplicate or conflicting requirement; ambiguous security/performance NFR; untestable acceptance criterion; orphan task with no requirement backing |
| **MEDIUM** | Terminology drift; missing NFR task coverage; underspecified edge case; vague adjective without metric |
| **LOW** | Style/wording improvements; minor redundancy not affecting execution order; optional improvement suggestions |

---

## Step 5: Produce Analysis Report

Generate a Markdown report with this structure:

```markdown
# Specification Review: {track-id}

> Reviewed: {YYYY-MM-DD HH:MM}
> Artifacts: spec.md, plan.md
> Principles: {loaded | not found}

## Findings

| ID | Category | Severity | Location | Summary | Recommendation |
|----|----------|----------|----------|---------|----------------|
| D1 | Duplication | HIGH | spec.md: FR-2, FR-5 | Near-duplicate requirements ... | Merge; keep FR-2 phrasing |
| A1 | Ambiguity | CRITICAL | spec.md: FR-3 | [NEEDS CLARIFICATION] unresolved | Resolve before implementation |
| C1 | Coverage | HIGH | spec.md: NFR-2 | No task covers performance target | Add performance test task to Phase 3 |

## Coverage Summary

| Requirement | Has Task? | Task IDs | Notes |
|-------------|-----------|----------|-------|
| FR-1: {slug} | ✅ | 1.1, 1.2 | |
| FR-2: {slug} | ❌ | — | No coverage |
| NFR-1: {slug} | ✅ | 3.1 | |

## Metrics

| Metric | Value |
|--------|-------|
| Total FRs | {N} |
| Total NFRs | {N} |
| Total Tasks | {N} |
| FR Coverage | {N}/{M} ({%}) |
| NFR Coverage | {N}/{M} ({%}) |
| Acceptance Criteria Coverage | {N}/{M} ({%}) |
| Critical Issues | {N} |
| High Issues | {N} |
| Medium Issues | {N} |
| Low Issues | {N} |
| Unresolved [NEEDS CLARIFICATION] | {N} |

## Verdict

{PASS | PASS WITH WARNINGS | FAIL}

- **PASS**: Zero CRITICAL issues, FR coverage ≥ 80%, no unresolved [NEEDS CLARIFICATION]
- **PASS WITH WARNINGS**: Zero CRITICAL issues but HIGH issues exist or coverage < 80%
- **FAIL**: Any CRITICAL issue present or unresolved [NEEDS CLARIFICATION] markers
```

---

## Step 6: Write review.md

Write the analysis report to:

```
conductor/tracks/{track-id}/review.md
```

This file serves as proof that specs were reviewed before implementation. The `/implement-track` command checks for this file as a preflight gate.

**Important:** The `review.md` file must contain:
- The `Verdict:` line (PASS, PASS WITH WARNINGS, or FAIL)
- The `Reviewed:` timestamp
- The full findings table and metrics

---

## Step 7: Next Actions

### If Verdict is FAIL

```
❌ Review FAILED — {N} CRITICAL issue(s) found

CRITICAL issues must be resolved before implementation:
{list critical issues with recommendations}

After resolving:
  1. Update spec.md and/or plan.md with fixes
  2. Re-run: /review-specs {track-id}

Do NOT run /implement-track until review passes.
```

### If Verdict is PASS WITH WARNINGS

```
⚠️ Review passed with warnings — {N} HIGH issue(s)

Recommended fixes (not blocking):
{list high issues with recommendations}

You may proceed:
  /implement-track {track-id}

Or fix issues first and re-run: /review-specs {track-id}
```

### If Verdict is PASS

```
✅ Review passed — track {track-id} is ready for implementation

Coverage: FR {%} | NFR {%} | Acceptance Criteria {%}
Issues: {N} low/medium (non-blocking)

Next:
  /implement-track {track-id}
```

---

## Epic Mode (`--epic {slug}`)

When `--epic` is specified:

1. Read `conductor/tracks.md` and find all tracks in the epic group for `{slug}`
2. Run the full analysis (Steps 1-6) for each track individually
3. After all individual reviews, produce a **Cross-Track Consistency Report**:

### Cross-Track Checks

- **Shared entity consistency:** Entities referenced across multiple tracks use the same names and shapes
- **Dependency alignment:** Tracks declaring dependencies reference tracks that actually exist and produce the expected outputs
- **NFR consistency:** Non-functional requirements (auth, performance, security) are addressed consistently across tracks
- **Terminology alignment:** Same concepts use the same terms across all track specs
- **Interface contracts:** Where one track produces an API/data structure another consumes, the contracts are compatible

### Cross-Track Report

```markdown
# Epic Review: {epic-slug}

> Tracks reviewed: {N}
> Overall verdict: {PASS | PASS WITH WARNINGS | FAIL}

## Per-Track Verdicts

| Track | Verdict | Critical | High | Medium | Low |
|-------|---------|----------|------|--------|-----|

## Cross-Track Findings

| ID | Tracks Involved | Summary | Recommendation |
|----|-----------------|---------|----------------|
```

Write individual `review.md` files per track AND a summary `review-epic.md` in `conductor/tracks/`.

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
Create a track first via /accept, /conductor:new-track, or manually.
```

### Artifacts Incomplete

```
❌ Track artifacts incomplete: conductor/tracks/{track-id}/
Missing: {spec.md | plan.md}

Both spec.md and plan.md are required for review.
```

---

## Related Commands

- `/prime` — Load context (prerequisite)
- `/accept {input}` — Accept issues/PRDs into tracks (creates spec.md + plan.md)
- `/implement-track {id}` — Implement a track (requires review.md from this command)
- `/implement-prd {slug}` — Batch implement epic (calls /implement-track per track)
- `/conductor:new-track` — Create track interactively
- `/conductor:status` — View all track progress
