# FIND (live job search)

## Step 1 — Read resume(s) AND search profile, then build the search plan

### 1A. Resume

Read every resume file in `resumes/` (skip the README and `search_profile.md`). Extract:

- **Target roles** — the user's most recent role title + adjacent titles (e.g. if they're a "Senior Data Scientist", also search "Staff Data Scientist", "Senior Product Data Scientist", "Lead Data Scientist"). Pull 3-6 titles total.
- **Top keywords / skills** — 10-15 from the skills section + repeated terms across bullets (technologies, methodologies, domain terms).
- **Default location** — the city/state from the resume's contact line.

If `resumes/` has no resume (only the README and/or `search_profile.md`), stop and tell the user to drop their resume in.

### 1B. Search profile (optional, but read every time)

Look for `resumes/search_profile.md`. If present, read the whole file as free-form prose. Apply it as overlays on the resume-inferred plan:

- **Locations:** if the search profile names locations, those replace the resume's default. (Example: profile says "Remote only" → drop the resume's city, search Remote only.)
- **Titles:** if the profile mentions specific titles or role types, narrow the title list to match. If it explicitly excludes some (e.g. "no people management"), drop manager-track titles.
- **Keywords:** add any domain interests from the profile (e.g. "climate tech", "LLM infra") to the keyword pool used for searches and scoring.
- **Salary floor:** if the profile gives a number, use it for scoring (jobs at or above floor get +1).
- **Deal-breakers:** if the profile lists exclusions (e.g. "no crypto"), drop matching jobs entirely from results, before scoring.
- **Company-size or stage preferences:** keep in mind for the score; small-company-only profile + a Fortune-500 result → drop or score low.

If the search profile contradicts the resume (e.g. resume is full of ML, profile says "I want to switch to PM"), the profile wins. The resume describes what the user *can* do; the profile describes what they *want* to do.

**No salary filter unless the search profile sets one.** Print the inferred plan as a one-line summary so the user can see it scrolling past, then proceed straight to Step 2 — no confirmation prompt.

---

## Step 2 — Search (Chrome + WebSearch in parallel)

**LinkedIn via Chrome — for each target role, for each location:**

```
https://www.linkedin.com/jobs/search/?keywords=<role>&location=<location>&f_TPR=r604800&sortBy=DD
```

(`f_TPR=r604800` = last 7 days · `sortBy=DD` = date posted descending)

Read results. Extract per job: Company, Role, Location, Type (Remote/Hybrid/On-site), Salary (if shown), URL. Paginate up to 2 pages per search.

**WebSearch sweep (run in parallel with the Chrome searches):**

Build 3-5 queries combining target titles, top keywords, and locations inferred from the resume. Examples:
- `"<target title>" jobs "<location>" site:linkedin.com OR site:indeed.com 2026`
- `"<target title>" "<keyword>" "<keyword>" Remote jobs 2026`
- `"<adjacent title>" "<keyword>" "<location>" 2026`

Combine all results into one pool.

**Rate limiting:** If LinkedIn throttles or shows a CAPTCHA mid-search, stop and tell the user. Work with results collected so far, or re-run later in smaller batches.

---

## Step 3 — Dedup, score, rank

**Dedup:** Remove any result where Company + similar Role already exists in `job_tracker.csv`. Track skipped names to report at the end.

**Score each job (0-10) against the search plan from Step 1:**

- Title matches a target role exactly: +3
- Title is adjacent (e.g. "Staff" instead of "Senior", or "Analyst" vs "Scientist" with overlapping skills): +1.5
- Location matches a profile-allowed location OR is Remote (when profile allows Remote): +2
- Each keyword overlap (resume keywords ∪ search profile interests): +0.5 (cap at +3)
- Posted ≤7 days ago: +1
- Salary at or above the profile's stated floor (skip this rule if no floor was given): +1
- **Search profile match for company stage / size / domain** (e.g. profile says "climate tech", job is at a climate-tech company): +1

**Hard exclusions from the search profile** (deal-breakers like "no crypto", "no consulting"): drop the job entirely BEFORE scoring. Don't surface it in the results table.

Drop jobs where total score < 4. Sort descending. Take top 20.

---

## Step 4 — Present and add (automatic)

Show as a markdown table for visibility:
`# | Company | Role | Location | Salary | Score | URL | Why it fits`

**Then add ALL surfaced candidates to the tracker automatically — no per-candidate confirmation.** The dedup + score filter already runs upstream, so anything that reaches this step should be added. The user can prune later.

For each candidate → run the ADD flow automatically:
- Pre-fill Company, Role, Location, URL from the result
- Set Priority by score: score ≥8 → HIGH, 6-7 → MEDIUM, <6 → LOW
- Run referral check (size threshold from `add.md`)

After adding, show:
- A summary table of the rows added (Priority, Company, Role, URL)
- A one-line **Skipped** list of companies already in the tracker that were deduped
