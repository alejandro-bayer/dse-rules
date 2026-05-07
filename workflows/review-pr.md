# Review PR

Workflow for reviewing a Pull Request from a team member against the project's coding standards and the ticket requirements.

---

## Input

- **pr-number**: The PR number to review (e.g., `635`)
- **repo** (OPTIONAL): Repository in `owner/repo` format. Default: `bayer-int/dse-apis`

---

## Steps

### 1. Gather context

```bash
gh pr view <pr-number> --json title,body,headRefName,baseRefName,files,additions,deletions,author
gh pr diff <pr-number>
```

Extract from the PR:
- The Aha! ticket ID (from the body or branch name)
- The stated purpose/summary
- The list of changed files

### 2. Understand the ticket

From the Aha! link in the PR body, understand:
- What was requested
- Acceptance criteria (if any)
- Scope boundaries

### 3. Analyze the diff

For each changed file, evaluate against these categories:

#### Correctness
- Does the implementation match what the ticket asked for?
- Are edge cases handled (nil checks, empty strings, enum UNSPECIFIED)?
- Do new proto fields have proper validation rules (protovalidate)?
- Are new enum values handled in switch statements with no silent fallthrough?

#### Architecture & Patterns
Reference [rules.md](../rules.md) for the full list. Key checks:

- **Proto as source of truth**: No hand-edited `generated/` files
- **Logging**: Uses `internal/logging/v2`, not v1 or raw slog
- **Error handling**: `logger.NewError()`, no double-logging
- **Constants**: Terraform input keys use `Key` suffix convention
- **Naming**: Proto fields match Go constants match terraform variable names
- **UNSPECIFIED defaults**: New enums default to safe behavior in Create handlers

#### Anti-patterns to flag
- Direct `slog` import instead of `internal/logging/v2`
- `logger.Error()` followed by `return err` (double-logging)
- Boolean fields in proto (should be enums with UNSPECIFIED)
- Hardcoded strings that should be constants in `svcconfig.go`
- Missing server-side validation for cross-resource references
- `log.Fatalf` or `os.Exit` outside of `main.go`
- Raw string concatenation for resource names (use `resourcename` package)

#### Testing
- Are there unit tests for new business logic?
- Do tests use the table-driven pattern?
- For pipeline features: is there E2E evidence (terraform payload, CloudWatch logs)?
- Are protovalidate rules tested?

#### Cross-repo coordination
If the PR touches terraform inputs, version bumps, or SQS message formats:
- Does `AssetStackJobTaskVersion` match the referenced git tag?
- Are `deploy/values/*/values.yaml` files consistent?
- Are new terraform inputs declared in both constants AND the external repos?

#### PR quality
- Is the summary clear about WHY, not just WHAT?
- Is the Aha! link present and correct?
- Are testing sections filled with actual evidence (screenshots, logs)?
- Is `service-tester` evidence included?

### 4. Produce the review

Structure the review as:

```markdown
## PR #<number> Review: <title>

### Summary
<1-2 sentences: what the PR does and whether it meets the ticket requirements>

### Findings

#### Must Fix (blocking)
<Issues that must be resolved before merging. Include file, line reference, and explanation.>

1. **[file:line]** <Issue description>
   - Why: <explanation>
   - Suggested fix: <concrete suggestion>

#### Should Fix (non-blocking)
<Improvements that are worth doing but don't block merge.>

1. **[file:line]** <Issue description>

#### Nitpicks
<Style, naming, minor improvements. Optional to address.>

1. **[file:line]** <Observation>

#### Positive
<What's done well. Acknowledge good patterns, thorough testing, clean code.>

### Verdict
- [ ] **Approve** — Ready to merge
- [ ] **Request changes** — Must-fix items need resolution
- [ ] **Comment** — No blocking issues, but has suggestions
```

### 5. Submit the review

If using `gh`:
```bash
# Approve
gh pr review <pr-number> --approve --body "LGTM. <brief comment>"

# Request changes
gh pr review <pr-number> --request-changes --body-file review.md

# Comment only
gh pr review <pr-number> --comment --body-file review.md
```

For inline comments on specific lines, use the GitHub web UI or:
```bash
gh api repos/{owner}/{repo}/pulls/<pr-number>/comments \
  -f body="<comment>" \
  -f path="<file>" \
  -F line=<line-number> \
  -f commit_id="$(gh pr view <pr-number> --json headRefOid -q .headRefOid)"
```

---

## Review severity guide

| Severity | When to use | Example |
|----------|------------|---------|
| **Must fix** | Bugs, security issues, broken logic, missing validation | Missing nil check on a dereferenced pointer |
| **Should fix** | Code quality, maintainability, patterns not followed | Using v1 logger instead of v2 |
| **Nitpick** | Style, naming preferences, minor cleanup | Variable could have a clearer name |
| **Positive** | Good code worth calling out | Clean table-driven tests, good error messages |

---

## Guardrails

- Always read the full diff before commenting — avoid context-free line comments
- Check testing evidence exists and is real (not placeholders)
- If you see cross-repo impacts, verify version consistency
- Don't block on style-only issues — use nitpick severity
- If unsure about a pattern, reference the specific section of rules.md
