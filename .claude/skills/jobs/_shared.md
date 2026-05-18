# Shared context — loaded by all /jobs sub-flows

Working directory: the cloned repo root (where `job_tracker.csv` lives).
Today's date: read from system (YYYY-MM-DD).

---

## User resume — the source of truth about WHO they are

Read whatever resume(s) the user has dropped into `resumes/`. Supported formats: `.pdf`, `.tex`, `.md`, `.docx`. Extract on demand:

- **Name** (top of resume) — used by Step 0 LinkedIn verification.
- **Current title + employer + years experience + 2-3 specializations** — used by `/jobs outreach` to build a third-person pitch sentence for the DM template.
- **Target job titles** (resume headline + most recent role titles + role-aligned alternatives) — used by `/jobs find` searches.
- **Top skills / keywords** (skills section + repeated terms across bullets) — used by `/jobs find` scoring.

If multiple resumes are present, read all of them and use the union. Never invent details that aren't on the resume.

If `resumes/` is empty (only the README), stop the flow and tell the user to drop their resume in.

## Search profile (optional) — the source of truth about WHAT they want

Look for `resumes/search_profile.md`. This is an OPTIONAL free-form file where the user writes their specific job-search preferences: must-haves, deal-breakers, interest areas, salary floors, location constraints, company-size preferences, anything that's not on the resume.

If it exists:
- Read it as plain prose. No structured schema is enforced — it can be bullets, paragraphs, mixed.
- Treat it as **higher-priority signal than the resume** when there's a conflict. The resume says what the user *can* do; the search profile says what they *want* to do.
- Examples of how it shapes flows:
  - "Remote only" → drop on-site results from `/jobs find`, even if the resume has a city in the contact line.
  - "$200K minimum" → boost or filter on salary in the score.
  - "Climate tech" → add to the keyword pool for searches and scoring.
  - "No crypto" → exclude matching jobs entirely.
  - "Senior IC, no people management" → drop manager-track titles.

If it doesn't exist, the search profile is inferred entirely from the resume. Don't tell the user this is missing — the file is optional.

---

## When to use WebSearch

Use `WebSearch` proactively whenever Chrome returns incomplete info or you need data not on screen:
- Company headcount, funding, LinkedIn followers
- Finding a careers page or verifying a job is still open
- Finding the right LinkedIn URL for a company or person
- Any question you can answer faster with a search than by navigating Chrome

---

## Tracker columns

`Priority, Company, Role, Location, Type, Salary, Status, Applied Date, Next Action, URL, Notes, Discovered Date, Referral Needed, Referral Status, Referral Deadline, Apply Via`

**Canonical statuses:** `To Apply` → `Applied` → `Recruiter Call` → `Phone Screen` → `Onsite` → `Offer` / `Rejected` / `Withdrew`

**Priority values:** `HIGH` / `MEDIUM` / `LOW`

**Referral Status values:**
`Not Needed` / `Outreach Pending` / `Connection Pending` / `Outreach Sent` / `Got Referral` / `Declined` / `No Referral`

- `Connection Pending`: connection request sent ("Send without a note"), waiting for acceptance.
- `Outreach Sent`: a first DM was sent to a 1st-degree connection (either an existing one, or one who recently accepted a connection request).
- `Declined`: contact replied but said they can't refer.
- `Referral Deadline`: Discovered Date + 5 days (only when Referral Needed=YES).
- `Apply Via`: `LinkedIn Easy Apply` / `Company Website` / `Blocked`.

---

## outreach/<Company>_contacts.md format

Create this file when first reaching out to anyone at a company. One file per company.

```markdown
# Outreach — <Company>

| Name | Title | LinkedIn | Stage | Last Action Date | Notes |
|---|---|---|---|---|---|
| Jane Doe | Sr Data Scientist | linkedin.com/in/... | 3. Cold DM Sent | 2026-04-22 | First DM, 1st-degree connection |
| John Smith | Recruiter | linkedin.com/in/... | 1. Connection Pending | 2026-04-22 | 2nd degree, user to send request |
```

### Conversation stages

| Stage | When |
|---|---|
| **1. Connection Pending** | Connection request sent ("Send without a note"), waiting for the contact to accept |
| **2. Connection Accepted** | They accepted, ready for first DM |
| **3. Cold DM Sent** | First outreach message sent (only via this repo's outreach flow), awaiting reply |
| **4. Reply Received** | They replied — handled by the user, not by Claude |
| **5. Terminal** | No reply within window, user gave up, or declined (terminal) |

Stages 4 and 5 are handled by the user manually (this repo doesn't automate replies or follow-ups).

---

## Writing style — STRICT

**NEVER use em-dashes (—) in any user-facing message: LinkedIn DMs, anything sent to a contact.** Use a comma, period, parentheses, semicolon, or split into two sentences instead. This is a hard rule. Em-dashes are a tell of AI-written text.

**Fit summaries / background paragraphs are written in THIRD PERSON.** The greeting stays first-person ("Hi <name>,"), but the background paragraph shifts to third person ("She is a senior data scientist..."). The signature is the user's first name. This applies to outreach DMs only. The third-person sentence is built from the user's resume in `resumes/` — see `outreach.md` for the construction recipe.

This rule does NOT apply to:
- Internal documentation, tracker notes, contacts.md
- Technical strings, file paths, code

When drafting any user-facing text, scan for `—` before showing it to the user and rewrite if present.

---

## General rules

- Render all tracker data as markdown tables, never raw CSV.
- URL column mandatory for any new job row.
- Date format always YYYY-MM-DD. If creating folders: spaces → underscores, strip commas/special chars.
- Connection requests: always use the "Send without a note" path — never personalized. Per-company quota in `outreach.md` Step 2C. No global weekly cap; the loop only stops on a LinkedIn-side block (CAPTCHA, rate-limit notice).
- Use `WebSearch` any time Chrome is slow, blocked, or returns incomplete info.

---

## File uploads — not supported, do not attempt

LinkedIn DM file attach, Gmail compose, Greenhouse, Ashby, and most ATS forms block automated file upload. Computer-use can't drive the macOS file picker either. If a flow requires an attachment, **type the message and stop** — the user attaches and sends.

This repo's flows are designed to never need an upload: outreach is text-only first DMs. If the user wants to send a resume in a follow-up, they do it manually.
