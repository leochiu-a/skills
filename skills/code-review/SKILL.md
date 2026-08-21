---
name: code-review
description: Reviews the diff between HEAD and a fixed point along three independent axes — AC (does it do what the confirmed AC asked?), Standards (does it follow this repo's conventions?), and Green trust (is the green build earned, or was it engineered?). Runs the axes as parallel sub-agents and reports them side by side without merging. Use when the developer asks to review a branch, a PR, or work in progress, or when /implement hands off after the TDD loop finishes.
---

# Code review — three axes, never merged

Reviews `git diff <fixed-point>...HEAD` along three axes that fail independently:

- **AC** — does the diff implement the confirmed AC, no less and no more?
- **Standards** — does it conform to how this repo writes code?
- **Green trust** — is the green build earned by the implementation, or manufactured by editing the tests?

Each axis runs as its own sub-agent, for two separate reasons: so the axes don't pollute each other's context, and so no axis is judged by a context that has a stake in the verdict. This skill only pins the inputs and aggregates the outputs — it does not judge the diff itself.

**If this skill is running in the context that wrote the code** (e.g. `/implement` called it right after `/tdd`), that context is a witness, not the reviewer. Its job here is strictly mechanical: resolve the ref, derive the diff, spawn the axes. It does not get to explain the diff to them.

## Precondition

Needs a **fixed point** and, for the AC axis, a **confirmed AC list** (see `/grill-me-ac`'s output format). Neither is guessable — ask rather than invent.

## 1. Pin the fixed point

Whatever the developer named is the fixed point — a SHA, a branch, a tag, `main`, `HEAD~5`. If they named none, ask.

Confirm it resolves and the diff isn't empty **before** spawning anything:

```
git rev-parse <fixed-point>
git diff --stat <fixed-point>...HEAD
git log <fixed-point>..HEAD --oneline
```

Three-dot, so the comparison is against the merge-base. A bad ref or an empty diff fails here, not inside three sub-agents.

## 2. Find the AC

In this order:

1. The confirmed AC from this session (if `/implement` or `/grill-me-ac` just ran, it's already in context). The AC is the only thing inherited from the implementing context — it's an input to the work, written before the code existed. Everything the review judges is re-derived from `git`, never taken from the implementer's account of what it did.
2. The PR description or ticket — `/grill-me-ac` logs the AC there precisely so this step works across sessions.
3. A path the developer passed in.

If none is found, ask. If the developer says there is no AC, the AC axis skips and reports "no AC available" — it does **not** reverse-engineer the AC from the diff, which would make the axis tautological.

## 3. Find the standards sources

Whatever this repo documents about how code should be written — `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md`, an ESLint config with hand-written rules.

On top of that, the Standards axis always carries the **smell baseline** below, so it still has something to say in a repo that documents nothing. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation. And skip anything tooling already enforces — a lint rule doesn't need a reviewer.

Each smell reads *what it is* → *how to fix*:

- **Mysterious Name** — a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **Duplicated Code** — the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy** — a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **Data Clumps** — the same few fields or params keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession** — a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches** — the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **Shotgun Surgery** — one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change** — one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality** — abstraction, parameters, or hooks added for needs the AC doesn't have. → delete it; inline back until a real need shows.
- **Message Chains** — long `a.b().c().d()` navigation the caller shouldn't depend on. → hide the walk behind one method on the first object.
- **Middle Man** — a class or function that mostly just delegates onward. → cut it, call the real target direct.
- **Refused Bequest** — a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

## 4. Spawn the axes in parallel

One message, three `Agent` calls (`general-purpose`). Each prompt carries the diff command and the commit list, plus the axis brief below.

Rules that bind all three prompts:

- **Commands and file paths, not narrative.** Give each sub-agent the `git` commands to run and the files to read. Never a summary of what the change does, what problem it solved, or which parts are "fine" — a sub-agent told where to look stops looking elsewhere.
- **No justification, pre-emptive or otherwise.** If a test change has a defence, the sub-agent is the one to find it in the diff and the AC. Supplying it up front converts the axis from an audit into a confirmation.
- **Derive scope mechanically.** Test paths for the Green trust axis come from `git diff --name-only` filtered by the repo's test convention, not from a hand-picked list. A narrowed glob is indistinguishable from a hidden finding.


**AC axis** — the confirmed AC list, verbatim.

> Report: (a) `machine` AC items that are missing or only partially implemented; (b) behaviour in the diff that no AC item asked for (scope creep); (c) items that look implemented but where the implementation looks wrong. Quote the AC item for each finding. Say nothing about `human` AC items — those are for the developer to judge, not you. Under 400 words.

**Standards axis** — the standards-source files found in step 3, **plus the smell baseline pasted in full** (the sub-agent has no other access to it).

> Report, per file or hunk: (a) every place the diff violates a documented standard — cite the standard, file and rule; (b) any baseline smell you spot — name it and quote the hunk. Distinguish hard violations from judgement calls: a documented-standard breach can be hard, a baseline smell never is, and a documented repo standard overrides the baseline. Skip anything tooling enforces. Under 400 words.

**Green trust axis** — the test-file diff specifically (`git diff <fixed-point>...HEAD -- <test paths>`), plus the confirmed AC, plus the per-file `--stat`.

> Assume the green build is a lie until the diff proves otherwise. Your question is not "is this code good" — it is "did the implementation earn this green, or did the tests get bent to fit the implementation?" Report every instance of:
> - a test deleted, renamed into irrelevance, `skip`ped, `only`d, or commented out
> - an assertion weakened — a narrower matcher swapped for a looser one, an exact value replaced by `any`/`expect.anything()`/a truthiness check, a removed assertion inside a kept test
> - an expected value edited to match what the code now returns, rather than what the AC says it should be
> - a snapshot updated with no corresponding intentional behaviour change
> - a tautological assertion — the expected value computed by the same logic as the code under test, so the test cannot fail
> - hardcoded returns, special-cased inputs, or branches in the implementation that exist only to satisfy a specific test fixture
> - a timeout, retry, or `waitFor` widened to make a flaky test pass
> - config-level suppression: coverage thresholds lowered, a test path excluded, a strict compiler or lint flag relaxed
>
> Quote the hunk for each. Flag the ratio when the test diff is large relative to the implementation diff — a change that edits mostly tests deserves suspicion. A legitimate test change is one the AC or an intentional behaviour change justifies; say which justification you accepted and why. Under 400 words.

If there's no AC, skip the AC axis and note it in the report. Never skip Green trust — it needs no AC to function.

## 5. Aggregate

Present the three reports verbatim, or lightly cleaned, under `## AC`, `## Standards`, and `## Green trust`. Do **not** merge, rerank, or dedupe across axes.

End with one line per axis: finding count, and the worst finding within that axis. No single verdict across axes — that blend is exactly what the separation exists to prevent.

## Why three axes

A change can pass any one of them and fail another:

- Follows every convention, implements the wrong thing → **Standards pass, AC fail.**
- Does exactly what the AC asked, ignores the repo's conventions → **AC pass, Standards fail.**
- Clean code, faithful to the AC, and green only because an assertion got loosened → **AC and Standards pass, Green trust fail.**

Green trust is separate for a structural reason: every other signal in the loop — exit codes, CI, the TDD report — is downstream of the tests. When the tests are the thing that moved, those signals confirm each other and all of them are wrong together. Nothing else in the loop is looking at this.

---

The two-axis structure and the Fowler smell baseline are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see `NOTICE`). The AC axis and the Green trust axis are this repo's.
