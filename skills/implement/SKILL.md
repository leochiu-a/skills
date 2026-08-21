---
name: implement
description: Entry point for implementing a feature or fixing a bug — confirms AC via /grill-me-ac, executes the red-green loop via /tdd, reviews the diff via /code-review, then reports back against the confirmed AC. Use when the developer asks to implement, build, or fix something that isn't a trivial one-line change.
---

# Implement — thin orchestrator

Does exactly four things: confirm AC → hand off to TDD → review the diff → report against AC. It doesn't redefine any logic itself — that lives in `/grill-me-ac`, `/tdd`, and `/code-review`.

## 1. Confirm AC

Run `/grill-me-ac` to get a confirmed AC list (machine and human buckets).

- If the developer already gave an explicit, unambiguous AC up front, `/grill-me-ac` should go straight into confirm mode rather than forcing extra rounds of questions.
- Don't move to the next step until AC is confirmed.

## 2. Run TDD

Once the `machine` AC list is confirmed, run `/tdd` and pass the list in. Don't re-describe the red-green rules here — that's `/tdd`'s responsibility.

## 3. Review the diff

Once `/tdd` reports green, run `/code-review` against the point the branch started from. The green build is `/tdd`'s own claim about its own work — this is the independent check on it, and the `Green trust` axis is the one that matters most here.

**The reviewer must not be the implementer.** Whatever context ran `/tdd` wrote the code, and knows why every compromise in it seemed reasonable at the time — that context cannot audit its own diff, and it must not be the one framing the audit either. So:

- Hand `/code-review` only the fixed point and the confirmed AC. Not a summary of what was built, not which files were touched, and above all no pre-emptive justification of any test change — "the snapshot update was intentional" is exactly the sentence that blinds the `Green trust` axis.
- The review derives everything else from `git` itself. If a test edit was legitimate, the diff and the AC have to carry that on their own.

Findings on the `AC` or `Green trust` axes go back into `/tdd` as another slice, not into the report as a caveat. Standards findings are judgement calls — surface them, don't auto-fix.

## 4. Report back

Once the review comes back clean, compile the results:

```
✅ machine AC: <item by item, with test/typecheck results>
⚠️ human AC: <item by item, what the developer needs to confirm, with links/screenshots>
```

Log the full AC (especially the `human` bucket) into the PR description or ticket, so the next round or the next person to pick this up doesn't have to ask again.

## Boundaries

- No plan document, no design review — the plan is one sentence, stated in the message before calling `/grill-me-ac`.
- Trivial one-line fixes, typos, and other obviously non-controversial changes skip this whole flow — just do them.
