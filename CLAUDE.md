# CLAUDE.md — Claude LinkedIn Assistant

Project rules loaded automatically when Claude works in this folder.

## What this is

A minimal job-search workspace that pairs `job_tracker.csv` with a Claude-driven daily orchestrator. The intended user path is simple: run `/jobs setup` once to configure the workspace, then run `/jobs daily` to check the tracker, find jobs, add strong matches, run referral outreach, and commit the day's work. Status edits are done by hand directly in the CSV.

See `README.md` for the user-facing overview and `REQUIREMENTS.md` for setup. The authoritative command flows live in `.claude/commands/jobs/`.

## Primary workflow

Default to the simple path unless the user asks for a specific sub-command:

1. `/jobs setup` — first-run setup and platform configuration.
2. `/jobs daily` — main orchestrator for day-to-day use.

`/jobs daily` is the recommended entry point after setup because it reduces user burden. It decides what to run next, skips empty steps, runs discovery, adds qualified jobs, runs outreach when needed, and commits once at the end.

Use individual flows (`check`, `find`, `add`, `outreach`, `indeed-setup`) for ad-hoc requests or when the user explicitly asks for that narrower action.

## Where the docs live (read before acting)

| File | Purpose |
|---|---|
| `CLAUDE.md` (this file) | Project-wide hard rules, canonical values |
| `README.md` | Human-readable workflow overview |
| `REQUIREMENTS.md` | Setup (Chrome extension, git) |
| `.claude/commands/jobs/_shared.md` | Shared rules loaded by every `/jobs` sub-flow |
| `.claude/commands/jobs/{setup,find,check,add,outreach,daily,indeed-setup}.md` | Per-flow procedures |
| `resumes/` | User's resume(s). Source of truth for name, target roles, skills, pitch |
| `resumes/search_profile.md` | (Optional) free-form preferences + platform config (LinkedIn/Indeed opt-in). Overrides resume-inferred defaults during `/jobs find`. |

## User resume — the source of truth

This repo has **no separate profile file**. Everything Claude needs about the user comes from their resume in `resumes/`. Read whatever's there (PDF / .tex / .md / .docx) whenever a flow needs:

- The user's **name** (top of resume) → used by `/jobs` Step 0 to verify the right LinkedIn account is active in Chrome.
- **Target job titles** (resume headline + experience entries) → used by `/jobs find` for searches.
- **Top skills / keywords** (skills section + repeated terms across bullets) → used by `/jobs find` for scoring.
- A short **third-person pitch** (current title + employer + years experience + 2-3 specializations) → used by `/jobs outreach` when drafting the first DM.

If `resumes/` is empty (only the README), stop the flow and tell the user to drop their resume in there before running anything.

If multiple resumes are present, read all of them and use the union of titles + skills.

## Optional search profile — what the user wants

If `resumes/search_profile.md` exists, read it alongside the resume. It's free-form prose: must-haves, deal-breakers, interest areas, salary floor, locations, anything that's not on the resume. Apply it as overlays on top of the resume-inferred plan in `/jobs find`. **The search profile wins on conflict** — the resume describes what the user *can* do, the search profile describes what they *want* to do. See `_shared.md` and `find.md` for how it's applied.

**Build the search profile WITH the user, not as homework.** When introducing the workspace, when starting a flow that would benefit from one, or any time the user mentions a preference ("I only want remote roles", "no crypto", "I'm into climate tech"), offer to build the file together. Don't tell them to "drop a search_profile.md" as if they have to write it themselves. Say something like:

> "Want me to build a search profile with you? Tell me your must-haves (locations, salary floor, level, role types), deal-breakers (industries, role shapes), and interest areas — I'll write `resumes/search_profile.md` for you."

When the user describes preferences in chat, capture them and **write `resumes/search_profile.md` using the Write tool** in the structured format from `resumes/README.md` (Must-haves / Interests / Deal-breakers / Salary). If the file already exists, read it first, then merge the new preferences in.

If the user clearly doesn't want a profile and wants to skip, fine — fall back to resume-only inference and don't keep nagging.

## Platforms

- **LinkedIn:** mandatory. Verified every `/jobs` run via Step 0.
- **Indeed:** optional. Opt-in via `/jobs setup` or `/jobs indeed-setup`. When enabled, `/jobs find` includes Indeed active search and recommendation harvest alongside LinkedIn and WebSearch. When disabled (default), `/jobs find` uses LinkedIn + WebSearch only.
- Indeed discovery is **fail-soft**: CAPTCHA, login failure, or blocked pages skip Indeed and continue with LinkedIn + WebSearch. Indeed must never block the rest of discovery.
- Indeed profile optimization is optional and confirmation-gated. Every public-facing profile edit requires explicit user approval before saving.
- Never commit user resume content, search profile, Indeed profile details, recommendations, or account-specific data.

## Hard rules

1. **`job_tracker.csv` is the single source of truth.** When updating, preserve every untouched row exactly.
2. **URL column is mandatory.** Every new job row must have a working apply link. Ask or search before saving.
3. **Date format:** always `YYYY-MM-DD`.
4. **Render tracker contents as a clean markdown table** — never raw CSV.
5. **NEVER use em-dashes (—) in any user-facing message the user sends** — emails, LinkedIn DMs, follow-ups, subject lines, anything outgoing. Use commas, periods, parentheses, or split sentences instead. Em-dashes are a tell of AI-written text. Scan every draft for `—` before showing it; rewrite if found. Internal notes, tracker, contacts files are fine.
6. **Connection requests: always "Send without a note", never personalized.** LinkedIn rate-limits personalized invites; bulk connection requests must always go through the "Send without a note" button. Per-company quota = `max(0, (10 − count_1st_degree) × 5)`. **No global weekly cap** — keep going until LinkedIn pushes back (CAPTCHA, rate-limit notice). See `.claude/commands/jobs/outreach.md` Step 2C and Step 4B.
7. **Never attempt file uploads.** LinkedIn / Gmail / ATS file inputs are blocked from automation. If a flow requires an attachment, type the message and stop — the user handles the attach + send.
8. **Outgoing messages are first person.** Every DM, email, or message sent from the user's account must be written as the user speaking ("I'm a senior data scientist...", "I saw the role..."). Never draft outgoing messages in third person ("She is a senior data scientist..."). Third person in a DM reads as obvious AI slop.
9. **DMs require user confirmation before sending.** Draft the message and show it to the user. Only click Send after they approve. Connection requests (no-note) do not need per-request confirmation.

## Canonical values

- **Statuses:** `To Apply` → `Applied` → `Recruiter Call` → `Phone Screen` → `Onsite` → `Offer` / `Rejected` / `Withdrew`
- **Priorities:** `HIGH`, `MEDIUM`, `LOW`
- **Referral Status:** `Not Needed` / `Outreach Pending` / `Connection Pending` / `Outreach Sent` / `Got Referral` / `Declined` / `No Referral`

## Referral-needed threshold

Companies with **>500 employees OR >100K LinkedIn followers** → `Referral Needed=YES`. Otherwise NO.

When `Referral Needed=YES`:
- `Referral Deadline = Discovered Date + 5 days`
- Outreach via LinkedIn DM (Chrome) to existing 1st-degree connections at the company.
- If no 1st-degree connection exists, surface that as a "Connection Pending" candidate the user must connect with manually.

## Unified command

Use `/jobs` for all tracker operations. Sub-flows: `setup` · `daily` · `check` · `find` · `add` · `outreach` · `indeed-setup`

Recommended path: `/jobs setup` once, then `/jobs daily` for normal use. Treat `/jobs daily` as the orchestrator, not just another sub-command.

## Tone

Match the user's voice. Skip over-explanation. Default to terse, technical responses. When drafting outreach DMs, follow the templates in `outreach.md` and build the pitch from the user's resume in `resumes/` — do not invent claims about their experience.
