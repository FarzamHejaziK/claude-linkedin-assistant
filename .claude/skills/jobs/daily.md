# DAILY (orchestrator)

`/jobs daily` runs the daily sequence end to end. Each step has a checkpoint: the user can run, skip, or stop. Empty steps are auto-skipped.

For ad-hoc work, run individual `/jobs <sub-command>` directly.

---

## How this command behaves

1. Run STEP 0 (Chrome + LinkedIn check from `jobs.md`) once at the start.
2. **Pre-flight: read `job_tracker.csv`. If it has 0 data rows (just the header), this is a first-run / empty-tracker state.** Skip Step 1 (check) entirely — there's nothing to dashboard. Jump straight to Step 2 (find). Tell the user: "Tracker is empty, starting with discovery."
3. Otherwise, walk through the steps below in order, **end to end with no between-step prompts**. Each step runs to completion, then the next one starts automatically. The user is not asked to continue between steps.
4. Before each step, print a one-line preview of what's queued. If the queue is empty, auto-skip and announce.
5. At the end, run the commit step automatically and print the daily summary.

If the user invokes a single sub-command from inside the daily flow (e.g. types `/jobs outreach Walmart`), exit the daily orchestrator and let that sub-command run standalone.

---

## Step 1 — `/jobs check` (dashboard)

**Skip if the tracker is empty** (per the pre-flight check above — go straight to Step 2).

Otherwise: read `check.md`, execute. Show all sections. Continue to Step 2 automatically.

---

## Step 2 — `/jobs find` (discover new jobs)

Read `find.md`. Surface candidate jobs based on the user's profile. Continue to Step 3 automatically.

---

## Step 3 — `/jobs add` (capture finds)

For each job surfaced in Step 2, read `add.md` and add to the tracker. The find flow already auto-adds in batch — this step exists to handle anything the user wants to add manually that wasn't surfaced (e.g. a link they saw elsewhere).

Skip if there's nothing manual to add. Continue to Step 4 automatically.

---

## Step 4 — `/jobs outreach` (cold sweep on new HIGH companies that need a referral)

**Skip if:** no `Referral Status=Outreach Pending` companies with `Priority=HIGH`.

**Else:** for each new HIGH company that needs a referral, read `outreach.md` and run the sweep. The sweep does both: connection requests to 2nd-degree contacts ("Send without a note", per-company quota from Step 2C) AND first DMs to existing 1st-degree connections. Keep going across companies until LinkedIn pushes back (CAPTCHA / rate-limit notice) — at which point stop and tell the user.

Continue to Step 5 (commit) automatically.

---

## Step 5 — Commit

Run a single git commit covering everything done in this session. Group changes by category in the commit message. Format:

```
git add -A && git commit -m "$(cat <<'EOF'
daily <YYYY-MM-DD>: <short summary>

New jobs added:
- <Company>: <Role> [<Priority>]

Connection requests sent:
- <Company>: <N> contacts (no-note)

Outreach DMs sent:
- <Company>: <Name>

EOF
)"
```

Skip if no changes were made.

---

## Final summary

After commit (or after the user quits), print:

```
Daily run complete (<YYYY-MM-DD>):
  ✅ <I> new jobs added
  ✅ <K> connection requests sent (no-note) — total in last 7 days: <X>
  ✅ <M> first DMs sent to 1st-degree connections
  ⏳ Pending: <list of items waiting on the user>

Tomorrow's surface:
  - <Co> deadline tomorrow
  - <Co> awaiting acceptance for <N> connection requests
```

---

## Hard rules (re-stated)

- Connection requests always use "Send without a note". Per-company quota from `outreach.md` Step 2C; no global weekly cap (loop only stops on LinkedIn-side block per Step 4B).
- Never sends a DM with an attachment. The first DM is text only.
- No per-message and no per-batch confirmations during outreach. The only stop signal is a LinkedIn-side block (CAPTCHA, rate-limit notice).
- Never use em-dashes in any drafted message.
- One commit at the end of the daily run, not per-step. No between-step prompts; the orchestrator runs end-to-end.
