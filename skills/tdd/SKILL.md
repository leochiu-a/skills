---
name: tdd
description: Implements a feature or fix using the red-green-refactor loop, one vertical slice at a time, testing at public seams instead of internals. Requires a confirmed AC (machine-verifiable list) as input — if AC isn't confirmed yet, run /grill-me-ac first rather than guessing. Use when the developer asks to "implement with TDD", or when /implement hands off after AC is confirmed.
---

# TDD — red, green, one seam at a time

## Precondition

Needs a `machine` AC list to work from (see `/grill-me-ac`'s output format). If none was supplied, ask for it or run `/grill-me-ac` — don't invent AC on your own.

## 1. Find the seam

Before writing a test, identify the public interface this change exposes and which seam (observable boundary) each `machine` AC item maps to. Tests should read like a spec of behavior through that seam, not an inspection of how it's implemented internally — that's what lets them survive a later refactor.

## 2. Red → Green, one slice at a time

Work through the `machine` AC list one item at a time. Do **not** write all the tests first and then implement everything (horizontal slicing) — each vertical slice finishes before the next starts:

1. **Red**: write one failing test at the seam for one AC item
2. **Green**: write the minimum implementation that makes it pass — no more
3. Repeat for the next AC item until the `machine` list is fully covered
4. **Refactor** only after all AC items are green, as a separate pass — not interleaved with red/green

## Anti-patterns to avoid

- **Implementation coupling** — mocking internal collaborators, testing private methods
- **Tautological assertions** — expected value computed with the same logic as the code under test, so it can't fail
- **Horizontal slicing** — writing every test up front, then implementing all of them together

## Report against AC, not "done"

```
✅ machine AC: <item> — test passes, typecheck clean
```

If a human AC item needs a look (UX, copy, business logic), flag it rather than marking it done — that's `/grill-me-ac`'s or `/implement`'s job to collect and hand to the developer.
