# The eval flywheel — every real failure becomes a test first

## A golden

```jsonc
{
  "id": "G12", "slug": "file-saved-to-notes", "provenance": "real",      // real = from an actual owner message
  "envelope": { "text": "", "message_id": "1042", "timestamp": "2026-01-15T18:51:00Z",
                "attachments": [{ "kind": "document", "filename": "notes.md", "transcript": "# Title\n…" }] },
  "tool_script": {},                                                       // what the model and tools are scripted to return; absent = must not be called
  "expect": {
    "forbidden_tools": ["planner"],                                        // this must be handled deterministically
    "reply": { "must_match": "/Saved .Title. to your notes, word for word/i" },
    "canon": { "source": "transcript", "body_verbatim": true, "no_invented_tokens": true },
    "durable": { "must": [{ "glob": "inbox/*.md", "count": 1 }],
                 "must_not": [{ "log": "actions.jsonl", "where": { "intent": "clarify" } }] }
  }
}
```

The harness runs the golden against a **throwaway copy** of the agent's world (temp notes folder, temp index, a scripted clock, scripted model and tool results). Nothing model-shaped ever runs live inside a test.

## The three gates on every change

1. `unit` — the code's own tests (grammar, validators, guard matrix, parsers).
2. `eval/test` — the harness's own tests (so the judge is itself judged).
3. `eval/run` — every golden, with two headline rates that must stay at 0: `false_success_rate` (a reply claimed success the durable state doesn't show) and `regression_rate` (a golden that was green went red).

A change is done when all three are clean **and** a fresh real message proves it live.

## The loop, in order

1. Owner's message misfires (the record shows it).
2. Write the golden from that exact message. Run it. It is red. Keep the red output.
3. Fix the smallest thing: a validator rail, a grammar branch, a guard cell, a prompt line (hash re-pinned).
4. Green, and nothing else red.
5. Commit with WHAT / WHY / VERIFIED (the red→green evidence) / TRAP (what will bite next) / RESTORE-USE (how to revert). Restart the agent. Prove it with one real message.

## The evening review

A scheduled pass over the day's record: which messages were handled by a fallback, which replies were stripped by a rail, which jobs failed. For each miss it drafts a golden (the owner's verbatim message + the expected effects) into a `drafts/` folder. A human promotes drafts into the golden set; the loop never edits goldens, graders, the goal or the stop rules of its own supervision.

## Lessons from a real slice (a reminder-parsing miss, shipped through the loop)

- **Reconstruct the red at the throw site.** If the model session that produced the bad plan is gone (pruned), rebuild the failing case from where the code threw, and say in the golden's note that it was reconstructed. The red run still lands on disk before the fix.
- **Rails never throw; they resolve in a fixed order and record which rung resolved.** A date/time rule tries: full ISO untouched → explicit date → start of a range → a phrase ("next week") → the derived default — and writes down which one it used, so a wrong answer is traceable.
- **A thinking-lane failure on a self-contained request keeps the words, queues a retry, and says what failed** — never a clarifying question back to the owner.
- **Cross-vendor review earns its seat.** A reviewer from the other model family corrected the plan's causal story and caught a state-loss bug that the full diff read had missed.

## Reality check

Green tests are not "done". The last step of every slice is the live surface: the real chat message, the real report on the phone, the real dashboard in a fresh browser — captured as evidence, not asserted.
