# always-on-agent-playbook

A skill for Claude Code (or any agent that reads skills) that teaches how to build a personal AI agent that’s always on and still trustworthy. Something that runs 24/7, remembers what matters, works in the background, and doesn’t spam you or make things up.

It comes from two places. The first is the architecture Lee Robinson laid out in his Stanford CS146S lecture on always-on agents ([October 2026](https://x.com/leerob/status/2108650243365736855?s=20)). His ideas are restated here, not copied. The second is the hard-won lessons of building a single-user agent that runs on one ordinary machine and actually gets used every day. When the lecture and real failures disagreed, the failures won.

What it covers:

- **Seven laws to write before any code:** nothing fails silently, no invented facts in your notes, one unprompted message a day, never report a step that didn’t happen.
- **The shape:** a server-side loop, state kept in plain files, a thin chat front door, and “hands” behind a deterministic guard.
- **Memory in layers,** the cheapest first: a profile in your own words, a dated log, what’s in flight right now, and search for everything else.
- **A context budget** you can measure, plus safety rules that block things outright instead of asking a second model for an opinion.
- **The eval flywheel:** every real miss becomes a failing test before the fix.
- **Visible work:** tasks dropped into your own sessions, so you watch it happen.
- **A gatekeeper for partner agents** from other vendors: answer, don’t dump.

There’s also a 10-stage build order, and a list of the anti-patterns that kill agent projects.

## Install

Copy this folder into your skills directory, for example `~/.claude/skills/always-on-agent-playbook/`. The skill triggers when you're designing or building an always-on or proactive personal agent, deciding what it should remember or when it should speak, or wiring two agents together.

## Contents

- [`SKILL.md`](SKILL.md): the method, the stage checklist, the anti-patterns.
- [`references/laws-and-rails.md`](references/laws-and-rails.md): how each law becomes a test; planner validator rails; the hands guard; the report contract.
- [`references/memory-and-context.md`](references/memory-and-context.md): memory layers, the context pack, cache discipline.
- [`references/eval-flywheel.md`](references/eval-flywheel.md): the golden format, the three gates, the evening review, lessons from real slices.
- [`references/federation-and-visible-hands.md`](references/federation-and-visible-hands.md): tickets into visible sessions; the mailbox, envelope and gatekeeper rules for a partner agent.

MIT licensed. The referenced lecture is not part of this repository.
