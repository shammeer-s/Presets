---
name: arresto-momentum
description: "Arresto Momentum: slows a falling object to a controlled stop before impact. Test-driven development in small, verified slices — write one failing test, then just enough code to pass it, before moving on. Use when the user wants to build a feature or fix a bug test-first, mentions TDD, red-green, red-green-refactor, or wants a feedback loop before writing implementation code."
---

# Arresto Momentum

Red, then green, one slice at a time. The point is never to write more than one unverified step of code — each test is where momentum gets arrested, catching a mistake at one slice instead of letting it crash into fifty lines of untested code.

## Before the loop: agree the seams

A seam is the public function or method you test through, never the internals behind it. List every seam you intend to test and confirm them before writing anything. You cannot test everything; agreeing seams up front puts the effort on the behaviours that matter, not every internal branch.

Ask: "Here are the seams I'll test — does this match what you want verified?"

## The loop

1. **Red.** Write one test for one behaviour. Run it. It must fail — if it passes, you tested nothing new.
2. **Green.** Write the smallest amount of code that makes it pass. No extra branches, no features the test didn't ask for.
3. **Next slice.** Pick the next behaviour and repeat.

Refactoring is not part of this loop. Clean up in a separate pass once a group of slices is green, never mid-cycle — mixing "make it pass" with "make it nice" is how a two-line change becomes an untested fifty-line change.

Never write a batch of tests before any implementation. Each test is a tracer bullet: it should respond to what the last slice taught you, not to a plan made before any code existed.

## What makes a test worth keeping

A good test asserts the *result* a caller gets, using only the public interface, and its expected value comes from an independent source — a literal, a worked example, a known-good figure — never recomputed the way the code computes it.

```python
def test_chart_slide_costs_two_units():
    assert generation_units(Manifest(kind="chart")) == 2
```

Two failure patterns to catch before they're written:

- **Coupled to internals.** Asserting a mock was called, testing a private method, or querying the database directly instead of going through the interface. The tell: it breaks on a refactor even though behaviour didn't change.
- **Tautological.** The expected value is computed the same way the code computes it (`UNIT_WEIGHTS[m.kind]` compared against itself), so it passes no matter what the table contains.

## Mocking

Fake only what you don't control: an LLM call, an external API, the clock, randomness. Never fake your own code.

Pass dependencies in rather than constructing them inside the function — that's what makes faking possible at all. Prefer one specific method per external operation over one generic call with branching inside.

```python
class FakeLLM:
    def __init__(self, manifest):
        self.manifest = manifest

    def generate_manifest(self, prompt, context):
        return self.manifest
```

Never assert against a live model call inside this loop — a nondeterministic response makes red/green meaningless. Test your own code around a fake; judge real model output separately, outside TDD.

## Stopping condition

The loop is done when every seam agreed at the start is covered by a passing test and no test was written without first seeing it fail.
