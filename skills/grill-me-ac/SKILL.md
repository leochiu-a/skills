---
name: grill-me-ac
description: Grills the developer with the minimum number of questions needed to pin down acceptance criteria before any code is written, split into machine-verifiable and human-judgment buckets. Runs as step 1 of /implement, and standalone when the developer wants the AC settled and agreed — with a teammate, a PM, a ticket — before implementation starts. An ordinary request to build or fix something is not that case: that's /implement, which calls this skill itself. Skip for trivial one-line fixes.
---

# Grill Me — AC

The goal isn't a document, it's pinning down AC to the point where work can start, with the fewest possible round trips. Never ask "what's the AC?" with an empty hand — read the request, draft your own AC first, and only ask about the parts you can't judge yourself.

## Flow

1. **Draft first**: from the developer's original request, write a first-pass AC list yourself, split into two buckets:
   ```
   AC (machine-verifiable): concrete, checkable conditions — test scenarios, typecheck, does the build pass
   AC (human-judgment): things that need a human look — UX, copy, business logic, risk
   ```
2. **Only ask about genuine ambiguity**: if the draft has items you can't judge or that are ambiguous, list all the questions at once and ask in a single pass — don't trickle one question per turn. Anything already clear goes straight into the draft, no question needed.
3. **Converge**: after getting answers, update the AC list and ask one final question: "Does this AC look right, or does it need adjusting?"
4. **Stopping condition**: AC is confirmed once there's no objection, or after two rounds of discussion produce no new information — don't loop indefinitely. If the scope genuinely can't be pinned down, say so directly ("this task's scope is unclear, need a decision between X or Y") instead of guessing an AC to move forward.

## Output (hand this to whoever picks up next)

Once confirmed, hand it off in this format so `/tdd` or any downstream step can read it directly:

```
## Confirmed AC
machine:
- ...
human:
- ...
```

Also log this AC into the PR description or ticket — whoever picks this up later can see what the acceptance bar was, without re-asking.
