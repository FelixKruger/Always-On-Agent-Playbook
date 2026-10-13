# Laws and rails — how each law becomes something a test can fail

| Law | What it forbids | The rail | The test shape |
|---|---|---|---|
| Severity | silent drops | every turn appends an action row (`proposed → committed → verified` or `intentional_non_outcome` with a reason) | a golden asserts the row exists; "nothing happened" rows are allowed only with a reason |
| Enrichment | invented words in notes, titles, reminders | a validator rejects any effect whose text contains tokens not present in the message or the evidence pack | a golden with a message that has no date must not produce a reminder with one |
| Effect plan | parent verified with unserved children | the ledger marks each effect served; the parent closes last | crash injection between two effects → on restart the missing effect runs once, the served one does not repeat |
| Eval-first | fixes without a reproducing test | a commit template with a VERIFIED section that names the red-then-green golden | review refuses a product change whose golden was never red |
| No knob lies | settings that do nothing; numbers that are wrong | each config key has a test that flips it and observes the change | the reported count equals a direct query of the ledger |
| Quiet by default | unprompted chatter | one send path; a daily budget; everything else "held for digest" with a ledger row | a scripted 24-hour day yields exactly one unprompted message |
| Never claim | success claims the evidence does not support | reply text is checked for success verbs when the plan did not serve the effect; failed jobs are delivered as failed | a job whose last message says "done" but whose report is missing → reported as not finished |

## The validator rails (planner side)

The planner returns a typed plan. Before anything runs, deterministic code checks it:

- shape (schema) and vocabulary (only known effect kinds);
- enrichment (no new content tokens);
- referents (a short message like "save that" must resolve to something in the pack; otherwise capture + one clarifying question — but **never** force a question onto a reply-only answer to a question);
- success claims (a reply that says "saved" while the plan saved nothing is stripped, not fatal);
- confirm questions (an approval question cannot report results that haven't happened yet).

A plan the rails reject twice is **not an outage**: say what was rejected, keep the owner's words, retry later.

## The guard (hands side)

A hook that runs before every tool call of the work session, reading a policy file:

- **profiles** per task shape (report-only, workspace-write, federation = no mail/drive/vault, no secrets, no network);
- **unlockable denies** carry the unlock phrase ("say allow <name>");
- **hard denies** (secrets paths, the memory store, the repo's own config, backups) carry no unlock;
- the guard **fails closed** on a timeout.

Keep a **guard matrix test**: every (profile × tool × target) cell has an expected verdict; a change in the policy file changes the matrix, never the other way round.

## The report contract

A work session ends by writing `report.md` (summary, what changed, how it was verified, what is left, what it needs from the owner). Delivery reads the file, not the session's last message. A missing or empty summary is reported as such; a failed final step never erases finished work recorded earlier in the file.
