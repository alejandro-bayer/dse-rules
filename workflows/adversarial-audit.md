# Adversarial Audit

Systematic verification loop that catches issues before they reach reviewers. Run this after completing implementation and after every round of PR comment fixes.

**Philosophy**: Your job is not to confirm things work — it's to prove they fail. If you can't break it with evidence, it passes.

---

## Principles

- **Descartes**: Doubt everything. If you can't verify it by executing a command with observable output, don't assume it's true. "Should work" doesn't exist.
- **Occam invertida**: If something can fail for 3 reasons, verify all 3. Don't stop at the first one that passes.
- **Dialéctica de Hegel**: For each component: (1) thesis — "this works", (2) antithesis — "this can fail when...", (3) synthesis — a test that resolves the contradiction with evidence.

---

## Rules

1. Read EACH changed file line by line. Don't summarize. Don't assume content.
2. For each verification, execute the real command and show the output.
3. If you find a problem, don't just report it — propose and apply the exact correction.
4. If a file references another, go read that one too.
5. Invent destructive tests: what if a resource is missing? A network error? Executed twice? Environment changes?
6. Every check: **VERIFIED ✓** or **FAILED ✗** with evidence.

---

## Phases

### 1. Inventory

Identify all changed files since branching from main:

```bash
cd /workspaces/dse-apis
git diff --name-only origin/main...HEAD
```

Read each changed file completely. Understand what was added, modified, and removed.

### 2. Pattern alignment (NEW — prevents PR #650-type failures)

For each new file or function added, verify it follows existing patterns:

```bash
# For each new file: does a similar file already exist in the same package?
ls internal/<package>/

# For each new function: does the package use struct methods or standalone functions?
grep -n "^func " internal/<package>/*.go | head -20
grep -n "^func (" internal/<package>/*.go | head -20

# For each new helper: does a similar helper already exist?
grep -rn "func build\|func new\|func New" internal/clients/ | head -20
```

Check:
- **No new files** when the functionality belongs in an existing file (e.g., don't create `query_user.go` when `profile.go` already has all similar methods)
- **No standalone functions** when the package uses struct methods (e.g., `func GetX()` when everything else is `func (c *Client) GetX()`)
- **No rebuilt infrastructure** — token caching, client builders, connection pooling — that already exists elsewhere
- **New code reads like it was written by the same author** as existing code in the package

If ANY of these fail, the implementation must be refactored before proceeding.

### 3. Internal consistency

- Do changed files reference each other correctly? (imports, constants, proto field names)
- Do paths, names, and versions match across proto → Go → constants → deploy values?
- Are generated files fresh? (`make generate` then `git diff`)
- Any typos in field names, log messages, or error strings?

### 4. Build & lint verification

Execute each command and verify clean output:

```bash
make lint
make ci
go build ./...
bash ./scripts/check-log-usage.sh
```

Every command must exit 0 with no warnings. If any fails, fix it immediately.

### 5. Test verification

```bash
make unit-tests
```

For proto changes, verify validation rules work:
- Valid inputs pass
- Invalid inputs are rejected with correct error messages
- Edge cases: empty strings, UNSPECIFIED enums, missing required fields

### 6. Adversarial tests

Invent at least 5 tests that try to break the implementation. Think:

- What happens with UNSPECIFIED enum values in legacy database records?
- What if a referenced resource doesn't exist?
- What if the same request is sent twice?
- Do new fields survive a round-trip (Create → Get)?
- Are new constants actually used, or are they dead code?
- Does the field naming chain hold? (proto → `.String()` → constant → terraform variable)

### 7. Cross-file coherence

For each new field or behavior:
- Is it in the proto definition?
- Is it in the generated code? (run `make generate` + `git diff`)
- Is it handled in the Create handler?
- Is it handled in the Get/List response?
- Is it handled in deployment input mapping (if pipeline feature)?
- Is it tested?
- Is it in the service-tester (if API surface changed)?

---

## Report format

```markdown
## Audit Report

### Summary
[X verified, Y failed, Z warnings]

### Critical (✗)
[What failed, evidence, correction applied]

### Warnings (⚠)
[Works today but fragile]

### Verified (✓)
[Brief list]

### Adversarial tests
[What you invented, result]
```

---

## The Loop

This audit runs in a **fix-and-recheck loop**:

```
┌─────────────────────────────┐
│   Run adversarial audit     │
└─────────────┬───────────────┘
              │
         Found issues?
        ╱            ╲
      Yes              No
       │                │
  Fix all issues    ✓ AUDIT PASSED
       │            (proceed to next step)
       │
  ┌────┴────┐
  │ Re-run  │
  │  audit  │──→ (back to top)
  └─────────┘
```

**The loop continues until zero issues are found.** No exceptions.

After each fix round, re-run ALL phases — not just the one that failed. A fix in one area can break another.

---

## When to run this audit

| Trigger | Context |
|---------|---------|
| After implementation, before creating PR | Run from [Implementation Checklist](implementation-checklist.md) |
| After addressing PR reviewer comments | Run from [Create PR](create-pr.md) iteration section |
| After any significant code change | Self-triggered when scope of changes warrants it |

---

## Guardrails

- Never skip a phase because "it probably works"
- Never mark something ✓ without executing the verification command
- Never proceed to the next workflow step with any ✗ items remaining
- If a fix introduces new changes, the loop resets — run the full audit again
