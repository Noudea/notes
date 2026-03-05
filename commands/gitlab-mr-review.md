---
description: Review a GitLab merge request using glab CLI and post a single review comment using Conventional Comments format.
argument-hint: MR_NUMBER [PROJECT_PATH]
allowed-tools: Read, Bash(glab *), Bash(git *), Bash(cat *), Bash(mkdir *), Bash(date *), Write, WebFetch
---

# GitLab Merge Request Review

Fetch the specified merge request using glab, analyze all changes, and post a single comprehensive review comment using `glab mr note`. All findings must follow the [Conventional Comments](https://conventionalcomments.org/) format and reference the exact file path and line number.

## Variables

MR*NUMBER: $1
PROJECT_PATH: ${2:-$(git remote get-url origin | sed 's/.*[:/]\(._\)\.git/\1/')}
OUTPUT_DIRECTORY: ./mr-reviews
REPORT_FILE: MR_${MR*NUMBER}\_REVIEW.md
REVIEW_TIMESTAMP: $(date +%Y-%m-%d*%H-%M-%S)

## Conventional Comments Format

All review findings MUST use Conventional Comments format:

```
<label> [decorations]: <subject>

[discussion]
```

### Labels

| Label         | Usage                                                                  |
| ------------- | ---------------------------------------------------------------------- |
| `praise:`     | Highlight something positive. Leave at least one per review.           |
| `nitpick:`    | Trivial preference-based requests. Non-blocking by nature.             |
| `suggestion:` | Propose improvements. Be explicit about what and why.                  |
| `issue:`      | Highlight specific problems. Pair with a `suggestion:` when possible.  |
| `todo:`       | Small, trivial, but necessary changes.                                 |
| `question:`   | Potential concern — ask for clarification or investigation.            |
| `thought:`    | Non-blocking idea that emerged during review.                          |
| `chore:`      | Simple tasks needed before acceptance (link to process docs).          |
| `note:`       | Non-blocking highlight for the reader's awareness.                     |
| `typo:`       | Misspelling (like `todo:` but specific).                               |
| `polish:`     | Nothing wrong, but could be immediately improved (like `suggestion:`). |

### Decorations

| Decoration       | Meaning                                                    |
| ---------------- | ---------------------------------------------------------- |
| `(non-blocking)` | Should NOT prevent the MR from being accepted.             |
| `(blocking)`     | MUST be resolved before merge.                             |
| `(if-minor)`     | Resolve only if the change ends up being minor or trivial. |
| `(security)`     | Related to a security concern.                             |
| `(performance)`  | Related to a performance concern.                          |
| `(test)`         | Related to test coverage or test quality.                  |
| `(ux)`           | Related to user experience.                                |

### Severity to Label Mapping

| Severity | Conventional Comments Approach                               |
| -------- | ------------------------------------------------------------ |
| CRITICAL | `issue (blocking):` or `issue (blocking,security):`          |
| HIGH     | `issue (blocking):` paired with `suggestion:`                |
| MEDIUM   | `suggestion:` or `issue (non-blocking):`                     |
| LOW      | `nitpick:`, `polish:`, `thought (non-blocking):`, or `typo:` |
| POSITIVE | `praise:`                                                    |

### Finding Format Within the Report

Each finding references file path and line number, then uses Conventional Comments:

```
#### `path/to/file.ext` (L42)

**issue (blocking,security):** User input is interpolated directly into the SQL query.

This opens the door to SQL injection. Consider using parameterized queries or an ORM method instead.
```

For multiple findings in one file, group them under the same file header:

```
#### `path/to/file.ext`

**L42** — **issue (blocking,security):** User input is interpolated directly into the SQL query.

This opens the door to SQL injection. Consider using parameterized queries or an ORM method instead.

---

**L78** — **suggestion (non-blocking):** This loop could be replaced with a list comprehension.

It would reduce the line count and improve readability without changing behavior.
```

## Instructions

- Focus on code quality, bugs, security vulnerabilities, and performance implications.
- Prioritize findings by severity: CRITICAL, HIGH, MEDIUM, LOW.
- Include file paths and line numbers for every finding.
- Use constructive language; assume positive intent from the MR author.
- Check for missing tests, documentation gaps, and breaking changes.
- Verify adherence to project coding standards and conventions.
- Do not modify code during review; this is analysis-only.
- ALL findings MUST follow Conventional Comments format.
- After generating the report, ask if I want to post it as a single MR comment.

## Workflow

1. Validate that MR_NUMBER is provided and is a valid number.
2. Verify glab is installed and authenticated:
   - Run `glab auth status` to confirm authentication.
3. Fetch MR details using glab:
   - MR title, description, author.
   - Source and target branches.
   - Current status, labels, and approvals.
   - Related issues and milestones.
4. Retrieve the complete diff using `glab mr diff`.
5. Get commit history using `glab mr commits`.
6. Analyze changed files systematically:
   - Identify potential bugs and logic errors.
   - Flag security concerns (SQL injection, XSS, auth issues, exposed secrets).
   - Check for performance anti-patterns (N+1 queries, inefficient algorithms).
   - Verify test coverage for new/modified code.
   - Review error handling and edge cases.
   - Check for code duplication and maintainability issues.
7. Check for breaking changes and migration requirements.
8. Assess commit quality and conventional commit compliance.
9. Review CI/CD pipeline status.
10. Generate structured review report with findings grouped by severity.
11. Write report to OUTPUT_DIRECTORY/REPORT_FILE.
12. Display summary and ask if I want to post the review to GitLab.
13. If confirmed, post the full report as a single MR comment using `glab mr note`.

## Commands

### Authentication & Setup

```bash
# Verify glab is installed and authenticated
glab version
glab auth status

# Get current git repository info
git rev-parse --git-dir
git remote get-url origin
```

### Fetch MR Data

```bash
# Fetch MR overview
glab mr view "$MR_NUMBER" --repo "$PROJECT_PATH"

# Get MR diff
glab mr diff "$MR_NUMBER" --repo "$PROJECT_PATH"

# Get MR commits
glab mr commits "$MR_NUMBER" --repo "$PROJECT_PATH"
```

### Post Review Comment

```bash
# Post the full review as a single MR comment
glab mr note "$MR_NUMBER" --repo "$PROJECT_PATH" -F "$OUTPUT_DIRECTORY/$REPORT_FILE"
```

### Other Useful Commands

```bash
# List existing MR notes/comments
glab mr note list "$MR_NUMBER" --repo "$PROJECT_PATH"

# Approve the MR
glab mr approve "$MR_NUMBER" --repo "$PROJECT_PATH"

# Request changes
glab mr note "$MR_NUMBER" --repo "$PROJECT_PATH" -m "**issue (blocking):** Changes requested — see review report above."

# Open MR in browser
glab mr view "$MR_NUMBER" --repo "$PROJECT_PATH" --web
```

## Documentation

```
https://conventionalcomments.org/ - Conventional Comments standard
https://docs.gitlab.com/cli/mr/note/ - glab mr note documentation
https://docs.gitlab.com/ee/development/code_review.html - GitLab code review guidelines
https://google.github.io/eng-practices/review/ - Google's code review best practices
https://owasp.org/www-project-code-review-guide/ - OWASP code review guide for security
https://gitlab.com/gitlab-org/cli - glab CLI documentation
```

## Report

Write review results to `OUTPUT_DIRECTORY/REPORT_FILE` with the following structure:

### Header

```
# MR !MR_NUMBER Review Report

**Title**: [MR Title from glab]
**Author**: [Author from glab]
**Created**: [Creation date]
**Source**: [source-branch] → **Target**: [target-branch]
**Status**: [open/merged/closed]
**Review Date**: REVIEW_TIMESTAMP
**Reviewer**: Claude Code AI Assistant
**Comment Format**: [Conventional Comments](https://conventionalcomments.org/)
```

### Executive Summary

- **Overall Assessment**: Approve / Request Changes / Comment
- **Risk Level**: LOW / MEDIUM / HIGH / CRITICAL
- **Key Concerns**: 2-3 sentence summary of main issues
- **CI/CD Status**: Passing / Failing / Pending

### Findings by Severity

#### 🔴 CRITICAL

```
#### `path/to/file.ext` (L42)

**issue (blocking):** Brief description of the critical issue.

Impact explanation and specific actionable fix recommendation.
```

#### 🟠 HIGH

```
#### `path/to/file.ext` (L78)

**issue (blocking):** Brief description.

Discussion and recommendation paired with suggestion.
```

#### 🟡 MEDIUM

```
#### `path/to/file.ext` (L103)

**suggestion:** or **issue (non-blocking):** Brief description.

Discussion and reasoning.
```

#### 🟢 LOW

```
#### `path/to/file.ext` (L15)

**nitpick:** / **polish:** / **typo:** / **thought (non-blocking):** Brief description.

Optional discussion.
```

### ✅ Positive Observations

```
#### `path/to/file.ext` (L25-40)

**praise:** Description of well-implemented feature or good practice.
```

### Test Coverage Analysis

- **New code covered by tests**: YES / NO / PARTIAL
- **Edge cases tested**: YES / NO / PARTIAL
- **Test quality**: Good / Adequate / Needs Improvement
- **Test gaps identified**:
  1. `**todo (test):**` [Specific gap]
  2. `**suggestion (test):**` [Specific gap]

### Breaking Changes

- **API changes**: YES / NO — [details if yes]
- **Database schema changes**: YES / NO — [details if yes]
- **Configuration changes**: YES / NO — [details if yes]
- **Migration required**: YES / NO — [migration plan if yes]
- **Deprecations**: [list any deprecated features]

### Documentation Review

- **Code comments adequate**: YES / NO
- **README updated**: YES / NO / N/A
- **API docs updated**: YES / NO / N/A
- **CHANGELOG updated**: YES / NO / N/A
- **Migration guide provided**: YES / NO / N/A

### Commit Quality

- **Commit messages follow conventions**: YES / NO
- **Logical commit structure**: YES / NO
- **Commits are atomic**: YES / NO
- **Co-authored properly**: YES / NO / N/A
- **Commit message details**: [specific feedback]

### Security Considerations

- **Secrets exposed**: YES / NO
- **SQL injection risks**: YES / NO
- **XSS vulnerabilities**: YES / NO
- **Authentication/Authorization issues**: YES / NO
- **Dependency vulnerabilities**: YES / NO
- **Details**: [specific security concerns if any]

### Performance Considerations

- **Database queries optimized**: YES / NO / N/A
- **Potential N+1 queries**: YES / NO
- **Caching implemented**: YES / NO / N/A
- **Algorithm efficiency**: Good / Adequate / Needs Improvement
- **Details**: [specific performance concerns]

### Recommendations (Prioritized)

1. `**issue (blocking):**` [Specific actionable recommendation]
2. `**suggestion:**` [...]
3. `**nitpick:**` [...]

### Approval Decision

- [ ] ✅ **APPROVE** — Ready to merge as-is
- [ ] 🔄 **REQUEST CHANGES** — Blocking issues must be addressed before merge
- [ ] 💬 **COMMENT** — Feedback provided, author decides next steps

---

**Review completed**: REVIEW_TIMESTAMP
**Command**: `glab mr view !MR_NUMBER`
**Reviewer**: Claude Code AI Assistant
**Comment Standard**: [Conventional Comments](https://conventionalcomments.org/)

---

### Post Review Actions

After reviewing this report, you can:

- **Post review to GitLab**: `glab mr note MR_NUMBER -F OUTPUT_DIRECTORY/REPORT_FILE`
- **Approve the MR**: `glab mr approve MR_NUMBER`
- **Request changes**: `glab mr note MR_NUMBER -m "**issue (blocking):** Changes requested — see review report."`
- **View in browser**: `glab mr view MR_NUMBER --web`
