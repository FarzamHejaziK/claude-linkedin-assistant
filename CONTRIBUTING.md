# Contributing to Claude LinkedIn Assistant

Thanks for contributing.

## Quick Links

- Workflow overview: [`README.md`](README.md)
- Setup: [`REQUIREMENTS.md`](REQUIREMENTS.md)
- Project rules / canonical values: [`CLAUDE.md`](CLAUDE.md)
- Per-flow procedures: [`.claude/skills/jobs/`](.claude/skills/jobs/)
- Maintainers: [`MAINTAINERS.md`](MAINTAINERS.md)

## How to Contribute

**Pull requests only.** Direct pushes to `main` are blocked.

1. **Bug fixes / small improvements**: fork, branch, open a focused PR.
2. **Bigger changes** (new flows, structural changes, anything touching multiple `.claude/skills/jobs/*.md` files): open an Issue first to align scope before doing the work.
3. **Questions**: open an Issue.

## Before You Open a PR

- Keep the PR scoped to one concern. Don't bundle unrelated changes.
- The whole repo is markdown + a CSV. There's no build step and no test suite. The "test" is reading the diff carefully and confirming the prose is clear, the rules are consistent across files, and nothing contradicts.
- If you're changing flow behavior, walk through each affected file in `.claude/skills/jobs/` and check that:
  - The flow file itself, `_shared.md`, `CLAUDE.md`, `README.md`, and `SKILL.md` all agree on the new behavior.
  - The "Hard rules (re-stated)" section at the bottom of the flow file matches the body of the file.
- For any new `/jobs <subcommand>`: add the file under `.claude/skills/jobs/`, register it in the menu and dispatch table in `.claude/skills/jobs/SKILL.md`, and document it in `README.md`.

## PR Description Checklist

Use the template — explain **what changed, why, and how you verified it.** "I read the diff and re-rendered the README locally" is a fine verification for docs-only changes.

## Review Policy

- **CODEOWNERS review is required before merge.** All paths default to [@FarzamHejaziK](https://github.com/FarzamHejaziK).
- **Final merge authority remains with the project owner** (BDFL model). Even with code-owner approval from a future co-maintainer, the project owner has the last word on direction.

## What to NOT contribute

- Code that violates LinkedIn's Terms of Service in ways the current scope doesn't already touch (e.g. scraping at scale, automating logged-out browsing, harvesting user data). The current scope is intentionally narrow; expansions need an Issue first.
- Personalized connection-request notes (the project deliberately uses "Send without a note" only).
- Reply automation, follow-up automation, application-form auto-submission. Those are out of scope by design.
- Re-introducing per-step confirmation prompts that were intentionally removed. If you think a confirm is warranted somewhere, open an Issue first.

## Reporting Security Issues

See [`SECURITY.md`](SECURITY.md). Don't open a public Issue for a security report — email instead.
