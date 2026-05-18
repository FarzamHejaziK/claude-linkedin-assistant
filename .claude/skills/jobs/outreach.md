# OUTREACH (cold sweep at a company)

The outreach flow does two things at a target company:

1. **Send connection requests** to 2nd-degree contacts (recruiters, peers, leads). Always "Send without a note" — LinkedIn rate-limits personalized invites.
2. **Send a first DM** to your existing 1st-degree connections at the company.

Out of scope (you handle these manually):
- ❌ Replying to DMs once a thread is warm
- ❌ Follow-up nudges on stale outreach
- ❌ Sending a resume / file attachments

---

## Step 1 — Pick a company

**If the command was called with an argument** (`/jobs outreach Mixpanel`), use that company. Skip the rest of this step.

**Otherwise**, read `job_tracker.csv` for outreach candidates: rows where `Referral Needed=YES` AND `Referral Status` ∈ `Outreach Pending` / `Connection Pending`.

- **Has candidates** → auto-pick the most urgent one: smallest `Days Left` (Referral Deadline − today; negative = overdue first). Tiebreaker: oldest `Discovered Date`. Print:
  > `Picked: <Company> — <Role> (deadline in <N> days)`

  Proceed to Step 2 immediately. No confirmation prompt. (If you want to override, stop the run and re-invoke as `/jobs outreach <OtherCompany>`.)

- **No candidates in the tracker** → ASK the user: "Tracker has no companies queued for outreach. Which company do you want to run outreach on? (You can also run `/jobs find` first to discover jobs.)" Wait for a company name, then proceed with that.

If the picked company is `Connection Pending`, jump to **Step 4 — Check pending acceptances** instead of Step 2.

---

## Step 2 — Find contacts (1st-degree FIRST, then 2nd)

LinkedIn via Chrome:

1. Navigate to `https://www.linkedin.com/company/<slug>/people/`. Extract the numeric company ID by scanning the page for `urn:li:fsd_company:<id>` and picking the most-frequent match.

2. **Step 2A (MANDATORY): 1st-degree only**, no keyword filter:
   ```
   https://www.linkedin.com/search/results/people/?currentCompany=["<id>"]&network=["F"]
   ```
   Read every result. These are your existing connections at the company and are always the top picks for direct DMs.

3. **Step 2B: 2nd-degree fallback**, with role-relevant keywords:
   ```
   https://www.linkedin.com/search/results/people/?currentCompany=["<id>"]&network=["S"]&keywords=recruiter OR <role keyword>
   ```
   Use role keywords inferred from the user's resume(s) in `resumes/` (e.g. "data scientist" / "engineer" / "product manager").

4. **Never combine `network=["F","S"]` in one search.** LinkedIn sorts combined results by keyword relevance, which buries 1st-degree contacts below 2nd-degree ones. Always do the two searches separately.

5. URL params are `network` and `currentCompany`, NOT `facetNetwork` / `facetCurrentCompany`.

Extract per contact: name, title, connection degree, mutual-connection count, profile URL.

**Priority order:**

1. **Any 1st-degree connection** at the company — always beats 2nd-degree, regardless of title. Goes through Step 3 (DM flow).
2. **2nd-degree recruiter / Talent Acquisition.** Connection request via Step 4.
3. **2nd-degree peer** (data scientist, engineer, product, etc. — match on the user's role from their resume in `resumes/`). Connection request via Step 4.
4. **2nd-degree lead / manager** in the user's function. Connection request via Step 4.

Show two ranked tables, one per network tier:

```
1st-degree (DM directly):
# | Name | Title | LinkedIn URL | Mutuals

2nd-degree (connection request):
# | Name | Title | LinkedIn URL | Mutuals
```

---

## Step 2C — Connection-request quota (per company)

Use a per-company quota based on how many 1st-degree connections the user already has there:

```
quota = max(0, (10 − count_1st_degree) × 5)
```

| 1st-degree available | Connection requests to send |
|---|---|
| 10+ | 0 (already have plenty of internal contacts) |
| 9 | 5 |
| 8 | 10 |
| 7 | 15 |
| 5 | 25 |
| 2 | 40 |
| 1 | 45 |
| 0 | 50 |

**No global weekly cap is enforced.** Send the per-company quota in full. The only stop signal is LinkedIn itself — see Step 4B for the CAPTCHA / rate-limit handling.

---

## Step 3 — DM existing 1st-degree connections

For each 1st-degree contact selected:

### A. Open profile and verify connection level

Navigate to the contact's profile URL. Confirm they show as 1st connection.

### B. Prior-thread check (MANDATORY before drafting)

Click Message to open the chat overlay. Scan the thread history:

| Thread state | Action |
|---|---|
| No prior messages | Proceed to **C (draft message)** |
| Prior message from user, contact replied | Skip — warm thread, user handles manually |
| Prior message from user, no reply, ≥7 days ago | **SKIP** — do not re-engage. Log `Skipped (prior unanswered DM <date>)` |
| Prior message from user, no reply, <7 days ago | Skip for now (still in reply window). Log and move on |

Close the dialog if skipping.

### C. Draft and send the first DM

**Step 1: Build the pitch from the user's resume.** Read whatever is in `resumes/`. Extract:
- The user's **first name** (top of the resume)
- The user's **current title** + **employer** (most recent role)
- **Years of experience** (from the experience timeline)
- **Top 2-3 specializations** (from the headline / summary / repeated keywords)

Construct a single third-person sentence about the user. Examples of the right shape:
> "She is a senior data scientist with 5+ years in experimentation and product analytics, currently at <Employer>."
> "He is a staff ML engineer with 8+ years building production LLM systems, currently at <Employer>."

The pronoun: infer from the name if obvious, otherwise default to omitting the pronoun and starting with the title ("A senior data scientist with..."). Don't invent details not in the resume.

**Message style rules (strictly enforced):**
- Short: 2-3 sentences max
- Conversational, not corporate
- No em-dashes (—) anywhere
- No filler phrases like "I hope you're doing well"
- No "Best, <name>" sign-off — end naturally
- Background sentence is third person; greeting and sign-off are first person

**Draft template (adapt by contact's title):**

*If the contact is a recruiter:*

> Hi <First>, <third-person pitch built from the resume>. I saw the <Role> role at <Company> and it looks like a great fit. Would you be open to a referral or sharing more about the team?
>
> Thanks,
> <user's first name>

*If the contact is a peer (DS / engineer / similar):*

> Hi <First>, <third-person pitch built from the resume>. I saw the <Role> opening at <Company> and it looks like a great fit. Would you be open to a referral if it feels right? Would really appreciate it.
>
> Thanks,
> <user's first name>

**Send directly, no confirmation.** Find the Message button → click → fill the message field with the draft → click Send. Then move on to the next contact in the loop.

If LinkedIn shows a CAPTCHA, "Verify it's you", or any rate-limit notice mid-loop → STOP immediately (same Step 4B rule applies to DMs as to connection requests). Report which contacts were sent and which are still pending.

### D. Log

`outreach/<Company>_contacts.md`:
```
| <Name> | <Title> | <LinkedIn URL> | 3. Cold DM Sent | <today> | First DM, 1st-degree |
```

Update tracker: `Referral Status=Outreach Sent`. Append to Notes: `Outreach: <Name> (<Title>) <today>`.

---

## Step 4 — Send connection requests to 2nd-degree contacts

Apply the per-company quota from Step 2C. Pick the top N from the 2nd-degree ranked table (recruiters first, then peers, then leads).

### A. Pre-batch announcement (no confirmation)

Print a one-paragraph summary so the user can see what's about to happen scrolling past — but do NOT wait for confirmation:

```
Sending <N> connection requests at <Company> (no-note path).
  1. <Name> — <Title>
  2. <Name> — <Title>
  ...
Will stop early if LinkedIn shows a CAPTCHA or rate-limit notice.
```

Then proceed straight to the send loop in Step 4B.

### B. Send loop

For each contact in the batch:

1. Navigate to the contact's profile URL.
2. Find the **Connect** button. (If buried under "More", click More first.)
3. Click Connect.
4. **A dialog appears asking to add a note.** Click **"Send without a note"** (or the equivalent no-note button).
   - **NEVER click "Add a note".** This burns a personalized-invite quota for no reason.
   - If only a note-required path is shown (rare), skip and log "Note required, skipped".
5. Wait for confirmation that the request was sent.

**Pace the loop:** insert a small delay between requests (~2-3s).

**Stop conditions (mandatory):** if at any point during the batch you see a CAPTCHA, a "Verify it's you" challenge, a "you've reached the weekly limit" notice, or any unusual interstitial — STOP the loop immediately. Do not attempt to click through. Tell the user:
- What was hit (CAPTCHA / rate-limit / other)
- How many requests went through before the block
- Which contacts are still pending

Do not retry until the user explicitly resumes the run (typically next day or later). Clicking through a CAPTCHA via automation makes the bot signal worse; let the user handle it manually if they want to.

### C. Log each request

For each successfully sent request, append to `outreach/<Company>_contacts.md`:

```
| <Name> | <Title> | <LinkedIn URL> | 1. Connection Pending | <today> | Sent without note |
```

### D. After the batch

Update tracker:
- If the company had no 1st-degree DMs sent (Step 3): `Referral Status=Connection Pending`
- If 1st-degree DMs were sent: leave the Outreach Sent status from Step 3

Append to Notes: `ConnReq: <N> contacts <today>`.

Tell the user:
> "Sent <N> connection requests at <Company>. They go in your `Connection Pending` queue — `/jobs check` will surface them once acceptances come in. Run `/jobs outreach <Company>` again later to send DMs to whoever accepted."

---

## Step 5 — Check pending acceptances (returning user, company in `Connection Pending`)

When `/jobs outreach <Company>` is run for a company already in `Connection Pending`:

1. Read `outreach/<Company>_contacts.md`. Find rows where Stage=`1. Connection Pending`.
2. For each, navigate to the LinkedIn profile and check connection level.
   - **Now 1st** → accepted. Update stage to `2. Connection Accepted`. Add to a "ready to DM" queue.
   - **Still 2nd** → still pending. Skip.
3. After the check, run **Step 3 (DM flow)** on the "ready to DM" queue.
4. Anyone still pending past `Referral Deadline` → leave them; user can decide to write them off.

---

## Step 6 — After outreach

```
git commit -am "outreach: <Company> → <N> conn reqs, <M> DMs"
```

---

## Hard rules (re-stated)

- **Always "Send without a note"** for connection requests. Never click "Add a note".
- **Per-company quota** from the formula in Step 2C. No global weekly cap — only LinkedIn-side blocks (CAPTCHA, rate-limit notice) stop the loop.
- **No per-message and no per-batch confirmations.** Connection requests fire silently up to the quota; DMs are typed and sent in the loop without asking. The only stop signal is a LinkedIn-side block.
- **No em-dashes** in any drafted message. Scan for `—` and rewrite.
- **No file attachments.** This flow is text-only LinkedIn (DMs and connection requests). If the user wants to send a resume, they do it manually after a contact agrees to refer.
- **Don't ask the contact for their email** as a workaround for upload failures.
