# Design Test Plan

**Mandatory workflow**. Run this after implementation is complete and before the adversarial audit. This workflow designs the manual API test scenarios, generates the `auth curl` commands, and creates the testing notes files that go into the PR.

**This is NOT optional** — reviewers will reject PRs without testing evidence. Every ticket that touches API behavior (new fields, validation rules, enum values, endpoint changes) requires a test plan.

---

## Input

- **ticket-id**: Aha! ticket ID (e.g., `D3P-2068`). Auto-detect from branch name if not provided.
- **feature-name**: Short description (e.g., `job-type`, `update-csw-call`). Auto-detect from branch name if not provided.

```bash
cd /workspaces/dse-apis
BRANCH=$(git branch --show-current)
# Extract: feature/d3p-2068-add-job-type-field → ticket=d3p-2068, feature=add-job-type-field
```

---

## When to run this workflow

| Change type | Test plan required? |
|-------------|-------------------|
| New proto field (enum, string, message) | **YES** — test CRUD with the new field |
| New validation rule (protovalidate, CEL) | **YES** — test valid + invalid + edge cases |
| New endpoint or RPC | **YES** — test all operations |
| New enum values | **YES** — test each value + UNSPECIFIED default |
| Deployment pipeline changes (terraform inputs) | **YES** — test API + downstream worker payload |
| Internal-only refactor (no API surface change) | **NO** — unit tests are sufficient |
| Config/version bump only | **NO** — service-tester is sufficient |
| Documentation-only change | **NO** |

---

## Steps

### 1. Identify what changed

```bash
git diff --name-only origin/main...HEAD
```

Read each changed file. From the diff, determine:
- **Which API endpoints are affected** (Create, Get, List, Update, Delete)
- **What new fields or behaviors were added**
- **What validation rules were added or changed**
- **Whether changes flow to downstream workers** (SQS, terraform, assetstack)

### 2. Determine the test environment

Check where the code is (or will be) deployed:

| Environment | Base URL | When to use |
|-------------|----------|-------------|
| Dev | `https://apis.dse-dev.bayer.com` | Feature branch testing |
| NonProd | `https://apis.dse-np.bayer.com` | After merge to main |

Ask the user: **"Is the branch deployed to dev? If not, deploy it first: `make deploy env=dev`"**

### 3. Identify auth prerequisites

The user needs to be authenticated. Include these commands in the test plan:

```bash
# In Codespaces, fix xdg-open first
sudo ln -sf "$BROWSER" /usr/local/bin/xdg-open

# Login
auth login

# Verify token works
auth curl -- https://apis.dse-dev.bayer.com/v1/jobs?page_size=1 -H "Cwid: <USER_CWID>"
```

Ask the user: **"What is your CWID? (e.g., LSZFW)"** — this is needed for the `Cwid` header in all requests.

### 4. Design test scenarios

For each changed behavior, create test scenarios covering:

#### Required scenarios (always include):

| Category | What to test |
|----------|-------------|
| **Happy path** | Create with the new field explicitly set → verify it appears in the response |
| **Default behavior** | Create WITHOUT the new field → verify default value is applied correctly |
| **Get round-trip** | Create → Get → verify all new fields survive the round-trip |
| **List round-trip** | Create → List → verify new fields appear in list responses |
| **Validation: valid input** | Send valid values for each new validation rule → verify 200 |
| **Validation: invalid input** | Send invalid values → verify correct error code and message |

#### Conditional scenarios (include when applicable):

| Category | When to include | What to test |
|----------|----------------|-------------|
| **Enum values** | New enum field | Test EACH non-UNSPECIFIED value individually |
| **UNSPECIFIED default** | New enum field | Omit the field → verify it defaults to the safe value |
| **Cross-field validation (CEL)** | New CEL rule | Test all cells of the truth table (see rules.md § CEL Cross-Field Validation) |
| **Update / field mask** | Field is updatable | Update only the new field → verify other fields unchanged |
| **Backward compatibility** | Existing resources in DB | Get an OLD resource (created before the field existed) → verify UNSPECIFIED is handled |
| **Pipeline flow** | Terraform/assetstack input | Create → Deploy → check CloudWatch logs for correct downstream payload |
| **Idempotency** | Create/Update endpoint | Send the same request twice → verify no duplicate or error |

### 5. Generate the detailed testing notes file

Create the file at: `testing-files/<ticket-id>-<feature-name>.md`

Use this structure (based on real examples from previous PRs):

```markdown
# Testing Notes — <Feature Name> (PR #<NUMBER>)

<One sentence describing what these tests verify.>

## Prerequisites

1. Branch `<branch-name>` deployed to **<environment>**.
2. Authenticated via `auth login` (see [csgda-auth](https://github.com/bayer-int/csgda-auth)).
3. Replace `<YOUR_CWID>` with your Corporate Worker ID.

### Authentication Commands

| Command | Description |
|---------|-------------|
| `auth login` | Interactive browser login (Azure AD). In Codespaces, run `sudo ln -sf "$BROWSER" /usr/local/bin/xdg-open` first. |
| `auth print-access-token` | Prints the raw JWT token to stdout. |
| `auth curl -- <curl args>` | Wraps `curl` with auto-injected `Authorization` header. Use `--` to separate auth flags from curl flags. |

## Environment

- **Base URL**: `<ENVIRONMENT_URL>`
- **Tenant**: `tenants/DONOTDELETE`
- **CWID**: `<USER_CWID>`

---

## Test 1: <Scenario name>

<One sentence describing the purpose.>

```bash
auth curl -- -X POST <URL> \
  -H "Content-Type: application/json" \
  -H "Cwid: <USER_CWID>" \
  -d '{
    <JSON body with real field names and values from the proto definition>
  }'
```

**Expected**: `<HTTP status>`, `<key fields in response>`

**Actual Result**:
```json
<paste actual response here after executing>
```

**Result**: ✅ PASS / ❌ FAIL

---

## Test 2: <Next scenario>

(repeat format)

---

## Summary

| # | Scenario | Result |
|---|----------|--------|
| 1 | <name> | ✅ / ❌ |
| 2 | <name> | ✅ / ❌ |
| ... | ... | ... |
```

**Field values in curl commands**: Use the REAL proto field names (snake_case as they appear in JSON). Look at the proto definition and the existing test files in `testing-files/` for examples of the correct JSON structure.

**Resource names**: Use the test tenant `tenants/DONOTDELETE` and create test resources with descriptive names (e.g., `jobs/TEST-DEFAULT-JOBTYPE`, `jobs/TEST-TRAINING-JOBTYPE`).

### 6. Generate the PR-ready version

Create a second file at: `testing-files/<ticket-id>-<feature-name>-PR.md`

This is a **condensed copy-paste version** of the same tests, formatted to be pasted directly into the PR's "API Testing" section. Differences from the detailed version:
- No auth prerequisites section (PR readers already know)
- Each test is collapsed in a `<details>` block
- Actual responses are included inline
- Summary table at the end

Structure:

```markdown
## Manual Tests — <Feature Name> (PR #<NUMBER>)

### Prerequisites

1. Code deployed to target environment.
2. Authenticated via `auth login`.
3. Replace `<YOUR_CWID>` with your Corporate Worker ID.

**Environment**: `<URL>` | Tenant: `tenants/DONOTDELETE`

---

## Test 1: <Scenario>

```bash
auth curl -- -X POST <URL> \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{ <body> }'
```

**Expected**: `<status>`, `<key fields>`

**Result**: ✅ Pass

<details><summary>Response</summary>

<actual JSON response>

</details>

---

(repeat for each test)
```

### 7. Execute the tests

After generating the files with placeholder responses:

1. **Ask the user to run each `auth curl` command** and paste the response
2. **Or** run the commands directly if `auth` is authenticated in the terminal
3. Fill in the actual responses in both files
4. Mark each test as PASS or FAIL
5. If any test FAILS → fix the code → re-run the [Adversarial Audit](adversarial-audit.md) → re-run the failing test

### 8. Verify completeness

Before proceeding, verify:
- [ ] Every new/changed API behavior has at least one test scenario
- [ ] Every new validation rule has both a valid and invalid test
- [ ] Every new enum value has its own test
- [ ] Default/UNSPECIFIED behavior is tested
- [ ] Both files exist: `testing-files/<ticket-id>-<feature>.md` and `testing-files/<ticket-id>-<feature>-PR.md`
- [ ] All tests show PASS with actual responses (no placeholders)
- [ ] The PR-ready file is formatted for copy-paste into the PR

---

## Reference: Existing test files

Use these as format examples:

| File | Feature | PR |
|------|---------|-----|
| [d3p-2068-job-type.md](../testing-files/d3p-2068-job-type.md) | `job_type` field (detailed) | #634 |
| [d3p-2068-job-type-PR.md](../testing-files/d3p-2068-job-type-PR.md) | `job_type` field (PR-ready) | #634 |
| [d3p-2140-update-csw-call.md](../testing-files/d3p-2140-update-csw-call.md) | CSW owner info (detailed) | #650 |
| [d3p-2140-update-csw-call-PR.md](../testing-files/d3p-2140-update-csw-call-PR.md) | CSW owner info (PR-ready) | #650 |

Read these files before generating new test plans to match the established format.

---

## Output

Two files created and populated with actual test results:
1. `testing-files/<ticket-id>-<feature>.md` — detailed version with full context
2. `testing-files/<ticket-id>-<feature>-PR.md` — PR-ready copy-paste version

→ Return to [Implementation Checklist](implementation-checklist.md) to continue with the adversarial audit.
