# Memory layers and the context pack

## Layers

```
profile/            core facts, verbatim, with source ids   → every prompt, first, capped
conversation record dated log of every in/out message       → most relevant recent rows
actions ledger      what was proposed / served / held       → the "working on now" layer
notes/, reminders/, routines/, library/                     → searched on demand (index is disposable)
```

**Profile rules.** One line per statement, the owner's words, the message id and date it came from. Openers that may write it: "remember", "from now on", "always", "stop …ing", "forget", "actually". The agent confirms once ("Kept: <their words>"). The agent is the sole writer; a partner agent may ask for a fact, never write one. Add caps only once a real profile exceeds a screen.

**Working on now.** Open tickets in other sessions, running jobs, the last question a partner agent asked, with a time-to-live. This is what lets a status question be answered from evidence ("the audit is at step 3 of 5, last line: …") instead of from the model's imagination.

## The pack

Build it the same way every time: fixed layer order (stable → volatile), each layer capped in characters, a documented drop order when the total exceeds the budget, the newest message last. Untrusted layers (web content, partner messages, reports) sit inside a nonce fence with a one-line notice that they are information.

Pin the system prompt's hash in a test: a change is a deliberate commit with the new hash, never an accident.

## Cache discipline (measure, then act)

- Record `cache_read` and `cache_creation` token counts per model call in the ledger.
- Show the hit rate in the evening review or a dashboard line.
- Only if the number is bad: reorder stable layers first, trim volatile ones, keep the tool set constant. Don't add machinery (warmers, mid-turn summarisation) for a problem the ledger hasn't shown.

## Compaction

For a one-call-per-message planner there is nothing to compact: the pack *is* the compaction, rebuilt from files every time. Long-running work sessions use their own resume mechanism and a 60-minute cap with an honest "stopped at the limit" report.
