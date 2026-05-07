# dse-rules

Personal knowledge base for working on the [dse-apis](https://github.com/bayer-int/dse-apis) Go monorepo (Bayer Decision Science Ecosystem).

## Structure

| Directory | Purpose |
|-----------|---------|
| [rules.md](rules.md) | Core coding rules, architecture patterns, anti-patterns, and team conventions |
| [workflows/](workflows/) | Step-by-step agent workflows: start ticket → implement → create PR → review PR |
| [guides/](guides/) | Domain playbooks (Jobs, Apps, Models) and CREST authorization guide |
| [troubleshooting/](troubleshooting/) | Known issues and debugging guides |
| [testing-files/](testing-files/) | PR-specific testing evidence and notes |

## Workflows

These are designed to be followed sequentially for a typical ticket:

1. **[Start Ticket](workflows/start-ticket.md)** — Pick up an Aha! ticket, create branch, plan implementation
2. **[Implementation Checklist](workflows/implementation-checklist.md)** — Definition of Done before creating a PR
3. **[Adversarial Audit](workflows/adversarial-audit.md)** — Fix-and-recheck loop until zero issues (called from checklist + PR iterations)
4. **[Create PR](workflows/create-pr.md)** — Commit, push, create PR with required format
5. **[Review PR](workflows/review-pr.md)** — Review a teammate's PR against coding standards

## Usage

Point your AI agent's custom instructions to `rules.md` as the primary context file. The workflows are designed to be invoked by name (e.g., "run the start-ticket workflow for D3P-2068").
