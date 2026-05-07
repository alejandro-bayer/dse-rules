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

All commands assume you are in the dse-apis root:

```bash
cd /workspaces/dse-apis
```

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

### Service Tester (mandatory)

```bash
make run-service-tester env=dev
```
- [ ] Service-tester E2E tests pass against a deployed environment
- [ ] Service-tester logs saved for PR evidence

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

### Adversarial Audit Loop

Run the [Adversarial Audit](adversarial-audit.md) on all changed files. This is a **fix-and-recheck loop**:

1. Run the full adversarial audit
2. If ANY issues found → fix them → re-run the audit from step 1
3. Only proceed when the audit reports **zero issues**

- [ ] Adversarial audit passed with zero issues

---

## Output

When all items pass (including adversarial audit):
```
All checks passed. Adversarial audit clean. Ready for PR.
```

Next step: run the [Create PR](create-pr.md) workflow.

If items fail, list what's pending and fix before proceeding.
