# Agent Bootstrap Prompt — Complete AI Coding Companion System

> **What is this?** A single, self-contained prompt you paste into a Copilot agent chat (or any LLM-powered coding assistant). The agent will interview you about your project, then create a complete meta-repo with coding rules, workflows, agents, prompts, and guides — fully customized to your stack. After setup, the agent operates autonomously across your entire development lifecycle.

> **Prerequisite**: Execute this prompt from inside a dev container, codespace, or workspace where **all your project repos are cloned as sibling folders** under a common root (e.g., `/workspaces/my-app`, `/workspaces/my-infra`). The agent will create a new meta-repo alongside them.

> **How to use after setup**: At the start of every coding session, tell the agent: "Read `rules.md` from `<path-to-meta-repo>`". That's it — auto-detection handles the rest.

---

## START OF PROMPT — Copy everything below this line

---

You are going to set up a complete AI coding companion system for my project. This system lives in its own git repo (a "meta-repo") that you will create. It contains coding rules, reusable workflows, agent definitions, prompt templates, guides, and troubleshooting docs. You will reference this meta-repo throughout all our future work together.

**Important**: Do NOT use placeholders like `<PROJECT_ROOT>` or `<LINT_COMMAND>` in any file you create. Every file must contain the REAL paths, commands, tools, and conventions from my project based on my answers to your interview questions.

---

## PHASE 1: Interview

Before creating anything, interview me in 3 batches. Wait for my answers before proceeding to the next batch.

### Batch 1 — Project Basics

1. What is the name of your project? (This becomes the meta-repo name, e.g., `my-project-rules`)
2. What is the primary programming language and framework? (e.g., Go + gRPC, TypeScript + Next.js, Python + FastAPI)
3. What is the repo structure? Monorepo or multi-repo? List the repos and their purpose.
4. What is the absolute path to the main project repo in this workspace? (e.g., `/workspaces/my-app`)
5. What is your git branching strategy? (e.g., `feature/<ticket-id>-description` off main)
6. What is your ticket/issue tracker? (e.g., Jira, Aha!, Linear, GitHub Issues) What is the ticket ID format? (e.g., `PROJ-1234`)
7. What is the URL pattern for tickets? (e.g., `https://mycompany.atlassian.net/browse/PROJ-1234`)

### Batch 2 — Architecture & Standards

8. What is your deployment target? (Kubernetes, AWS Lambda, Vercel, etc.)
9. What CI/CD system do you use? (GitHub Actions, GitLab CI, Jenkins, etc.)
10. What are your linting/formatting tools and commands? (e.g., `make lint`, `npm run lint`, `ruff check .`)
11. What is the full build command? (e.g., `go build ./...`, `npm run build`, `cargo build`)
12. What is the full CI command (the one that runs everything)? (e.g., `make ci`, `npm test && npm run lint`)
13. What is your testing strategy? List:
    - Unit test command (e.g., `make unit-tests`)
    - Integration test command (if any)
    - E2E test command (if any)
    - Test conventions (build tags, file naming, test framework)
14. What is your logging convention? (library, structured/unstructured, import path)
15. What is your error handling convention? (wrapping patterns, specific helper functions)
16. Do you use code generation? (protobuf, OpenAPI, GraphQL codegen, etc.) What command regenerates? (e.g., `make generate`)

### Batch 3 — Team & Workflow

17. What is your PR review process? (required approvers, CI gates, testing evidence requirements)
18. Do you have a PR template? If yes, paste it here. If not, I'll create one.
19. List your team's reviewers and their focus areas (optional but helpful). Example:
    - `@alice` — focuses on testing evidence
    - `@bob` — focuses on architecture patterns
20. What are common anti-patterns your team fights against? (things that keep coming up in code reviews)
21. Are there cross-repo coordination requirements? (e.g., changing an API schema requires updating 3 repos). If yes, describe the flow.
22. Do you have custom CLI tools in your dev container? List them with their purpose.
23. What are the environment names and their purpose? (e.g., dev, staging, prod)
24. Any other conventions, tools, or patterns I should know about? (auth, secret management, deployment commands, etc.)

---

## PHASE 2: Explore the codebase

After the interview, **before creating any files**, explore the project codebase to gather real data:

```bash
# Get project structure
cd <PROJECT_ROOT>
find . -maxdepth 3 -type f -name "*.go" -o -name "*.ts" -o -name "*.py" -o -name "*.java" -o -name "*.rs" | head -50
ls -la
cat Makefile 2>/dev/null || cat package.json 2>/dev/null || cat Cargo.toml 2>/dev/null || echo "No build file found"

# Check for existing CI/CD
ls .github/workflows/ 2>/dev/null
cat .github/pull_request_template.md 2>/dev/null

# Check for existing linting config
ls .golangci* .eslintrc* .prettierrc* pyproject.toml ruff.toml 2>/dev/null

# Check git info
git remote -v
git log --oneline -5

# Check for existing agents/prompts
ls .github/agents/ .github/prompts/ 2>/dev/null
```

Use this exploration to fill in any gaps from the interview and ensure all commands/paths are accurate.

---

## PHASE 3: Create the meta-repo

Create the meta-repo at `<WORKSPACE_ROOT>/<project-name>-rules/` with this structure:

```
<project-name>-rules/
├── rules.md                              # Core coding rules (SINGLE ENTRY POINT)
├── README.md                             # Repo overview and usage guide
├── .gitignore                            # Standard ignores
├── workflows/
│   ├── start-ticket.md                   # Pick up ticket → branch → plan
│   ├── implementation-checklist.md       # Definition of Done before PR
│   ├── adversarial-audit.md              # Fix-and-recheck verification loop
│   ├── create-pr.md                      # Commit → push → create PR
│   └── review-pr.md                      # Review teammate PRs
├── agents/
│   ├── documentation-doctor.md           # Documentation-only agent
│   └── plan-writer.md                    # Planning agent
├── prompts/
│   ├── create-pr-summary.prompt.md       # Generate PR summary from diff
│   └── update-docs.prompt.md             # Update docs after code changes
├── templates/
│   ├── plan.md                           # Plan template for plan-writer agent
│   └── testing-notes.md                  # Testing evidence template for PRs
├── guides/                               # Domain-specific playbooks (populated over time)
│   └── .gitkeep
├── troubleshooting/                      # Known issues & solutions (populated over time)
│   └── .gitkeep
└── testing-files/                        # PR testing evidence (populated over time)
    └── .gitkeep
```

---

## PHASE 4: Generate all files

Create every file with full, real content. No placeholders. Below are the detailed specifications for each file.

---

### FILE: `rules.md`

This is the **single entry point** the agent reads at the start of every session. It must contain ALL of the following sections, in this order, fully populated with real data from the interview and codebase exploration.

#### Section 1: Header

```markdown
# <Project Name> — Coding Rules & Best Practices

You are an expert in <LANGUAGES, FRAMEWORKS, TOOLS> operating within the <PROJECT> codebase. Your role is to ensure code is idiomatic, modular, testable, and aligned with the patterns already established in this project.
```

#### Section 2: Workflow Auto-Detection

```markdown
## Workflow Auto-Detection

After reading this file, you have access to the workflows listed below. **Do not wait for the user to name a workflow explicitly** — detect which one applies from context and run it automatically:

| User says something like... | Run this workflow |
|----------------------------|-------------------|
| "I have ticket PROJ-123", "let's start this feature", "pick up this ticket", "empecemos con este feature" | [Start Ticket](workflows/start-ticket.md) |
| "I'm done implementing", "is this ready?", "check my work", "ya terminé" | [Implementation Checklist](workflows/implementation-checklist.md) |
| "audit this", "verify everything", "run the audit", "revisa que todo esté bien" | [Adversarial Audit](workflows/adversarial-audit.md) |
| "create the PR", "push and open PR", "let's submit this", "crea el PR" | [Create PR](workflows/create-pr.md) |
| "review this PR", "review PR #42", "check this pull request", "revisa este PR" | [Review PR](workflows/review-pr.md) |

When in doubt, ask: **"Should I run the [X] workflow?"**
```

#### Section 3: Tooltip Reminders

```markdown
## Tooltip Reminders

At the **end of every response**, append a single-line reminder. Format:

> 💡 *Tip: [brief, actionable reminder]*

Rules:
- Always the **last line** of the response — only ONE per response, never two
- Max 1 sentence
- Rotate tips — don't repeat the same one twice in a row
- Only suggest workflows/capabilities relevant to the current context
```

#### Section 4: Related Documentation

A table linking to ALL files in the meta-repo. Update this table whenever a file is added.

```markdown
## Related Documentation

### Workflows
| Workflow | Description |
|----------|-------------|
| [Start Ticket](workflows/start-ticket.md) | Pick up a ticket → create branch → plan implementation |
| [Implementation Checklist](workflows/implementation-checklist.md) | Definition of Done — all verification steps before creating a PR |
| [Adversarial Audit](workflows/adversarial-audit.md) | Fix-and-recheck loop: prove the code works by trying to break it |
| [Create PR](workflows/create-pr.md) | Commit, push, and create PR with required format and testing evidence |
| [Review PR](workflows/review-pr.md) | Review a teammate's PR against coding standards |

### Agents
| Agent | Description |
|-------|-------------|
| [Documentation Doctor](agents/documentation-doctor.md) | Creates and updates documentation only — never touches application code |
| [Plan Writer](agents/plan-writer.md) | Creates detailed implementation plans for future agent sessions |

### Prompts
| Prompt | Description |
|--------|-------------|
| [Create PR Summary](prompts/create-pr-summary.prompt.md) | Generates PR description from the current diff |
| [Update Docs](prompts/update-docs.prompt.md) | Updates documentation to reflect code changes |

### Templates
| Template | Description |
|----------|-------------|
| [Plan Template](templates/plan.md) | Structured plan format for the plan-writer agent |
| [Testing Notes](templates/testing-notes.md) | Testing evidence template for PRs |

### Guides
(Populated as you add domain-specific playbooks)

### Troubleshooting
(Populated as you document known issues and solutions)
```

#### Section 5: Technology Stack

A table listing every technology, tool, and version used. Based on the interview answers and codebase exploration. Example format:

```markdown
## Technology Stack

| Layer | Technology |
|-------|-----------|
| Language | <language and version from go.mod/package.json/etc.> |
| Framework | <framework> |
| Database | <database> |
| Cloud | <cloud provider> |
| CI/CD | <CI system> |
| Linting | <linters and config> |
| Testing | <test framework, assertion library> |
| Deployment | <deployment tools> |
| ... | ... |
```

#### Section 6: Project Structure

A directory tree with annotations explaining what each folder/file is for. Based on actual `ls`/`find` output from the codebase. Use the same format as a `tree` command output with inline comments.

#### Section 7: Architecture Patterns

Document the real patterns from the codebase:
- Service/application lifecycle (how services start, how workers run)
- Dependency injection patterns
- Database/storage layer patterns
- Message queue / event patterns (if applicable)
- Inter-service communication patterns
- Configuration management

Include **real code snippets** from the project as examples (sanitized of secrets).

#### Section 8: Coding Standards

Document the project's actual conventions for:
- **Logging**: Which library, import path, usage rules, common mistakes
- **Error handling**: Wrapping patterns, when to use which approach, examples
- **Configuration**: How config is loaded, naming conventions for env vars
- **Naming conventions**: Resource names, constants, variables, files
- **Validation**: How input validation works (annotations, manual, middleware)
- **Audit/metadata**: How records are stamped with created/updated metadata

Include **real code examples** from the project.

#### Section 9: Testing

Document the testing strategy with:
- Testing tiers table (unit, integration, e2e — commands, file patterns, build tags)
- Test conventions (naming, patterns, framework, assertion library)
- Mock patterns (how mocks are created — struct-based, generated, etc.)
- Real code examples of the preferred test pattern

#### Section 10: Development Workflow

- Branch naming convention with examples
- Key make/build targets table (command → purpose)
- CI pipeline description (what runs, in what order)
- PR requirements (what must be included for approval)
- Manual testing notes (when needed, where to put them)
- Code review response patterns (how to respond to different reviewer concerns)
- Team reviewer tendencies table (if provided)

#### Section 11: Linting & Formatting

- Tools and configuration
- Key linters and what they enforce
- Custom enforcement scripts (if any)

#### Section 12: Security & Resilience

- Auth mechanism
- Authorization model
- Error recovery patterns
- Retry strategies

#### Section 13: Deployment

- Infrastructure overview
- Environment promotion pipeline
- Environment-specific configuration

#### Section 14: Anti-Patterns

A numbered list of at least 15 things NOT to do. Combine project-specific anti-patterns (from the interview) with universal best practices. Format:

```markdown
## Anti-Patterns — What NOT to Do

1. **Don't** <specific bad practice>. <Why it's bad and what to do instead.>
2. **Don't** <specific bad practice>. <Why it's bad and what to do instead.>
...
```

Universal anti-patterns to always include (adapted to the project's language):
- Don't hand-edit auto-generated files
- Don't write tests that connect to real external services in unit tests — mock all external interactions
- Don't add features, refactors, or "improvements" that weren't requested
- Don't hardcode secrets, credentials, or environment-specific values
- Don't use deprecated libraries/patterns when a newer standard exists in the project
- Don't create broad God-interfaces/classes — keep them focused and single-purpose
- Don't skip error handling for "impossible" cases at system boundaries
- Don't add compatibility workarounds or deprecated fallback fields that weren't requested
- Don't rename shared identifiers for "readability" if they flow across repos/systems — document the constraint instead

#### Section 15: Cross-Repo Coordination (if applicable)

If the user described cross-repo dependencies:
- How changes flow across repos (table: repo → what to change → example)
- Version synchronization rules
- Field naming chains across systems
- Backward compatibility patterns

#### Section 16: Key Conventions Summary

A numbered list of the top 10 conventions that define the project's coding culture.

#### Section 17: Git Push — Multi-Repo Workspace (if applicable)

If the workspace has multiple repos, document:
- Which repos can be pushed with the default token
- Which repos need a PAT (and the procedure to get one)
- Never store PATs in files — terminal commands only

---

### FILE: `workflows/start-ticket.md`

Full content — adapt every command to the real project:

```markdown
# Start Ticket

Pick up a ticket, create the feature branch, understand the scope, and plan before writing any code.

---

## Input

The user provides:
- **Ticket ID** (e.g., `<TICKET_PREFIX>-123`) and a description or link.
- Optionally, a paste of the ticket body/acceptance criteria.

If the user doesn't provide a ticket ID, ask: **"What's the ticket ID? (e.g., <TICKET_PREFIX>-123)"**

---

## Steps

### 1. Load project context

Read these files (in parallel if possible):
- `rules.md` — coding standards, architecture, anti-patterns
- Any relevant domain guide from `guides/` (if applicable to the ticket domain)
- Any relevant troubleshooting doc from `troubleshooting/` (if applicable)

### 2. Verify clean state

```bash
cd <REAL_PROJECT_ROOT>
git status
git branch --show-current
```

If there are uncommitted changes, warn the user and ask how to proceed (stash, commit, or discard).

### 3. Create feature branch

```bash
git checkout main
git pull origin main
git checkout -b <REAL_BRANCH_CONVENTION>
```

Branch naming rules:
- Features: `<feature-prefix>/<ticket-id>-short-description`
- Bug fixes: `<fix-prefix>/short-description`
- Hotfixes: `<hotfix-prefix>/short-description`

### 4. Understand the ticket

Read the ticket description carefully. Identify:
- **What is being asked** — the explicit requirements
- **What domain this touches** — which packages, modules, files
- **Cross-repo impact** — does this require changes in other repos?
- **Testing scope** — API-only? E2E through workers? UI changes?

### 5. Ask clarifying questions

Before writing any code, ask the user about anything unclear. Common questions:
- "Does this change affect other repos or downstream consumers?"
- "Should this be backward compatible with existing data?"
- "Is there an existing test resource I should use, or create a new one?"
- "Which environments need testing?"

### 6. Plan the implementation (Socratic analysis)

For non-trivial tickets (anything beyond a single-file change), produce a written plan:

| Section | Content |
|---------|---------|
| **Scope** | What is in scope and what is explicitly out |
| **Gap analysis** | Things the ticket doesn't mention but the codebase requires |
| **Files to change** | Every file to create or modify, with a one-line description |
| **Implementation order** | Sequence with dependencies |
| **Cross-repo changes** | Any changes needed in other repos |
| **Testing plan** | Which tests to write and which scenarios to verify |
| **Open questions** | Anything needing user input before starting |

**Trivial change escape hatch**: Skip the plan for single-file edits, typo fixes, doc-only changes, or config bumps. Just do it.

Present the plan to the user. **Wait for approval before writing code.**

### 7. Begin implementation

Once the plan is approved:
1. Start with schema/contract changes (if any) → run code generation
2. Business logic
3. Configuration and constants
4. Unit tests
5. Deploy/infrastructure values (if needed)
6. E2E test updates (if API surface changed)

Follow the implementation order from the plan. Mark progress using the todo list.

---

## Guardrails

- Never commit to main directly — always use a feature branch
- Never skip the planning step for multi-file changes
- Never start coding before understanding cross-repo impact
- Always pull latest main before branching
- If the ticket is ambiguous, ask — don't guess

---

## Next step

When implementation is complete:
1. Run the [Implementation Checklist](implementation-checklist.md) to verify everything
2. The checklist includes an [Adversarial Audit](adversarial-audit.md) loop — it must pass with zero issues before creating a PR
```

---

### FILE: `workflows/implementation-checklist.md`

```markdown
# Implementation Checklist (Definition of Done)

Run this checklist before creating a PR. Every item must pass. If something fails, fix it before proceeding.

---

## Input

- **ticket-id** (OPTIONAL): Auto-detect from branch name if not provided.

```bash
git branch --show-current
# Extract ticket ID from branch name
```

---

## Checklist

All commands assume you are in the project root:

```bash
cd <REAL_PROJECT_ROOT>
```

### Code Quality

```bash
<REAL_LINT_COMMAND>
```
- [ ] Linting passes with zero warnings

```bash
<REAL_CI_COMMAND>
```
- [ ] Full CI suite passes

```bash
<REAL_BUILD_COMMAND>
```
- [ ] Project builds/compiles cleanly

<IF_CUSTOM_ENFORCEMENT_SCRIPTS_EXIST>
```bash
<REAL_SCRIPT_COMMAND>
```
- [ ] Custom enforcement checks pass
</IF_CUSTOM_ENFORCEMENT_SCRIPTS>

### Architecture & Code

- [ ] Implementation matches the approved plan scope (no scope creep)
- [ ] Auto-generated files NOT hand-edited
- [ ] Logging uses the project's standard library/pattern
- [ ] Error handling follows the project's conventions
- [ ] New enum/status fields default to a safe value for existing records
- [ ] No hardcoded secrets, credentials, or environment-specific values in code

### Cross-Repo Coordination (if applicable)

- [ ] Version references are consistent across all repos
- [ ] Shared contracts (schemas, interfaces, configs) are in sync
- [ ] Field names match across all layers in the pipeline

### Testing

- [ ] Unit tests written for new business logic
- [ ] Tests follow project conventions (naming, patterns, tools)
- [ ] Tests pass: `<REAL_TEST_COMMAND>`

### E2E / Service Tester (mandatory if available)

```bash
<REAL_E2E_COMMAND>
```
- [ ] E2E tests pass against a deployed environment
- [ ] E2E test logs saved for PR evidence

### E2E Testing (for pipeline/integration features)

If the ticket modifies a deployment pipeline, message queue, or cross-service flow:

- [ ] Deployed to a test environment
- [ ] API-level tests: Create/Get resources with new fields, verify validation
- [ ] Downstream service receives correct inputs (check logs)
- [ ] Test evidence documented in `testing-files/`

### Documentation

- [ ] README or contributing docs updated if new setup steps added
- [ ] Relevant troubleshooting docs updated if new known issues found

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

→ Proceed to [Create PR](create-pr.md)
```

---

### FILE: `workflows/adversarial-audit.md`

```markdown
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
cd <REAL_PROJECT_ROOT>
git diff --name-only origin/main...HEAD
```

Read each changed file completely. Understand what was added, modified, and removed.

### 2. Internal consistency

- Do changed files reference each other correctly? (imports, types, constants, names, versions)
- Do paths, names, and versions match across all layers (schema → code → config → deploy)?
- Are generated files fresh? (run code generation command then `git diff`)
- Any typos in field names, log messages, or error strings?

### 3. Build & lint verification

Execute each command and verify clean output:

```bash
<REAL_LINT_COMMAND>
<REAL_CI_COMMAND>
<REAL_BUILD_COMMAND>
<REAL_CUSTOM_SCRIPTS>
```

Every command must exit 0 with no warnings. If any fails, fix it immediately.

### 4. Test verification

```bash
<REAL_UNIT_TEST_COMMAND>
```

For schema/contract changes, verify validation rules work:
- Valid inputs pass
- Invalid inputs are rejected with correct error messages
- Edge cases: empty strings, null/nil, zero values, missing required fields, boundary values

### 5. Adversarial tests

Invent at least 5 tests that try to break the implementation. Think:

- What happens with unexpected/default values in existing database records?
- What if a referenced resource doesn't exist?
- What if the same request is sent twice (idempotency)?
- Do new fields survive a round-trip (create → read)?
- Are new constants/configs actually used, or are they dead code?
- Does the naming consistency chain hold across all layers?
- What if this runs in a different environment than expected?

### 6. Cross-file coherence

For each new field or behavior:
- Is it in the schema/contract definition?
- Is it in the generated code (if auto-generated)? Run generation + `git diff`
- Is it handled in the create/write path?
- Is it handled in the read/list/get path?
- Is it handled in downstream processing (workers, pipelines)?
- Is it tested?
- Is it in the E2E/service tester (if API surface changed)?

---

## Report format

```markdown
## Audit Report

### Summary
[X verified, Y failed, Z warnings]

### Critical (✗)
[What failed, evidence, correction applied]

### Warnings (⚠)
[Works today but fragile — explain the risk]

### Verified (✓)
[Brief list of what passed with evidence]

### Adversarial tests
[What you invented, what you tried, result]
```

---

## The Loop

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
```

---

### FILE: `workflows/create-pr.md`

```markdown
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
- The ticket ID from the branch name
- Whether this is a feature, fix, or hotfix

### 2. Stage and commit

Stage specific files — avoid `git add .` to prevent committing .env, generated cruft, or temp files:

```bash
git add <specific files>
```

Commit message format: concise description of what changed. The team does not enforce a strict commit message regex, but commits should be descriptive. Examples:
- `Add default job type handling for legacy records`
- `Update service version to v0.0.9 across all environments`
- `Fix validation rule to enforce bidirectional constraint`

### 3. Push to remote

```bash
git push origin <branch-name>
# First push: git push -u origin <branch-name>
```

### 4. Generate PR description

Use the project's PR template (from the interview). Fill EVERY section — reviewers will reject PRs with empty sections.

The PR description must include:
- **Summary**: WHY this change is needed (not just what changed). Reference business context.
- **Changes**: Bullet list grouped by area (schema, logic, tests, deploy, etc.)
- **Ticket link**: Link to the ticket in the tracker
- **Dependencies**: Links to other PRs this depends on, or "None"
- **Testing Performed**: All verification commands run with evidence
- **Environment tested**: Which environment(s) were tested
- **Screenshots/logs**: Collapsible sections with actual command output
- **E2E/service-tester results**: Mandatory if available
- **Tech debt**: Any tech debt created, or "None"
- **Reviewer checklist**: Items for the reviewer to verify

### 5. Save the PR description and create the PR

```bash
# Write the filled-in PR description to a temp file
cat > PR.md << 'PREOF'
<filled-in PR description here>
PREOF

# Create the PR
gh pr create \
  --title "<Concise PR title>" \
  --body-file PR.md \
  --base main

# Clean up temp file
rm PR.md
```

If a PR already exists for the branch:
```bash
gh pr edit <PR_NUMBER> --body-file PR.md
```

### 6. Verify

```bash
gh pr view --web  # Open in browser to verify formatting
```

Check that:
- All sections are filled
- Screenshots are visible
- CI checks are passing
- The ticket link works

---

## Iterating on PR Comments

After reviewers leave comments:

1. Read all comments carefully
2. For each comment, determine if it's:
   - **A valid fix** → implement it
   - **A misunderstanding** → explain with evidence (truth tables, cross-repo constraints, etc.)
   - **An enhancement beyond scope** → suggest a follow-up ticket
3. After making changes, run the [Adversarial Audit](adversarial-audit.md) loop:
   - Run audit → fix issues → re-run until zero issues
   - This replaces manual CI checks — the audit covers them and more
4. Push and reply to each comment thread
5. If new tests are needed, add them and update the testing section of the PR

### Common reviewer patterns to anticipate

| Reviewer concern | How to address proactively |
|-----------------|---------------------------|
| "Can you rename this field?" | If cross-repo, explain the naming chain across systems |
| "Can you simplify this logic?" | Show the truth table for edge cases that the "simpler" version misses |
| "Add testing evidence" | Include screenshots + command output + downstream logs in the PR |
| "Add server-side validation" | If referencing another resource, validate it exists before persisting |

---

## Guardrails

- Never create a PR with empty testing sections
- Never push to main directly — always via PR
- Never skip CI checks before pushing
- Always include the ticket link
- Always include E2E/service-tester results if available
```

---

### FILE: `workflows/review-pr.md`

```markdown
# Review PR

Workflow for reviewing a Pull Request from a team member against the project's coding standards and the ticket requirements.

---

## Input

- **pr-number**: The PR number to review (e.g., `42`)
- **repo** (OPTIONAL): Repository in `owner/repo` format. Default: current repo.

---

## Steps

### 1. Gather context

```bash
gh pr view <pr-number> --json title,body,headRefName,baseRefName,files,additions,deletions,author
gh pr diff <pr-number>
```

Extract from the PR:
- The ticket ID (from the body or branch name)
- The stated purpose/summary
- The list of changed files

### 2. Understand the ticket

From the ticket link in the PR body, understand:
- What was requested
- Acceptance criteria (if any)
- Scope boundaries

### 3. Analyze the diff

For each changed file, evaluate against these categories:

#### Correctness
- Does the implementation match what the ticket asked for?
- Are edge cases handled (nil/null checks, empty strings, default values)?
- Are new schema fields properly validated?
- Are new enum/status values handled in all conditional logic with no silent fallthrough?

#### Architecture & Patterns
Reference rules.md for the full list. Key checks:
- Source of truth files (schemas, contracts) not hand-edited in generated form
- Logging uses the standard library/pattern
- Error handling follows conventions
- Naming is consistent across all layers
- Default values set for backward compatibility with existing data

#### Anti-patterns to flag
Check every item against the anti-patterns list in rules.md.

#### Testing
- Are there unit tests for new business logic?
- Do tests follow project conventions (naming, patterns, tools)?
- Is there E2E evidence for integration/pipeline changes?
- Are validation rules tested (valid + invalid inputs)?

#### Cross-repo coordination
If the PR touches shared contracts, versions, or deployment configs:
- Are versions consistent across all repos?
- Are field names consistent across all layers?
- Are downstream repos updated (or PRs planned)?

#### PR quality
- Is the summary clear about WHY, not just WHAT?
- Is the ticket link present and correct?
- Are testing sections filled with actual evidence (not placeholders)?
- Is E2E/service-tester evidence included?

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

```bash
# Approve
gh pr review <pr-number> --approve --body "LGTM. <brief comment>"

# Request changes
gh pr review <pr-number> --request-changes --body-file review.md

# Comment only
gh pr review <pr-number> --comment --body-file review.md
```

For inline comments on specific lines:
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
| **Must fix** | Bugs, security issues, broken logic, missing validation | Null pointer dereference, SQL injection, missing auth check |
| **Should fix** | Code quality, maintainability, patterns not followed | Using deprecated logging library, missing error wrapping |
| **Nitpick** | Style, naming preferences, minor cleanup | Variable could have a clearer name |
| **Positive** | Good code worth calling out | Clean table-driven tests, thorough edge case handling |

---

## Guardrails

- Always read the full diff before commenting — avoid context-free line comments
- Check testing evidence exists and is real (not placeholders)
- If you see cross-repo impacts, verify version consistency
- Don't block on style-only issues — use nitpick severity
- If unsure about a pattern, reference the specific section of rules.md
```

---

### FILE: `agents/documentation-doctor.md`

```markdown
---
description: 'Agent for creating and updating documentation within this repo'
tools: ['edit', 'search', 'runCommands', 'runTasks', 'usages', 'changes', 'fetch', 'todos']
---

This agent is specifically designed to assist with creating and updating documentation. The only changes it should make are to docstrings, doc files, and markdown files within the repo. It does NOT touch application code logic.

Guidelines:
1. **Documentation Focus**: Concentrate on the application code and schema definitions. Ignore:
   - Auto-generated code directories
   - Build artifacts
   - Third-party dependencies
2. **Documentation Standards**:
   - Follow the language's documentation best practices (GoDoc, JSDoc, Python docstrings, etc.)
   - Be clear and verbose enough to remove doubts, but concise enough to be consumable
   - Keep pages under 5000 characters and docstrings under 50 words unless prompted otherwise
   - Use diagrams to explain complex ideas in READMEs (Mermaid preferred)
3. **Branching**: Always create or update documentation on a feature branch tied to a ticket. Usually part of an existing PR.
4. **Contextual Updates**: Base documentation updates on changes to the source code. If there are no clear changes to reference, ask for clarification.
```

---

### FILE: `agents/plan-writer.md`

```markdown
---
description: 'Agent for creating detailed implementation plans for future agent sessions'
tools: ['edit', 'search', 'runCommands', 'runTasks', 'usages', 'changes', 'fetch', 'todos']
---

This agent creates detailed implementation plans that other agents (or future sessions) can follow autonomously.

Guidelines:
1. **Plan Focus**: Create and update plan documents in markdown format. Save plans in the meta-repo.
2. **Clarity and Detail**: Each plan must be detailed enough to guide a follow-up agent without additional context. Specify which agent the plan should be assigned to.
3. **Naming**: Plans should be named based on the ticket ID (e.g., `proj-12345.md`), or for general work, use descriptive names starting with `feat-`, `fix-`, `docs-`, or `hotfix-`.
4. **Template**: All plans should follow the plan template in `templates/plan.md`.
```

---

### FILE: `prompts/create-pr-summary.prompt.md`

```markdown
# Create PR Summary

Generate a concise summary of the changes made in the current branch compared to main. Be as concise as possible while covering all changes.

## Steps

1. First, check if the project has a PR template (e.g., `.github/pull_request_template.md`). If it exists, use it as the structure.
2. Compare this branch to main using git CLI:
   ```bash
   git diff --stat origin/main...HEAD
   git diff origin/main...HEAD
   ```
3. Generate a summary as a `PR.md` file containing:
   - A brief overview of the purpose of the PR (the WHY, not just the WHAT)
   - Key changes made to the codebase, grouped by area
   - Any new tests added or existing tests modified
   - Documentation updates reflecting the changes
   - All sections from the PR template filled out appropriately
4. Ensure all sections are present and filled — reviewers will reject PRs with empty sections.
```

---

### FILE: `prompts/update-docs.prompt.md`

```markdown
---
agent: documentation-doctor
---

Review the latest changes in this repo compared to main and update the relevant documentation files (README, docstrings, inline comments, etc.) to reflect the code changes.

Steps:
1. Review existing documentation for accuracy and completeness
2. Update anything that has become out of date
3. Add new documentation for features or changes that are not yet documented
4. Do NOT modify application code logic — documentation only
```

---

### FILE: `templates/plan.md`

```markdown
---
title: 'PLAN TITLE HERE'
branch: feat/<ticket-id>-feature-name
description: 'A brief description of what this plan covers'
assigned_agent: 'agent-filename-here.md'
---

## Objective

A clear and concise statement of the problem this feature is intended to solve.

## Technical Context

- Overview of the current system architecture relevant to this feature.
- Any existing components or services that will be impacted.

## Requirements

- A list of specific requirements the agent must fulfill.
- Each requirement should be clear and verifiable.

## Tasks

### Phase 1: Initial Setup

Purpose: Project initialization and basic structure

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize project with dependencies
- [ ] T003 Configure linting and formatting tools

### Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core changes that MUST be complete before ANY other phase

**⚠️ CRITICAL**: No work on later phases can begin until this phase is complete

- [ ] T004 Schema/contract changes (if applicable)
- [ ] T005 Run code generation
- [ ] T006 Core business logic implementation
- [ ] T007 Configuration and environment setup

### Phase 3: Implementation

**Purpose**: Feature-specific work

- [ ] T008 Implement the main feature logic
- [ ] T009 Add unit tests
- [ ] T010 Add integration/E2E tests (if applicable)

### Phase 4: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple phases

- [ ] T011 Documentation updates
- [ ] T012 Code cleanup
- [ ] T013 Additional tests (if needed)
- [ ] T014 Security review

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — can start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 — BLOCKS all subsequent phases
- **Phase 3 (Implementation)**: Depends on Phase 2
- **Phase 4 (Polish)**: Depends on Phase 3

### Cross-Repo Dependencies

- List any changes needed in other repos
- Note which repo changes must be merged first
```

---

### FILE: `templates/testing-notes.md`

```markdown
# Testing Notes: <Ticket ID> — <Feature Name>

## Environment

- **Environment tested**: dev / staging / prod
- **Branch**: `<branch-name>`
- **Deployed at**: <timestamp or commit hash>

## Prerequisites

- [ ] Authenticated (auth login / SSO / etc.)
- [ ] Code deployed to target environment
- [ ] Required access/permissions confirmed

## Test Scenarios

### Scenario 1: <Description>

**Purpose**: <What this test verifies>

**Request**:
```bash
<actual command or curl request>
```

**Expected Result**: <What should happen>

**Actual Result**:
```json
<actual response>
```

**Screenshot**: <if applicable>

**Result**: ✅ PASS / ❌ FAIL

---

### Scenario 2: <Description>

(repeat format)

---

## E2E Pipeline Verification (if applicable)

### Downstream Logs

<details>
<summary>Worker/service logs showing the new data was received</summary>

```
<paste logs here>
```

</details>

## Summary

| Scenario | Result |
|----------|--------|
| Scenario 1 | ✅ PASS |
| Scenario 2 | ✅ PASS |

**Overall**: All scenarios passed. Ready for PR.
```

---

### FILE: `README.md`

```markdown
# <project-name>-rules

AI coding companion meta-repo. Contains coding rules, reusable workflows, agent definitions, prompt templates, and domain knowledge for working on the <project-name> codebase.

## How to use

1. At the start of a coding session, tell your AI agent: **"Read `rules.md` from `<path-to-this-repo>`"**
2. The agent auto-detects which workflow to run based on your conversation — you never need to name workflows explicitly
3. Just describe what you want to do naturally

## Workflow sequence

```
Start Ticket → Implement → Implementation Checklist → Create PR → Review PR
                                    ↓
                           Adversarial Audit Loop
                          (fix → re-check → repeat
                           until zero issues)
```

## Structure

| Directory | Purpose |
|-----------|---------|
| `rules.md` | Core entry point — coding standards, architecture, anti-patterns |
| `workflows/` | Step-by-step workflows for the development lifecycle |
| `agents/` | Specialized agent definitions (documentation, planning) |
| `prompts/` | Reusable prompt templates (PR summaries, doc updates) |
| `templates/` | Structured templates (plans, testing notes) |
| `guides/` | Domain-specific playbooks (add as needed) |
| `troubleshooting/` | Known issues & solutions (add as needed) |
| `testing-files/` | PR testing evidence (add per ticket) |

## Adding content

- **New guide**: Create a `.md` file in `guides/`, then add it to the Related Documentation table in `rules.md`
- **New troubleshooting doc**: Create a `.md` file in `troubleshooting/`, then add to the table
- **Testing evidence**: Create a `.md` file in `testing-files/` per ticket
- **New workflow**: Create in `workflows/`, add a row to the auto-detection table in `rules.md`

## Evolving the rules

Edit `rules.md` directly as your project grows. Add new anti-patterns when you discover them, update conventions when the team agrees on changes, and remove outdated guidance.
```

---

### FILE: `.gitignore`

```
.DS_Store
*.swp
.vscode/
PR.md
.env
.env.*
node_modules/
__pycache__/
*.pyc
.idea/
*.class
target/
dist/
build/
```

---

## PHASE 5: Initialize git and verify

After creating all files:

```bash
cd <META_REPO_PATH>
git init
git add -A
git commit -m "Initial setup: rules, workflows, agents, prompts, and templates"
```

Then run link validation to ensure all internal markdown links resolve:

```bash
broken=0
while IFS= read -r line; do
  src=$(echo "$line" | cut -d: -f1)
  dir=$(dirname "$src")
  target=$(echo "$line" | grep -oP '\]\(\K[^)]+' | head -1)
  clean="${target%%#*}"
  resolved="$dir/$clean"
  if [ ! -f "$resolved" ]; then
    echo "✗ $src -> $target"
    broken=$((broken+1))
  fi
done < <(grep -rnP '\[[^\]]+\]\((?!https?://|#|mailto:)[a-zA-Z0-9_./-]+\.md' --include="*.md" . | grep -v '\.git/')
echo "Broken links: $broken"
```

Fix any broken links before proceeding.

---

## PHASE 6: Report

After everything is created, give me:
1. The full directory tree of the meta-repo
2. Line count of `rules.md`
3. Line count of each workflow file
4. Confirmation that all internal links resolve (zero broken)
5. A summary of what each file contains
6. Instructions for pushing to a remote (ask me for the remote URL and whether I need a PAT)

---

## PHASE 7: Daily usage guide

Explain to me:
- **Starting a session**: Just say "read `rules.md` from `<path>`" — the agent loads everything
- **Auto-detection**: Describe what you want naturally, the agent picks the right workflow from context
- **Tooltips**: You'll see a one-liner at the end of every response reminding you of relevant capabilities
- **Adding knowledge over time**: Create new files in `guides/`, `troubleshooting/`, or `testing-files/`, then update the Related Documentation table in `rules.md`
- **Evolving rules**: Edit `rules.md` as conventions change — it's a living document

---

## DESIGN PRINCIPLES

1. **Single entry point**: The agent only needs `rules.md` — everything else is linked from there
2. **Auto-detection over explicit invocation**: The agent infers workflows from conversation context
3. **Fix-and-recheck loops**: The adversarial audit doesn't just report — it fixes and re-runs until clean
4. **Tooltips for discoverability**: The agent reminds you of capabilities you might forget
5. **Living documentation**: The meta-repo evolves with your project — add guides, update anti-patterns, record solutions as you discover them
6. **No placeholders**: Every file contains real commands, real paths, real tools from YOUR project
7. **Agents and prompts**: Specialized agents (docs, planning) and reusable prompts (PR summary, doc update) are included and ready to use

---

## FAQ

**Q: Can I use this for multiple projects?**
A: Create one meta-repo per project. The rules and workflows are specific to each stack.

**Q: What if my project doesn't use `gh` CLI?**
A: The agent will adapt the PR creation and review steps to your preferred tool during setup.

**Q: What if I work alone (no PR reviews)?**
A: Skip the review workflow. The adversarial audit still runs as your automated reviewer.

**Q: How do I add a new workflow?**
A: Create a new `.md` file in `workflows/`, add a row to the auto-detection table in `rules.md`, and add it to the Related Documentation table.

**Q: What if the agent doesn't auto-detect the right workflow?**
A: Just tell it explicitly: "run the start-ticket workflow." Auto-detection is a convenience, not a requirement.

**Q: What if I want the agent to speak Spanish (or another language)?**
A: Tell the agent your preferred language during the interview. It will adapt all tooltips, questions, and workflow prompts accordingly.

**Q: Can I use this with non-Copilot agents (ChatGPT, Claude, Cursor, etc.)?**
A: Yes. The meta-repo is just markdown files. Any agent that can read files can use this system. The workflow auto-detection table and tooltip instructions work universally.

---

## END OF PROMPT
