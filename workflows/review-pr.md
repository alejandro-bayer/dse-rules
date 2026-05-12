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

### 3. Local verification (when warranted)

**Decide whether to checkout the branch locally** based on the nature of the changes. Do it when:

- Code logic changed → run the build and lint commands for the project language
- New or modified test files → run the test suite to confirm tests pass
- Proto files changed (Go repos) → run `make generate` and verify generated code is fresh
- Validation logic changed (protovalidate, CEL) → run the specific validation tests
- Helm/deploy changes → run `make lint` to catch K8s manifest issues

**Skip local checkout** when the PR is docs-only, dependency-file-only (`go.mod`, `requirements.txt`, `package.json`), or trivially small (e.g., a one-line constant rename).

**First**, detect the project language from the repo root. Look for `go.mod` (Go), `pyproject.toml` or `setup.py` or `requirements.txt` (Python), `package.json` (Node/TS), or `Makefile`/shell scripts (Shell). Then use the matching command set:

```bash
# Checkout the PR branch locally (same for all languages)
git fetch origin pull/<pr-number>/head:pr-<pr-number>
git checkout pr-<pr-number>
```

#### Go projects (`go.mod` present — e.g. dse-apis)

```bash
go build ./...          # Does it compile?
make lint               # golangci-lint + proto lint
make unit-tests         # Test suite

# If proto changed, verify generated code is fresh:
make generate && git diff --exit-code generated/
```

#### Python projects (`pyproject.toml` / `setup.py` — e.g. dse-sdk)

```bash
pip install -e ".[dev]" 2>/dev/null || pip install -e .   # Install in dev mode
python -m pytest tests/ -x                                # Run tests, stop on first failure
python -m ruff check .                                    # Lint (if ruff configured)
python -m mypy src/     2>/dev/null || true               # Type check (if mypy configured)
```

#### Shell / infrastructure repos (e.g. dse-assetstacks, dse-custom-development-images)

```bash
shellcheck **/*.sh 2>/dev/null || true   # Lint shell scripts
# For Dockerfiles:
docker build --check . 2>/dev/null || true
# For Terraform:
terraform validate 2>/dev/null || true
terraform fmt -check 2>/dev/null || true
```

#### Node/TypeScript projects (`package.json` present)

```bash
npm ci                  # Clean install
npm run lint            # Lint
npm test                # Test suite
npm run build           # Does it compile?
```

```bash
# Return to previous branch when done (same for all)
git checkout -
git branch -D pr-<pr-number>
```

If any check fails, include the **exact error output** in the review findings as a **must-fix** item. The author may not have run CI locally.

> **Relationship to adversarial audit**: This step runs the same verification commands as [adversarial-audit.md](adversarial-audit.md) phases 3–5, but **without the fix loop** — as a reviewer you report failures, you don't fix them.

### 4. Analyze the diff

For each changed file, evaluate against these categories:

#### Correctness
- Does the implementation match what the ticket asked for?
- Are edge cases handled (nil checks, empty strings, enum UNSPECIFIED)?
- Do new proto fields have proper validation rules (protovalidate)?
- Are new enum values handled in switch statements with no silent fallthrough?

#### Architecture & Patterns
Reference [rules.md](../rules.md) for the full list. Key checks:

- **Pattern alignment**: Do new files/functions follow the same patterns as existing code in the same package? Flag if:
  - A new file was created when the functionality belongs in an existing file (e.g., `query_user.go` when `profile.go` already has all similar methods)
  - Standalone functions were added when the package uses struct methods (e.g., `func GetX()` when everything else is `func (c *Client) GetX()`)
  - Infrastructure was rebuilt that already exists elsewhere (token caching, client builders, connection factories)
- **Proto as source of truth**: No hand-edited `generated/` files
- **Logging**: Uses `internal/logging/v2`, not v1 or raw slog
- **Error handling**: `logger.NewError()`, no double-logging
- **Constants**: Terraform input keys use `Key` suffix convention
- **Naming**: Proto fields match Go constants match terraform variable names
- **UNSPECIFIED defaults**: New enums default to safe behavior in Create handlers

#### Anti-patterns to flag
- New file created when functionality belongs in an existing file in the same package
- Standalone function when the package convention is struct methods
- Custom token caching / client construction when a builder already exists (e.g., `buildPapiClient()`)
- `var FuncName = package.Function` monkey-patching pattern for testability
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

### 5. Produce the review

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

### 6. Generate inline comment list

After the review findings, produce a **copy-paste ready** list of inline comments for the user to post on the PR. Each item must include:

- **File** — relative path
- **Line anchor** — the specific line of the *new* code in the diff (not the old code)
- **Comment** — concise, B2-English, self-explanatory. Do NOT reference rules.md or internal docs
- **Fix example** (when applicable) — a short code snippet showing the fix, OR a reference to an existing file in the repo where the correct pattern is already implemented

Format each comment as:

```
N. **`path/to/file.go`** — `<line content or identifier>`

> <Comment text>
> ```go  (or protobuf, etc.)
> <fix example if applicable>
> ```
```

**Rules for this section:**
- One entry per finding (must-fix, should-fix, nitpick). Skip positives
- Comments must stand alone — a reader seeing only the inline comment should understand the problem and the solution without needing the full review
- When the fix is "do it like it's already done elsewhere", include the file path and a brief snippet of the existing pattern
- When the fix is "add something missing", show the final code with the addition
- Keep comments under 6 lines of prose (code blocks don't count)

### 7. Submit the review

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
