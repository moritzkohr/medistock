## 2026-10-29 - Defining which parts of the code is mine

**Situation:** The agent proposed implementing all eight entities in sequence.
Nothing in CLAUDE.md told it which parts I wanted to write myself, so it would
have written Batch and the FEFO logic too.

**What I did:** Added a "Code I write myself" section to CLAUDE.md with an
explicit list (first entity, first relationship, core business rules, first
controller). For those parts the agent may review, explain, give hints and
write tests around my code, but not implement them.

**Learned:** An agent only respects boundaries it can read. "Interview-relevant
code" would be too vague; a concrete list works. The line "even if it blocks the
next step" matters most, because otherwise the agent fills a gap with its own
implementation to finish its task.