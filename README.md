# claude-code-review-workflow-creator-skill

A Claude Code skill that sets up `anthropics/claude-code-action` (automated PR review + interactive `@claude`) in a project, including a project-specific `REVIEW.md` grounded in a scan of the actual codebase rather than a generic checklist.

## What it does

- Copies `.github/workflows/claude-code-review.yml` and `claude.yml` into the current project
- Scans the project's stack, existing `CLAUDE.md`, and code patterns to draft a `REVIEW.md` with concrete, project-specific review focus areas
- Leaves the draft for you to review and edit before finalizing

## Install

Clone into your Claude Code skills directory (user-level, available across all projects):

```bash
git clone https://github.com/zxzinn/claude-code-review-workflow-creator-skill.git ~/.claude/skills/setup-claude-review-src
ln -s ~/.claude/skills/setup-claude-review-src/skills/setup-claude-review ~/.claude/skills/setup-claude-review
```

Or copy `skills/setup-claude-review/` directly into `~/.claude/skills/` or a project's `.claude/skills/`.

## Usage

In a project without Claude Code review set up yet:

```
/setup-claude-review
```

Claude will scan the project, install the workflows, and draft `REVIEW.md` for your review.

## Why not a reusable workflow?

The two workflow files (`claude-code-review.yml`, `claude.yml`) rarely change once installed, and `REVIEW.md` content is inherently project-specific — it should never be shared across repos. A `workflow_call` architecture also runs into secret-sharing limits across separate GitHub accounts/orgs (see the design notes in `skills/setup-claude-review/SKILL.md`). Copying a template once per project is simpler and avoids those cross-account constraints.

## Prerequisites

- `CLAUDE_CODE_OAUTH_TOKEN` secret configured in the target repo (or org). Run `/install-github-app` from Claude Code against the repo, or configure manually — see [Claude Code GitHub Actions docs](https://code.claude.com/docs/en/github-actions).
