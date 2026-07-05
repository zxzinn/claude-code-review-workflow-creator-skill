---
name: setup-claude-review
description: Set up Claude Code GitHub Actions (PR auto-review + interactive @claude) for a project, including a project-specific REVIEW.md. Use when the user asks to "set up claude code review", "add claude github action", "初始化 claude review", or wants automated PR review with anthropics/claude-code-action in a repo that doesn't have it yet.
---

# Setup Claude Review

Installs two GitHub Actions workflows (`claude-code-review.yml` for automated PR review, `claude.yml` for interactive `@claude` mentions) into the current project, and drafts a project-specific `REVIEW.md` that the review workflow reads for its priorities.

The workflow YAML is mechanical and copied as-is. The value of this skill is in `REVIEW.md`: a generic review checklist is nearly worthless, so this skill scans the actual project first and drafts focus areas grounded in what it finds, for the user to review and edit, not a template to fill in blind.

## Steps

0. **Confirm setup decisions with the user before touching any files.** Ask (a single question is fine if bundling is natural):
   - Which model id to put in `claude-code-review.yml`'s `--model` flag. Do not silently pick a default.
   - Whether `CLAUDE_CODE_OAUTH_TOKEN` is already configured for this repo/org. If not, tell the user now that the workflows will not run until they set it up (via `/install-github-app` or manually under Settings -> Secrets and variables -> Actions), so they aren't surprised by a failing run later.
   - If this is a private repo under an org, confirm org-level Actions permissions allow `anthropics/claude-code-action` (or all actions) to run.

1. **Check for existing setup.** If `.github/workflows/claude-code-review.yml` or `claude.yml` already exist, show the diff against the templates here and ask before overwriting: don't clobber project-specific customizations silently.

2. **Copy the workflow templates.**
   - Copy `templates/claude-code-review.yml` -> `.github/workflows/claude-code-review.yml`
   - Copy `templates/claude.yml` -> `.github/workflows/claude.yml`
   - Replace `{{CLAUDE_MODEL}}` in `claude-code-review.yml` with the model id confirmed in step 0.

3. **Scan the project** to ground the REVIEW.md draft. At minimum:
   - Identify the stack: read `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` etc. to determine language, framework, and key libraries.
   - Read the existing `CLAUDE.md` (repo root and `.claude/CLAUDE.md`) if present. It likely already documents conventions, and REVIEW.md should not duplicate what's already enforced there (linters, formatters). Cross-reference instead of repeating.
   - Sample a handful of representative source files (models/schemas, a service or handler layer, a test file) to spot real patterns worth enforcing: architectural layering conventions, error handling style, async patterns, naming conventions actually in use.
   - Check for existing lint/type-check config (biome.json, ruff.toml, mypy.ini, tsconfig strict settings, eslint config). Anything already caught by tooling belongs in the "Skip" section, not as a review focus area, to avoid the reviewer flagging things CI already blocks on.
   - While sampling code, distinguish two categories of finding, they go in different sections of REVIEW.md (see step 4): a **convention to enforce going forward** (a pattern the codebase wants new code to follow, worth flagging on every PR that touches it) versus **pre-existing tech debt** (a problem that already pervades the codebase broadly, e.g. every router lacking a response schema). Never route the second category into per-PR Focus Areas: reviewing a small PR by relitigating a codebase-wide gap it didn't introduce is noise, not signal, and it repeats on every unrelated PR forever. Route it into "Known Tech Debt" instead, so it's recorded without being re-litigated.

4. **Draft `REVIEW.md`** at the repo root using this structure (see the reference example in `templates/REVIEW.example.md` for the level of specificity to aim for: concrete code patterns and file paths, not generic advice like "write clean code"):

   ```markdown
   # REVIEW.md

   ## Review Language

   All review comments MUST be written in <language, ask if unclear, default English>.

   ## Severity Levels

   - **Blocking**: Must fix before merge
   - **Nit**: Suggested improvement, not required for merge

   ## Focus Areas

   ### Blocking

   <3-8 concrete, project-specific rules found during the scan. Each should name
   a real pattern, file, or convention from this codebase, not a generic
   principle. Prefer showing a WRONG/CORRECT code snippet when the rule is
   about a specific code shape. Only rules that apply to code a PR actually
   touches, not codebase-wide gaps, those go in Known Tech Debt below.>

   ### Nit

   <Lower-severity style/efficiency preferences specific to this codebase.>

   ## Known Tech Debt (Reference Only, Do NOT Raise in PR Reviews)

   <Pre-existing, codebase-wide problems found during the scan that are out of
   scope for per-PR review: e.g. "routers return raw dicts with no response
   schema, checked as of <date>". Recorded so the problem isn't silently lost,
   but the review workflow must not comment on these unless the PR being
   reviewed is the one fixing them. Do not flag other code merely for matching
   this pattern if the PR didn't introduce or touch it.>

   ## Skip (Do NOT Review)

   <Things already enforced by linters/formatters/type-checkers/CI in this
   repo, list the specific tool so it's clear why it's excluded.>
   ```

   Keep the draft honest about uncertainty: if the scan didn't turn up enough evidence for a strong opinion in some area, leave it out rather than inventing generic filler ("write good tests", "avoid bugs"). Those add noise to every future PR review without adding signal.

5. **Present the draft to the user for review before finalizing.** This file encodes judgment calls about what matters in the codebase, including the Focus Areas vs. Known Tech Debt split, the user must confirm or edit it, not just receive it.

## Notes

- This skill intentionally does not use a reusable-workflow (`workflow_call`) architecture. Review focus areas are project-specific by nature, and the two workflow files rarely change once installed. Copying a template is simpler than maintaining a cross-repo/cross-account call chain, especially since this user's projects span two separate GitHub accounts (personal and a company org) with no shared secrets.
- When re-running this skill on a project that already has a REVIEW.md, treat it as an update pass: read the existing file, scan for what's changed (new patterns adopted, old ones removed), and propose a diff rather than regenerating from scratch.
