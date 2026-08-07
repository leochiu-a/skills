---
name: implement
description: Entry point for implementing a feature or fixing a bug — confirms AC via /grill-me-ac, then executes the red-green loop via /tdd, then reports back against the confirmed AC. Use when the developer asks to implement, build, or fix something that isn't a trivial one-line change.
---

# Implement — thin orchestrator

Does exactly three things: confirm AC → hand off to TDD → report against AC. It doesn't redefine any logic itself — that lives in `/grill-me-ac` and `/tdd`.

## 1. Confirm AC

Run `/grill-me-ac` to get a confirmed AC list (machine and human buckets).

- If the developer already gave an explicit, unambiguous AC up front, `/grill-me-ac` should go straight into confirm mode rather than forcing extra rounds of questions.
- Don't move to the next step until AC is confirmed.

## 2. Run TDD

Once the `machine` AC list is confirmed, run `/tdd` and pass the list in. Don't re-describe the red-green rules here — that's `/tdd`'s responsibility.

## 3. Report back

Once `/tdd` finishes, compile the results:

```
✅ machine AC: <item by item, with test/typecheck results>
⚠️ human AC: <item by item, what the developer needs to confirm, with links/screenshots>
```

Log the full AC (especially the `human` bucket) into the PR description or ticket, so the next round or the next person to pick this up doesn't have to ask again.

## Boundaries

- No plan document, no design review — the plan is one sentence, stated in the message before calling `/grill-me-ac`.
- Trivial one-line fixes, typos, and other obviously non-controversial changes skip this whole flow — just do them.
