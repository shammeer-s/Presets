---
name: diffindo
description: "Diffindo: the severing charm, cuts cleanly along a chosen line. Splits a written spec or vision document into small, ordered tickets with explicit blocking dependencies, saved as a single file under docs/. Use when the user wants a spec or plan broken into buildable pieces, mentions to-tickets, or has a spec/vision doc ready to split into tasks."
---

# Diffindo

Cut a spec into tickets small enough to build and verify one at a time, each declaring what blocks it. Do not interview the user here — this skill only synthesizes what a spec or vision document already settled. If no such document exists yet, tell the user to run the grilling/vision skill or `/to-spec` first.

## Process

1. **Read the source document.** The source is `docs/roadmap/<phase-slug>/plan.md`, and, when subphases exist, `docs/roadmap/<phase-slug>/phases.md` for which decisions belong to which subphase. If the user points to a different document, use that instead. Do not proceed on a vague verbal description alone — ask which document to cut if none is given.

If the source is a `skulblakas-ven` plan (`plan.md`), its sections map to this process directly:

- **Settled Decisions** is the only section tickets are cut from.
- **Parked Items** and **Open Questions for Next Session** are explicitly out of scope. Never turn a parked or open item into a ticket — list them under "Deferred" in the output instead (see template) so they stay visible without being treated as ready work.

If cutting one subphase specifically, only pull the decisions `phases.md` lists under that subphase — not the whole plan.

2. **Find the cut lines.** Identify the natural seams in the settled work: each independently testable piece of behaviour, schema, or interface change. Prefer one ticket per seam. A ticket that bundles two unrelated seams, or that bundles a settled decision with a parked one, should be split further.

3. **Order by dependency.** For each ticket, work out what it needs already in place before it can start. A ticket with no unmet dependency is ready now; everything else is blocked. Chain dependencies explicitly — don't leave an implied order unstated.

4. **Write one ticket file per subphase.** `docs/roadmap/<phase-slug>/<subphase-slug>-tickets.md`, inside that phase's own folder, using the template below. If the phase has no subphases (small enough to cut directly), write `docs/roadmap/<phase-slug>/tickets.md` instead.

<tickets-template>

# Tickets: <Subphase or Phase name>

Source: <path to plan.md, and phases.md if subphase-scoped>

## Ticket 1: <short title>

**Status**: Ready / Blocked by #<n>

**Scope**: what this ticket covers, in one or two sentences.

**Depends on**: none, or a list of ticket numbers.

**Blocks**: which later tickets need this one done first.

**Acceptance**: how to tell this ticket is actually done — the seam or behaviour that proves it.

## Ticket 2: <short title>

...

## Deferred

Items from the source document's Parked or Open Questions sections. Not tickets — listed here so they stay visible and aren't silently dropped.

</tickets-template>

5. **Check with the user before finalizing.** Show the ticket list and the dependency order. Confirm the split matches how they'd actually want to build it — a ticket that looks small on paper may hide more work, and only the user can catch that.

## What makes a good split

- Each ticket should be buildable and testable on its own, without needing to guess at a ticket that comes later.
- Prefer few, clearly-bounded tickets over many small fragments — the ideal is the smallest number of tickets that still lets you verify each one independently.
- Never hide a real dependency inside prose. If ticket 3 needs ticket 1's schema, say so under "Depends on," not just in a sentence buried in the scope.
- If a ticket's acceptance criteria can't be stated concretely, the ticket is still too vague to hand off — go back to the source spec and cut differently.

## Stopping condition

Done when every seam from the source document has a ticket, every ticket has a stated dependency (or none), and the user has confirmed the split and order.

## Keeping documents in sync

If cutting tickets surfaces a problem with `plan.md` or `phases.md` — a decision that doesn't actually split cleanly, a subphase boundary that doesn't hold — fix that document too, in the same turn, before finalizing the ticket file. Don't write tickets against a plan you already know is wrong and leave the mismatch for later.
