# Start Ticket

Pick up an Aha! ticket, create the feature branch, understand the scope, and plan before writing any code.

---

## Input

The user provides:
- **Aha! ticket ID** (e.g., `D3P-2068`) and a description or link to the feature.
- Optionally, a paste of the ticket body/acceptance criteria.

If the user doesn't provide a ticket ID, ask: **"What's the Aha! ticket ID? (e.g., D3P-2068)"**

---

## Steps

### 1. Load project context

Read these files (in parallel if possible):
- `rules.md` — coding standards, architecture, anti-patterns
- The relevant domain guide from `guides/` (if applicable to the ticket domain)
- `troubleshooting/common-jobs-issues.md` (if the ticket is jobs-related)

### 2. Verify clean state

```bash
cd /workspaces/dse-apis
git status
git branch --show-current
```

If there are uncommitted changes, warn the user and ask how to proceed (stash, commit, or discard).

### 3. Create feature branch

```bash
git checkout main
git pull origin main
git checkout -b feature/d3p-XXXX-short-description
```

Branch naming rules:
- Features: `feature/d3p-XXXX-short-description`
- Bug fixes: `fix/short-description`
- Hotfixes: `hotfix/short-description`

Use the Aha! ticket ID in lowercase. Use hyphens, never underscores.

### 4. Understand the ticket

Read the ticket description carefully. Identify:
- **What is being asked** — the explicit requirements
- **What domain this touches** — which `internal/` packages, which proto files, which workers
- **Cross-repo impact** — does this touch the deployment pipeline? (new terraform inputs → dse-assetstacks + dse-terraform-modules)
- **Testing scope** — API-only? E2E through terraform worker? UI changes?

### 5. Study existing patterns (MANDATORY)

Before writing any code, study the packages you're about to modify:

```bash
# 1. List all files in the target package
ls -la internal/<domain>/

# 2. Read the main files end-to-end — especially client structs and helper functions
cat internal/<domain>/*.go | head -500

# 3. Search for existing helpers that might already do what you need
grep -rn "func " internal/<domain>/ | grep -i "<keyword>"
grep -rn "func build" internal/clients/ # check for existing client builders
```

Answer these questions before proceeding:
- **What structs already exist?** (e.g., `*Client`, `*Server`, `*Runner`) — new methods go on these, not in new files.
- **What helper functions already exist?** (e.g., `buildPapiClient()`, `NewGRPCConn()`) — don't rebuild what exists.
- **What patterns do similar features follow?** Find 2-3 existing implementations of similar functionality and note the pattern.
- **Does the package use interfaces, struct methods, or standalone functions?** Match the existing style.

If you skip this step and jump straight to coding, you WILL create duplicated infrastructure. This is the #1 cause of PR rework (see PR #650 post-mortem in troubleshooting docs).

### 6. Ask clarifying questions

Before writing any code, ask the user about anything unclear. Common questions:
- "Does this new field need to flow to terraform? That requires coordinating dse-assetstacks and dse-terraform-modules."
- "Should this be backward compatible with existing resources in the database?"
- "Is there an existing test job/model/app I should use for E2E testing, or create a new one?"
- "Which environments need testing? (dev, nonprod, both)"

### 7. Plan the implementation (Socratic analysis)

For non-trivial tickets (anything beyond a single-file change), produce a written plan:

| Section | Content |
|---------|---------|
| **Scope** | What is in scope and what is explicitly out |
| **Gap analysis** | Things the ticket doesn't mention but the codebase requires (e.g., svcconfig version bump, constants naming, backward compat for UNSPECIFIED) |
| **Files to change** | Every file to create or modify, with a one-line description |
| **Patterns to follow** | Existing functions/methods that implement similar features — cite the file and function name. New code must match these patterns. |
| **Implementation order** | Sequence with dependencies (e.g., proto first → generate → Go logic → tests → deploy values) |
| **Cross-repo changes** | Any changes needed in dse-assetstacks or dse-terraform-modules |
| **Testing plan** | Which tests to write (unit, protovalidate) and which E2E scenarios to verify |
| **Open questions** | Anything needing user input before starting |

**Trivial change escape hatch**: Skip the plan for single-file edits, typo fixes, doc-only changes, or config bumps. Just do it.

Present the plan to the user. **Wait for approval before writing code.**

### 8. Begin implementation

Once the plan is approved:
1. Start with proto changes (if any) → `make generate`
2. Go business logic
3. Constants and svcconfig
4. Unit tests and protovalidate tests
5. Deploy values (if version bumps needed)
6. Service tester updates (if API surface changed)

Follow the implementation order from the plan. Mark progress using the todo list.

---

## Guardrails

- Never commit to `main` directly — always use a feature branch
- Never skip the planning step for multi-file changes
- Never start coding before understanding cross-repo impact
- **Never skip Step 5 (Study existing patterns)** — this is the #1 cause of PR rework. If the plan's "Patterns to follow" row is empty, the plan is incomplete. Reject it and go back to Step 5.
- Always pull latest `main` before branching
- If the ticket is ambiguous, ask — don't guess

---

## Next step

When implementation is complete:
1. Run the [Implementation Checklist](implementation-checklist.md) to verify everything
2. The checklist includes an [Adversarial Audit](adversarial-audit.md) loop — it must pass with zero issues before creating a PR
