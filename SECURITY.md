# Security Policy

## Reporting a Vulnerability

If you find a security issue, **do not open a public GitHub Issue.** Instead, email the maintainer directly:

- [@FarzamHejaziK](https://github.com/FarzamHejaziK)

Include:
- A clear description of the issue.
- Reproduction steps.
- The specific commit / file / line if applicable.
- Your assessment of impact.

We aim to acknowledge reports within 5 business days and respond with a timeline once triaged.

## Scope

This repo is a markdown-only Claude Code workspace. The "security surface" is small but real:

- **Instructions Claude follows** (under `.claude/skills/`). A malicious PR could embed instructions that exfiltrate user data (resume content, contact lists, LinkedIn cookies) or trigger destructive Chrome actions.
- **Scripts** if any are added later.
- **`.gitignore` correctness** — protecting the user's resume and outreach contact log from accidental commits.

If you spot something in any of those areas that could harm a downstream user of this repo, please report it.

## What's Out of Scope

- LinkedIn's own Terms of Service. This repo automates LinkedIn behavior; that risk lies with the user, not the maintainer. We follow LinkedIn's published rate limits and stop on platform-side blocks (CAPTCHA, "weekly limit reached"). Reports about that aren't security issues; they're product-design discussions and belong in regular Issues.
- Anthropic's Claude API or the Claude Code desktop app. Report those upstream.
