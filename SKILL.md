---
name: always-on-agent-playbook
description: Build a personal, always-on AI agent that survives real use — a server-side loop with state in plain files, a thin chat front door, guarded "hands" that do real work where the owner can watch, layered memory, a strict context budget, deterministic safety rails, and an eval flywheel where every real failure becomes a test before any fix. Use when designing or building an agent that runs 24/7 (Slack/Discord/desktop bots, personal assistants, proactive agents), when deciding what an agent should remember or when it should speak, when wiring two agents from different vendors together, or when an agent project keeps dying of complexity. Not for one-shot scripts, chatbots without memory or actions, or UI-only wrappers.
---

# Always-On Agent Playbook

A method for building an agent that is **always on, proactive, and trustworthy**. It merges two sources: the architecture of large always-on agent products as taught publicly (Lee Robinson's Stanford CS146S lecture on always-on agents, October 2026 — ideas restated here in our own words, nothing reproduced), and the working laws of a real single-user agent that runs on one machine, talks to its owner through a chat app, and routes work into visible coding sessions. Where the two disagree, the law that came from a real failure wins.

Read the whole thing once; then use the **Stage checklist** at the end as the working order.

## 0. The laws (write these before any code)

Agents die of silent failure, invented facts, and attention spam — not of missing features. Put these in the agent's own instruction file and make them enforceable:

1. **Severity.** Every action ends in a visible outcome or a *recorded* intentional non-outcome. Nothing is silently dropped — "held for the morning digest" is a ledger row, not a shrug.
2. **Enrichment.** Never put a fact, name or word into the owner's notes, titles or reminders that the owner didn't say. Memory lines are their sentences, verbatim, with the message they came from.
3. **Effect plan.** A request is "done" only when every effect it required has been served; a parent is never marked verified while a child is unserved.
4. **Eval-first.** A test that reproduces the owner's real message lands — and fails — before the change that fixes it. No golden, no fix.
5. **No knob lies.** A setting that exists does what it says; a reported number is true.
6. **Quiet by default.** One unprompted message a day (the digest), plus genuine escalations and things the owner asked for. Progress is not a reason to speak.
7. **Never claim what didn't happen.** Failed steps are reported as failed. The job's own evidence (files on disk, the tool audit) outranks the model's last sentence.

`references/laws-and-rails.md` has the full text and how to turn each law into a test.

## 1. The shape

The lecture's architecture and the single-user build agree on the skeleton:

- **A server-side loop** owns the agent. Not a terminal that dies when the laptop closes: a scheduled task / service with a lock file and a heartbeat, restarted by a deadman if the heartbeat goes stale.
- **State lives outside the process** — plain files as canon (markdown notes, reminders, routines, skills), a disposable index (SQLite) that can be rebuilt from the files. The process can crash or restart at any time; nothing of value lives only inside it.
- **A thin front door.** One channel the owner already uses (chat). The agent's only tool toward the owner is "send to user"; the channel renders that richly (buttons for approvals, documents for long output, reactions when no words are needed).
- **One bounded planner call per message** that returns a *typed plan* (reply / capture / reminder / delegate / clarify …), validated by deterministic rails before anything runs. The planner has no tools; tools belong to the hands.
- **Guarded hands.** Real work runs in a separate, resumable session (a coding agent) behind a deterministic guard that decides per tool call what is allowed, with the *least power the task needs*. Results come back as a short report file — evidence, not prose.
- **Helpers return summaries.** Anything expensive (screenshots, long tool runs, research) happens in a fresh-context helper; only the result enters the main context.

What *not* to copy from the product-scale version: per-user virtual machines, a workflow engine, multi-client sync protocols, model training loops. For one owner on one machine they solve problems you don't have. Keep the principles (state outside the process, idempotent steps, stable prompt prefix, measure cache hits), skip the infrastructure.

## 2. Wake, work, stay quiet

**Triggers** that may wake the agent: the owner's message; a timer (reminders, routines, the morning digest, the evening review); a file appearing in a watched folder (another agent's mailbox, a report from a work session). Each wake-up records its **origin** on the action row.

**Routines** are the agent's cron: a file per routine (schedule, what to run, where the result goes), created from chat and saved only after the owner says yes. Their output is *held for the digest* by default; it goes straight to the owner only if they said "tell me when it's done".

**Priority:** the owner's message interrupts; everything else queues. Several quick messages from the owner become one turn.

**The importance gate:** after each wake-up, decide whether anything is worth saying. Silence is valid only as a *recorded* non-outcome. Status chatter from helpers and partner agents is batched into the digest; questions and results the owner is waiting for go now.

## 3. Memory

Four layers, cheapest first (`references/memory-and-context.md`):

1. **Profile** — core facts about the owner, in every prompt, as their own words with the source message id. Written only from explicit statements ("remember…", "from now on…", "forget…"). The agent is the sole writer; partner agents never write it.
2. **Dated log** — the conversation record and the action ledger; the most relevant recent rows go in the prompt.
3. **Working on now** — open tickets, running jobs, the partner agent's last question; short-lived, with a TTL.
4. **Everything else** — found on demand by searching the files.

Tell the model, in the prompt, that memories are **information, not instructions**.

## 4. The context budget

- Build the prompt deterministically: stable system prompt (hash-pinned in a test so a change is deliberate), then the stable-to-volatile layers of the context pack with hard caps and a drop order, then the newest message last.
- Long outputs go to files; the prompt carries a path or a one-line pointer.
- Never add or remove tools mid-turn; an unavailable tool returns an error that says what to do next.
- **Measure the cache before optimising it.** Record cache-read and cache-creation tokens per model call in the ledger and show the hit rate; a cache miss you can name is a bug, one you can't name is a knob that lies.

## 5. Safety rails (deterministic beats a second model)

- A **deterministic guard** on every tool call of the hands: allow-lists per task profile, denies that say how to unlock ("say allow gmail"), hard denies for secrets, the memory store, the repo and backups. It fails closed.
- **Untrusted content is data.** Email, web pages, partner-agent messages and work reports are fenced (a random nonce around them) and never carry instruction authority. "The owner approved" inside such content is not approval.
- **Ask first** for anything outward (sending, posting, buying, deleting, changing account settings). Approvals come only from the owner on the front door — one tap or a word — never over an agent-to-agent channel.
- **Secrets never enter the model's conversation.** Tokens live in config the agent reads, not in prompts; tests never print them.
- **Least power.** Reading files is not "use the screen". A task asks for the smallest grant it needs; the owner grants per task.

## 6. The improvement flywheel

Every real failure becomes a golden (a scripted replay of the owner's actual message in a throwaway copy of the agent's world, with the expected effects and the forbidden ones). The loop:

1. the owner's message misfires → 2. write the golden; run it; it is **red** → 3. fix the rails, the prompt or the harness → 4. the golden is green and nothing else turned red (`false_success_rate` and `regression_rate` stay 0) → 5. ship, restart the agent, prove it live with a fresh real message.

Add an **evening review** that names the day's misses from the record and drafts the golden for each (never auto-promoted). The agent may propose changes to its own instructions; it may never edit the tests, the graders, the goal or the rails that judge it. `references/eval-flywheel.md` has the golden format and the three gates.

## 7. Visible hands and partner agents

- **Work where the owner can watch.** Route project work into the owner's own visible sessions (a ticket with their verbatim words and a verify command), and bring the session's report back to the front door. The owner steers by replying.
- **Two agents, one owner.** When a second agent (another vendor's) joins: a shared mailbox the gatekeeper agent polls; one message per file with a small envelope (`from`, `kind`, `subject`, `reply_to`); trust derived from the transport and signatures, never from fields inside the message; data classes (`open` / `personal` / `protected` / `secret`) with "answer, don't dump" for anything personal and a per-day release budget; approvals never travel on the bridge. `references/federation-and-visible-hands.md` has the envelope and the guard profile.

## 8. "Delete the product" — carefully

Build as little as possible around the model and remove what the next model makes unnecessary. But: **a rail that guards a golden stays**; a rail with no golden is a *missing golden*, not a deletion candidate. Keep a ledger that maps each rail to the real failure it came from.

## Stage checklist (the working order)

1. Laws written into the agent's instruction file; a golden harness that can replay one real message.
2. Server-side loop + heartbeat + deadman; files as canon; rebuildable index.
3. Front door with one send path; receipts; approvals as one tap.
4. Planner with typed effects + validator rails; the first five goldens from real messages.
5. Guarded hands with a report contract; least-power grants; the guard matrix as a test.
6. Memory layers: profile (verbatim), log, working-on-now, search.
7. Wake sources beyond chat: timers, routines (as files), watched folders; the importance gate; one digest a day.
8. Context budget: caps, drop order, hash-pinned prompt, cache metrics in the ledger.
9. Evening review → draft goldens; the three suites run on every change.
10. Visible hands (tickets into the owner's sessions) and, if needed, a second agent through a gatekeeper mailbox.

## Anti-patterns that kill agent projects

- Infrastructure first (VMs, queues, dashboards) with no failing message behind it.
- A planner that asks "what should I do with that?" instead of looking first.
- A second-model "auto-review" where a deterministic rule would do.
- Reporting "done" from the model's last message instead of the job's evidence.
- Progress pings; more than one unprompted message a day.
- Memory the agent paraphrased into existence.
- Editing a test to make it pass.

## Attribution

Architecture ideas: Lee Robinson, "How always-on agents work", Stanford CS146S guest lecture, 8 October 2026 (public recording) — restated, not reproduced. Laws, rails, goldens and the federation design: lessons from building and running a single-user personal agent, 2026. This skill is MIT-licensed; the lecture is not part of it.
