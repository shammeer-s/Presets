---
name: skulblakas-ven
description: "Skulblaka's ven: like the rider's spell for seeing through the dragon's eyes, reach one shared vision with the user on a plan, design, or decision by interviewing them in rounds until both see the same thing, before anything is built. Use whenever the user says skulblakas ven, asks to share vision or get aligned, wants a plan stress-tested, or is about to build something whose key decisions are still unsettled."
---

Interview the user relentlessly until you reach a shared understanding. Map this as a design tree: every decision branches into the decisions that hang off it.

Work the tree in rounds. The frontier is every decision whose prerequisites are already settled: the questions you can ask now without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a later round, not this one.

Finding facts is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The decisions are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

## Write the vision down

Once the user confirms, write the session to a document instead of leaving it in the conversation. Do not interview further at this stage; only synthesize what was already settled.

Save it as `docs/vision/<topic-slug>.md` if the repo has a `docs/` folder, otherwise ask the user where it should live.

<vision-template>

# <Topic>

## Starting Point

What the user brought to this session — the plan, decision, or idea being grilled.

## Settled Decisions

Every decision reached, in the order the tree was walked. For each:

- **Decision**: what was decided
- **Reason**: why, including any fact a sub-agent found that shaped it

## Parked Items

Anything raised but deliberately left open, and why it was parked rather than settled.

## Open Questions for Next Session

Anything the frontier couldn't reach yet because it depends on work outside this conversation (e.g. a decision blocked on an external answer).

</vision-template>

This document is an input to `diffindo`, not a replacement for it: it records what was agreed, not tickets, ordering, or acceptance criteria. Once saved, tell the user the vision doc is ready to be cut into tickets with `diffindo`.
