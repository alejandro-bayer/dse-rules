# Implementation Checklist (Definition of Done)

Run this checklist before creating a PR. Every item must pass. If something fails, fix it before proceeding.

---

## Input

- **ticket-id** (OPTIONAL): Aha! ticket ID to reference. Auto-detect from branch name if not provided.

If not provided:
```bash
git branch --show-current
# Extract D3P-XXXX from feature/d3p-XXXX-...
```

---

## Checklist

### Code Quality

```bash
make lint
```
- [ ] Protobuf lint passes
- [ ] golangci-lint passes (formatting + static analysis)

```bash
make ci
```
- [ ] Full pre-commit hook suite passes (proto lint, code gen freshness, go mod tidy, govulncheck, unit tests, logging checks, proto policy checks, shell linting, K8s manifest validation)

```bash
go build ./...
```
- [ ] Entire module compiles cleanly

```bash
bash ./scripts/check-log-usage.sh
```
- [ ] No direct `log/slog` imports, logger.With() uses `=` not `:=`, no manual function_name keys

### Architecture & Code

- [ ] Implementation matches the approved plan scope (no scope creep)
- [ ] Proto definitions are the source of truth — `generated/` files NOT hand-edited
- [ ] `internal/logging/v2` used for all new logging (never v1 or raw slog)
- [ ] Error handling uses `logger.NewError()` — no double-logging
- [ ] New enum fields default UNSPECIFIED to a safe value (both in Create handler AND in downstream logic)
- [ ] Constants follow `Key` suffix convention for terraform input names
- [ ] No hardcoded secrets, credentials, or environment-specific values in code

### Cross-Repo Coordination (if applicable)

- [ ] `AssetStackJobTaskVersion` in svcconfig.go matches the git tag in dse-assetstacks
- [ ] All `deploy/values/*/values.yaml` files updated with the same version
- [ ] New terraform inputs declared in `dse-assetstacks` (assetstack.yaml) and `dse-terraform-modules` (variables.tf)
- [ ] Field names match across proto → constants → terraform variables

### Testing

- [ ] Unit tests written for new business logic (`//go:build unit`)
- [ ] Protovalidate tests written for new proto validation rules
- [ ] Table-driven test pattern used where applicable
- [ ] Tests pass: `make unit-tests`

### E2E Testing (for pipeline features)

If the ticket modifies the deployment pipeline (new terraform inputs, assetstack version bumps, SQS message format changes):

- [ ] Deployed to dev or nonprod
- [ ] API-level tests: Create/Get resources with new fields, verify validation
- [ ] Terraform worker receives correct inputs (check CloudWatch logs)
- [ ] Test evidence documented (see [Jobs E2E Testing Guide](../guides/jobs-e2e-testing.md))
- [ ] Testing notes file created in `testing-files/` with actual curl commands and responses

### Documentation

- [ ] CONTRIBUTING.md updated if new dev setup steps added
- [ ] Relevant troubleshooting docs updated if new known issues found
- [ ] Postman collection updated if API surface changed

### Final Verification

- [ ] All changes committed to feature branch
- [ ] No merge conflicts with main: `git fetch origin && git diff --stat origin/main...HEAD`
- [ ] Branch is up to date with main: `git merge origin/main` (resolve conflicts if any)

---

## Output

When all items pass:
```
All checks passed. Ready for PR.
```

Next step: run the [Create PR](create-pr.md) workflow.

If items fail, list what's pending and fix before proceeding.
