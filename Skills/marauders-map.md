---
name: marauders-map
description: "Marauder's Map: shows the whole layout at once, but only reveals detail on the room you're standing in. Plans multi-phase work as one lightweight master roadmap up front, then opens each phase's full detailed plan only when that phase actually starts. Use when a project is too large to grill or ticket in one sitting, when the user wants a roadmap broken into phases and subphases, or mentions phase planning, a project roadmap, or multi-session work."
---

# Marauder's Map

A map for work too large for one `skulblakas-ven` session. The master pass draws the whole layout, low detail; it never reveals a single phase's full detail — that detail is earned later, one room at a time, when that phase is actually about to be built.

## 1. Master plan (once, at project start)

Interview the user using the same round/frontier format as `skulblakas-ven` — number each question, give a recommendation, wait for answers — but the frontier here stops at phase boundaries. Never let a question about one phase's internal implementation into this pass; if a question can only be answered by deciding something inside a phase, park it for that phase's own session instead of answering it now.

What this pass resolves:
- What are the phases, in what order?
- What does each phase depend on finishing first?
- What's true across every phase (constraints, non-negotiables) that shouldn't be re-asked per phase?

Write the result to `docs/roadmap/<project-slug>.md`:

<roadmap-template>

# Roadmap: <Project>

## Phases

| # | Phase | Scope (one line) | Depends on | Status |
|---|---|---|---|---|
| 1 | <name> | <one-line scope> | none | Not started |
| 2 | <name> | <one-line scope> | Phase 1 | Not started |

## Project-wide constraints

Decisions from the master pass that apply to every phase, so they aren't re-asked per phase.

## Parked (belongs to a future phase's own session)

Anything raised during the master pass that only makes sense to decide once that phase starts.

</roadmap-template>

Tell the user the roadmap is ready and which phase, if any, is unblocked and ready to start.

## 2. Starting a phase

Trigger: the user says "start phase N," names a phase, or asks what's next.

1. Read `docs/roadmap/<project-slug>.md`. If the phase's dependencies aren't marked Done, say so and ask whether to proceed anyway rather than proceeding silently.
2. Run a full `skulblakas-ven` session scoped only to this phase. Seed it with the phase's one-line scope and the roadmap's project-wide constraints — don't re-ask what the master pass already settled.
3. This produces the normal `skulblakas-ven` output: `docs/roadmap/<phase-slug>/plan.md`.

## 3. Subphases

Once `docs/roadmap/<phase-slug>/plan.md` is confirmed, split its Settled Decisions into subphases before handing anything to `diffindo`. A subphase is a slice of the phase big enough to be worth planning as a unit, but small enough to ship and verify on its own — bigger than a ticket, smaller than the whole phase.

Write `docs/roadmap/<phase-slug>/phases.md`:

<phase-template>

# Phase: <Phase name>

Source: `docs/roadmap/<phase-slug>/plan.md`

## Subphases

### Subphase 1: <name>
Scope: <what this covers>
Decisions covered: <which settled decisions from plan.md belong here>

### Subphase 2: <name>
...

</phase-template>

Run `diffindo` once per subphase, not once for the whole phase — each subphase gets its own `docs/roadmap/<phase-slug>/<subphase-slug>-tickets.md`, inside that same phase folder. A phase-wide ticket cut tends to produce tickets too large to verify independently; per-subphase cuts stay small.

## 4. Build

Each subphase's tickets get built the normal way: `arresto-momentum` for the logic, `verdimillious` for anything user-facing.

## 5. Close the loop

When every subphase under a phase has all its tickets done, mark that phase **Done** in `docs/roadmap/<project-slug>.md`. Check whether marking it Done unblocks another phase (its dependency is now satisfied) and tell the user which phase is next.

## Keeping documents in sync

`docs/roadmap/<phase-slug>/` holds four documents that describe the same phase at different levels of detail: `plan.md` (decisions), `phases.md` (subphase grouping), each `<subphase-slug>-tickets.md` (tickets), and the phase's row in the root `docs/roadmap/<project-slug>.md` (status). A change discovered at any level must propagate to every level above and below it, in the same turn it's discovered — never left for the user to notice as a mismatch later:

- A decision in `plan.md` turns out wrong mid-build → update `plan.md` itself, then check whether it changes which subphase a decision belongs to in `phases.md`, and whether any ticket in a `-tickets.md` file now has stale scope or acceptance criteria.
- A subphase gets split, merged, or renamed → update `phases.md`, and rename/move the corresponding `-tickets.md` file to match.
- A ticket's scope changes during `arresto-momentum` or `verdimillious` in a way that changes what the subphase or phase actually covers → update the ticket file, then walk the change up through `phases.md` and, if it affects the phase's one-line scope or its dependencies, up into `docs/roadmap/<project-slug>.md` too.

Never edit one document in this chain and leave the others describing the old version.

## Guardrails

- Never grill a phase to full depth during the master pass — that's what step 2 is for, later.
- If the user asks to start a phase and no roadmap exists yet, offer to run the master pass first rather than starting an ungrounded phase session — proceed without one only if they explicitly say to skip it.
- Don't let subphase count balloon past what a phase actually needs — if every subphase would only produce one ticket, the phase didn't need subphases at all; go straight to `diffindo` on the whole phase.
- The root roadmap file is `docs/roadmap/<project-slug>.md`, sitting directly under `docs/roadmap/`; each phase's own files sit one level deeper in `docs/roadmap/<phase-slug>/`. If a phase's slug would collide with the project's own slug, ask the user to disambiguate before writing anything — the folder and the root file must never resolve to the same name.
