# Prompt Engineering Template

Use this template to create structured prompts for agents/tools.  
Each section below explains what to include and why it matters.

## Metadata

Use frontmatter-style metadata so the runner can parse prompt capabilities.

```yaml
---
description:
argument-hint:
allowed-tools:
---
```

- `description`: one-line summary of what this prompt does.
- `argument-hint`: expected user input shape (for example, `path/to/file` or `project-name`).
- `allowed-tools`: explicit tool allowlist for safer execution.

## Title

Give the prompt a clear, specific title.

```md
# <Prompt Title>
```

Under the title, add one short sentence describing the goal.

Example:
`Execute the Workflow and Report sections. Read Instructions first, then resolve all required Variables.`

## Variables

Define reusable values that appear across sections.  
Use uppercase names for readability and consistency.

```md
VARIABLE_EXAMPLE: $1
DEFINED_VARIABLE_EXAMPLE: ./path/to/plan
OUTPUT_DIRECTORY: ./report
```

Guidelines:
- Keep variable names stable once published.
- Prefer relative paths unless an absolute path is required.
- Reference these variables verbatim in `Workflow`, `Command`, and `Report`.
Example:
- `argument-hint`: `plan_path`
- `Variables` section: `PLAN_PATH: $ARGUMENT`

## Instructions

Add supporting constraints that affect how the workflow is executed.
This section should be behavioral/policy guidance, not command execution steps.

Common examples:
- coding style rules
- safety constraints
- file editing constraints
- agent behavior notes

Keep this concise. Put step-by-step actions in `Workflow`, not here.

## Workflow

List ordered execution steps the agent should follow.

```md
1. Validate required variables and inputs.
2. Execute implementation steps.
3. Run verification checks.
4. Generate report output.
```

Guidelines:
- Write imperative actions (for example, `Run tests`, `Update file`, `Summarize output`).
- Keep steps atomic and sequential.
- If a step has prerequisites, state them explicitly.

## Command

Document project commands to run for this prompt.

Example:

```bash
npm run lint
npm test
npm run build
```

Only include commands relevant to this prompt.

## Codebase Structure

Provide a quick map of important paths so the agent can navigate faster.

Example:

```md
src/        - application source
tests/      - test suite
docs/       - documentation
```

Focus on high-signal directories/files only.

## Documentation

List supporting docs and references used by this prompt.

Example:

```md
/workspaces/notes/docs/ARCHITECTURE.md - system boundaries and design
/workspaces/notes/docs/API.md - API contracts and payloads
https://example.com/sdk/docs - third-party SDK reference
```

## Report

Define the required output format and location.

Example:

```md
Write a summary of changes to OUTPUT_DIRECTORY/REPORT.md.
Include:
1. Files changed
2. Commands run
3. Validation results
4. Follow-up items
```

Be explicit about where the report goes and what it must contain.

## Example Workflow: Code Review (Current Branch vs master)

````md
---
description: Review current git branch against master and produce a risk-focused report.
argument-hint: BASE_BRANCH
allowed-tools: Read, Bash, Write, WebFetch
---

# Branch Code Review Prompt
Review all changes in the current branch compared to `master`.

## Variables
BASE_BRANCH: $1
OUTPUT_DIRECTORY: ./report
REPORT_FILE: BRANCH_REVIEW.md

## Instructions
- Focus on bugs, regressions, security issues, and missing tests.
- Prioritize findings by severity: high, medium, low.
- Include file paths and line references for every finding.
- Do not modify code; this prompt is review-only.

## Workflow
1. Validate that `BASE_BRANCH` exists locally or fetch it.
2. Compute diff between current branch and `BASE_BRANCH`.
3. Inspect changed files and identify risks or regressions.
4. Classify findings by severity and add evidence.
5. Write final review report to `OUTPUT_DIRECTORY/REPORT_FILE`.

## Command
```bash
git rev-parse --abbrev-ref HEAD
git fetch origin "$BASE_BRANCH" --quiet
git diff --name-only "$BASE_BRANCH"...HEAD
git diff --stat "$BASE_BRANCH"...HEAD
git diff "$BASE_BRANCH"...HEAD
```

## Codebase Structure
```md
src/        - primary application logic
tests/      - automated tests
docs/       - design notes and specifications
```

## Documentation
```md
https://conventionalcomments.org/ - use this format for all PR review comments
```

## Report
Write review results to `OUTPUT_DIRECTORY/REPORT_FILE` with:
1. Branch and base comparison summary
2. Findings grouped by severity
3. File/line references for each finding
4. Test gaps and recommended follow-ups
5. Overall risk assessment (low/medium/high)
````
