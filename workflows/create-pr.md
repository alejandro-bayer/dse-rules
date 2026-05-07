# Create PR

Commit changes, push to remote, and create a Pull Request with the team's required format.

---

## Prerequisites

Run the [Implementation Checklist](implementation-checklist.md) first. All items must pass before creating a PR.

---

## Steps

### 1. Review the diff

```bash
git status
git diff --stat origin/main...HEAD
```

Identify:
- Which files changed and why
- The Aha! ticket ID from the branch name (e.g., `feature/d3p-2068-...` → `D3P-2068`)
- Whether this is a feature, fix, or hotfix

### 2. Stage and commit

Stage specific files — avoid `git add .` to prevent committing .env or generated cruft:

```bash
git add <specific files>
```

Commit message format:
```
<Concise description of what changed> (#PR_NUMBER if amending)
```

The team does not enforce a strict commit message regex, but commits should be descriptive. Examples from the repo:
- `Add default job type handling for legacy jobs in asset stack deployment`
- `update asset_stack_job_task_version to v0.0.9 across all environments`
- `fixing pr comment by respecting enum options training and standard`

### 3. Push to remote

```bash
git push origin <branch-name>
# First push: git push -u origin <branch-name>
```

### 4. Generate PR description

Use the template below. Fill every section — reviewers will reject PRs with empty sections.

```markdown
# Summary

<2-3 sentences explaining WHY this change is needed, not just what changed. Reference the business context.>

## Changes

<Bullet list of concrete changes. Group by area: proto, Go logic, tests, deploy, etc.>

- Added/Modified `proto/dse/<domain>/v1/<domain>.proto`: <what changed>
- Added/Modified `internal/<domain>/<file>.go`: <what changed>
- Added/Modified `internal/<domain>/<file>_test.go`: <what tests>
- Updated `deploy/values/*/values.yaml`: <version bumps>
- Updated `internal/svcconfig/svcconfig.go`: <config changes>

## Aha! Feature name & link

[D3P-XXXX](https://monsanto.aha.io/features/D3P-XXXX)

## Dependencies

<Link to other PRs this depends on, or "None">

## Testing Performed

- [x] Ran `make lint`
- [x] Ran `make ci`
- [x] Ran `go build ./...`
- [x] Ran `bash ./scripts/check-log-usage.sh`
- [x] Ran `service-tester`

### Summary & Environment Tested In

Env(s) tested in:

- [ ] local
- [ ] dev
- [ ] np

<High-level summary of testing approach and results>

### Screenshots

<details>
<summary>Screenshots</summary>

- `make lint`
<screenshot>

- `make ci`
<screenshot>

- `go build ./...`
<screenshot>

- `bash ./scripts/check-log-usage.sh`
<screenshot>

- `service-tester`
<screenshot>

</details>

### Service Tester

`service-tester` logs:

<details>
<summary>Deployment (<code>make tail-logs</code>) logs</summary>

COPY-PASTE HERE

</details>

<details>
<summary><code>service-tester</code> client logs</summary>

COPY-PASTE HERE

</details>

### API Testing (if applicable)

<For each test scenario, include the curl command and actual response. Collapse long outputs in <details> tags.>

<details>
<summary>Test 1: <description></summary>

Request:
auth curl -- -X POST/GET <URL> ...

Response:
{ ... }

Result: PASS/FAIL

</details>

### E2E Pipeline Testing (if applicable)

<Include terraform worker payload from CloudWatch logs showing new inputs are received correctly.>

<details>
<summary>Terraform Worker payload</summary>

{ ... JSON from CloudWatch ... }

</details>

## Tech debt this creates

<Describe any tech debt, or "None". If tech debt exists, reference or create an Aha! ticket.>

## Reviewer to check

- [ ] `service-tester` was ran as part of testing notes
- [ ] Code formatting consistency (style, naming convention, idiomatic code)
- [ ] Presence of docstring comments, TODO: comments, logic explanations where needed
- [ ] Log statements use ONLY `internal/logging/v2` package
- [ ] If unit-testable, presence of meaningful unit tests
```

### 5. Save the PR description to a file

```bash
# Save the filled-in template above to PR.md in the repo root
cat > PR.md << 'EOF'
<paste your filled-in PR description here>
EOF
```

> **Note**: `PR.md` is a temporary file for `gh pr create`. Delete it after the PR is created, or add it to `.gitignore`.

### 6. Create the PR

```bash
gh pr create \
  --title "<Concise PR title>" \
  --body-file PR.md \
  --base main
```

If a PR already exists for the branch:
```bash
gh pr edit <PR_NUMBER> --body-file PR.md
```

Clean up:
```bash
rm PR.md
```

### 7. Verify

```bash
gh pr view --web  # Open in browser to verify formatting
```

Check that:
- All sections are filled
- Screenshots are visible
- CI checks are passing
- The Aha! link works

---

## Iterating on PR Comments

After reviewers leave comments:

1. Read all comments carefully
2. For each comment, determine if it's:
   - **A valid fix** → implement it
   - **A misunderstanding** → explain with evidence (truth tables, cross-repo constraints, etc.)
   - **An enhancement beyond scope** → suggest a follow-up ticket
3. After making changes, push and reply to each comment thread
4. Re-run `make ci` and `go build ./...` after changes
5. If new tests are needed, add them and update the testing section of the PR

### Common reviewer patterns to anticipate

| Reviewer concern | How to address proactively |
|-----------------|---------------------------|
| "Can you rename this field?" | If cross-repo, explain the naming chain (proto → constant → terraform) |
| "Can you simplify this validation?" | Show the truth table for edge cases |
| "Add testing evidence" | Include screenshots + curl responses + CloudWatch logs in the PR |
| "Add server-side validation" | If referencing another resource, validate it exists via gRPC |
| "Add Key suffix to constants" | Follow the repo convention for terraform input key constants |

---

## Guardrails

- Never create a PR with empty testing sections
- Never push to `main` directly — always via PR
- Never skip `make ci` before pushing
- Always include the Aha! ticket link
- Always include `service-tester` results
- If adding E2E pipeline changes, always include terraform worker payload evidence
