# /setup-project

> One-time GitHub project setup: creates a Project board, issue templates, labels, and the `prd/` directory. Run once per repository.

## Usage

```
/setup-project
```

---

## Prerequisites

1. **`gh` CLI authenticated** — `gh auth status` must succeed.
2. **Repository has a GitHub remote** — `gh repo view` must resolve.

---

## Process

### Step 1: Verify GitHub Access

```bash
gh auth status
gh repo view --json name,owner
```

If either fails:

```
❌ GitHub CLI not authenticated or no remote found.

Fix:
  gh auth login
  git remote add origin https://github.com/{org}/{repo}.git
```

**Stop here.**

### Step 2: Create GitHub Project Board

Create a GitHub Projects (v2) board linked to the repository:

```bash
# Create the project
gh project create --owner {org} --title "{PROJECT_NAME}" --format board

# Note the project number from the output
```

#### Add Status Field Options

The Project board should have a **Status** field with these options (in order):

1. **Backlog** — Issue created, not yet accepted
2. **Accepted** — Track created via `/accept`, awaiting implementation
3. **Implementing** — `/implement-track` in progress
4. **In Review** — PR created, awaiting human review
5. **Done** — PR merged

```bash
# GitHub Projects v2 creates a default "Status" field.
# Add custom options via the project settings UI or gh CLI:
gh project field-list {project-number} --owner {org}
```

> **Note:** GitHub Projects v2 field customization via CLI is limited. If custom status options can't be set via `gh`, instruct the user to configure them manually in the Project board settings UI.

#### Add Custom Fields (Optional)

| Field | Type | Options |
|-------|------|---------|
| Priority | Single select | High, Medium, Low |
| Type | Single select | Story, Bug, Epic |
| Track ID | Text | — |

### Step 3: Create Labels

Create labels used by issue templates and `/accept` for routing:

```bash
# Type labels (used by /accept for routing)
gh label create "story" --description "User story — maps to one Conductor track" --color "0E8A16"
gh label create "bug" --description "Bug report — maps to one Conductor track" --color "D73A4A"
gh label create "epic" --description "Epic/PRD — decomposes into multiple tracks" --color "5319E7"

# Priority labels
gh label create "priority:high" --description "High priority" --color "B60205"
gh label create "priority:medium" --description "Medium priority" --color "FBCA04"
gh label create "priority:low" --description "Low priority" --color "0075CA"

# Status labels (optional — supplements GitHub Projects status)
gh label create "accepted" --description "Track created, awaiting implementation" --color "C2E0C6"
gh label create "implementing" --description "Implementation in progress" --color "BFD4F2"
```

If a label already exists, `gh label create` will error — that's fine, skip it.

### Step 4: Create Issue Templates

Write the following files to `.github/ISSUE_TEMPLATE/`:

#### `.github/ISSUE_TEMPLATE/story.yml`

```yaml
name: "User Story"
description: "A user story that maps to one Conductor track"
labels: ["story"]
body:
  - type: input
    id: summary
    attributes:
      label: Summary
      description: One-line summary of this story
      placeholder: "e.g., Users can reset their password via email"
    validations:
      required: true

  - type: textarea
    id: use-case
    attributes:
      label: Use Case
      description: Who needs this and why?
      value: |
        **As a** {role}
        **I want** {capability}
        **So that** {benefit}
    validations:
      required: true

  - type: textarea
    id: acceptance-criteria
    attributes:
      label: Acceptance Criteria (Gherkin)
      description: |
        Define acceptance criteria using Gherkin syntax.
        Each scenario should have one When and one Then.
        These flow verbatim into the Conductor track spec.
      value: |
        ```gherkin
        Feature: {Feature name}

          Scenario: {Happy path}
            Given {precondition}
            When {action}
            Then {expected result}

          Scenario: {Edge case}
            Given {precondition}
            When {action}
            Then {expected result}
        ```
    validations:
      required: true

  - type: textarea
    id: technical-notes
    attributes:
      label: Technical Notes
      description: Optional implementation hints, constraints, or dependencies.
      placeholder: "e.g., Must integrate with existing auth middleware. See ADR-003."
    validations:
      required: false

  - type: dropdown
    id: priority
    attributes:
      label: Priority
      options:
        - High
        - Medium
        - Low
    validations:
      required: true
```

#### `.github/ISSUE_TEMPLATE/bug.yml`

```yaml
name: "Bug Report"
description: "A bug that maps to one Conductor track"
labels: ["bug"]
body:
  - type: input
    id: summary
    attributes:
      label: Summary
      description: One-line description of the bug
      placeholder: "e.g., Login form shows 500 error after password reset"
    validations:
      required: true

  - type: textarea
    id: steps-to-reproduce
    attributes:
      label: Steps to Reproduce
      description: Exact steps to trigger the bug
      value: |
        1.
        2.
        3.
    validations:
      required: true

  - type: textarea
    id: expected-behavior
    attributes:
      label: Expected Behavior
      description: What should happen?
    validations:
      required: true

  - type: textarea
    id: actual-behavior
    attributes:
      label: Actual Behavior
      description: What actually happens?
    validations:
      required: true

  - type: textarea
    id: acceptance-criteria
    attributes:
      label: Acceptance Criteria (Gherkin)
      description: Define the fix verification using Gherkin syntax.
      value: |
        ```gherkin
        Feature: {Bug fix verification}

          Scenario: {Bug no longer occurs}
            Given {precondition}
            When {action that previously triggered the bug}
            Then {correct behavior}
        ```
    validations:
      required: true

  - type: textarea
    id: technical-notes
    attributes:
      label: Technical Notes
      description: Optional — logs, screenshots, stack traces, suspected root cause.
    validations:
      required: false

  - type: dropdown
    id: priority
    attributes:
      label: Priority
      options:
        - High
        - Medium
        - Low
    validations:
      required: true
```

#### `.github/ISSUE_TEMPLATE/epic.yml`

```yaml
name: "Epic / PRD"
description: "A product requirement that decomposes into multiple tracks"
labels: ["epic"]
body:
  - type: input
    id: feature-title
    attributes:
      label: Feature Title
      description: Short name for this feature/epic
      placeholder: "e.g., Multi-tenant dashboard"
    validations:
      required: true

  - type: textarea
    id: executive-summary
    attributes:
      label: Executive Summary
      description: What is this feature and why does it matter?
    validations:
      required: true

  - type: textarea
    id: user-stories
    attributes:
      label: User Stories
      description: |
        List the user stories that make up this epic.
        Each story should follow the As a / I want / So that format.
        Include Gherkin acceptance criteria per story.
      value: |
        ### Story 1: {title}

        **As a** {role}
        **I want** {capability}
        **So that** {benefit}

        ```gherkin
        Feature: {Feature name}

          Scenario: {Happy path}
            Given {precondition}
            When {action}
            Then {expected result}
        ```

        ### Story 2: {title}

        **As a** {role}
        **I want** {capability}
        **So that** {benefit}

        ```gherkin
        Feature: {Feature name}

          Scenario: {Happy path}
            Given {precondition}
            When {action}
            Then {expected result}
        ```
    validations:
      required: true

  - type: textarea
    id: requirements
    attributes:
      label: Requirements
      description: |
        Functional and non-functional requirements.
        Use FR-N for functional, NFR-N for non-functional.
      value: |
        **Functional Requirements**

        - **FR-1:** {requirement}
        - **FR-2:** {requirement}

        **Non-Functional Requirements**

        - **NFR-1:** {requirement}
    validations:
      required: true

  - type: textarea
    id: dependencies
    attributes:
      label: Dependencies & Ordering
      description: |
        List dependencies between stories. This determines the implementation order
        and worktree chaining strategy used by /implement-prd.
      placeholder: |
        Story 2 depends on Story 1 (needs the auth API)
        Story 3 is independent
    validations:
      required: false

  - type: textarea
    id: technical-considerations
    attributes:
      label: Technical Considerations
      description: Architecture notes, constraints, or risks.
    validations:
      required: false

  - type: dropdown
    id: priority
    attributes:
      label: Priority
      options:
        - High
        - Medium
        - Low
    validations:
      required: true
```

#### `.github/ISSUE_TEMPLATE/config.yml`

```yaml
blank_issues_enabled: false
contact_links: []
```

### Step 5: Create prd/ Directory

```bash
mkdir -p prd
touch prd/.gitkeep
```

The `prd/` directory stores committed PRD files saved by `/accept` from epic issues.

### Step 6: Commit Setup

```bash
git add .github/ISSUE_TEMPLATE/ prd/.gitkeep
git commit -m "chore: add GitHub issue templates and prd directory"
```

### Step 7: Output Summary

```
✅ Project setup complete

GitHub Project: {project-name} ({project-url})
  Status field: Backlog → Accepted → Implementing → In Review → Done

Labels created:
  story, bug, epic, priority:high, priority:medium, priority:low, accepted, implementing

Issue templates:
  .github/ISSUE_TEMPLATE/story.yml    — User stories with Gherkin acceptance criteria
  .github/ISSUE_TEMPLATE/bug.yml      — Bug reports with reproduction steps + Gherkin
  .github/ISSUE_TEMPLATE/epic.yml     — Epics/PRDs with user stories + requirements
  .github/ISSUE_TEMPLATE/config.yml   — Blank issues disabled

prd/ directory: created (for committed PRD files)

Next steps:
  1. Push to remote: git push origin main
  2. Create your first issue on GitHub using the templates
  3. Accept it: /accept #1
```

---

## Error Handling

### gh CLI Not Found

```
❌ gh CLI not found

Install: https://cli.github.com
```

### Labels Already Exist

Non-fatal — skip the existing label and continue.

### Project Creation Fails

```
⚠️ Could not create GitHub Project automatically.

Create it manually:
  1. Go to https://github.com/{org}/{repo}/projects
  2. Create a new Board project
  3. Add Status field options: Backlog, Accepted, Implementing, In Review, Done
```

---

## Related Commands

- `/accept {input}` — Accept a GitHub Issue or PRD file into Conductor tracks
- `/implement-track {id}` — Implement a single track
- `/implement-prd {slug}` — Implement all tracks from a PRD
- `/prime` — Load project context (run after setup)
