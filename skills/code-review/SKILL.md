---
name: code-review
description: Reviews the diff between HEAD and a fixed point along three independent axes — AC (does it do what the confirmed AC asked?), Standards (does it follow this repo's conventions?), and Green trust (is the green build earned, or was it engineered?). Runs at most two sub-agents — AC and Standards share one context, Green trust always gets its own — and reports every axis side by side without merging. Use when the developer asks to review a branch, a PR, or work in progress, or when /implement hands off after the TDD loop finishes.
---

# Code review — three axes, never merged

Reviews `git diff <fixed-point>...HEAD` along three axes that fail independently:

- **AC** — does the diff implement the confirmed AC, no less and no more?
- **Standards** — does it conform to how this repo writes code?
- **Green trust** — is the green build earned by the implementation, or manufactured by editing the tests?

Three axes, but **never more than two sub-agents**. AC and Standards share one context: both judge the diff against a fixed written reference, and Standards *needs* the AC anyway — the Speculative Generality smell is defined against it. Green trust always gets a context of its own, because its stance ("assume the green build is a lie") cannot survive next to the account of why the change is fine. Step 4 decides whether either sub-agent is needed at all.

This skill only pins the inputs and aggregates the outputs — it does not judge the diff itself.

**If this skill is running in the context that wrote the code** (e.g. `/implement` called it right after `/tdd`), that context is a witness, not the reviewer. Its job here is strictly mechanical: resolve the ref, derive the diff, spawn the sub-agents. It does not get to explain the diff to them, and it may not run any axis inline.

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

Three-dot, so the comparison is against the merge-base. A bad ref or an empty diff fails here, not inside a sub-agent.

## 2. Find the AC

In this order:

1. The confirmed AC from this session (if `/implement` or `/grill-me-ac` just ran, it's already in context). The AC is the only thing inherited from the implementing context — it's an input to the work, written before the code existed. Everything the review judges is re-derived from `git`, never taken from the implementer's account of what it did.
2. The PR description or ticket — `/grill-me-ac` logs the AC there precisely so this step works across sessions.
3. A path the developer passed in.

If none is found, ask. If the developer says there is no AC, the AC half of the first sub-agent skips and reports "no AC available" — it does **not** reverse-engineer the AC from the diff, which would make the axis tautological. Standards and Green trust still run.

## 3. Find the standards sources

Whatever this repo documents about how code should be written — `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md`, an ESLint config with hand-written rules.

**Pass only the guideline files whose scope the changed paths actually touch.** A `server/`-only diff has no use for the SCSS, component, or design-token guidelines, and irrelevant guidelines are most of this axis's token bill. Always include the repo-root ones (`AGENTS.md`, `CLAUDE.md`, root ESLint), and include anything you're unsure about. This trims the *reference material*, never the diff under review — the rule against narrowing scope in step 5 still binds.

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

## 4. Scope the run

Two sub-agents is the ceiling, not the default. Decide from `--stat` and `--name-only` alone — never by reading the full diff, which is the isolation this skill exists to buy.

**Does Green trust run?** It has something to audit iff the diff touches either:

- a test file — added, modified, renamed, or deleted (check `git diff --diff-filter=D --name-only` too), or
- a config that can suppress a failure — test runner, coverage threshold, `tsconfig`, lint config, CI workflow.

Neither present → skip it and say so in the report: "no test or config churn to audit". Otherwise it runs, and **always as its own sub-agent**. By the time you reach this skill you have usually read the PR description or the implementer's account; that is exactly the justification this axis must never be handed.

**Does the AC + Standards sub-agent run, or go inline?** It is checklist work against a fixed reference with no stake in the verdict, so the deciding factor is whether the diff crowds out judgement:

- The calling context wrote the code → sub-agent, always. It is a witness, not the reviewer.
- Large diff — roughly 500+ changed lines or 15+ files, a rule of thumb rather than a cutoff → sub-agent.
- Otherwise → inline, in one pass, still bound by the separate-reports rule in step 5 and the no-merge rule in step 6.

So: a two-file fix with one test touched costs one sub-agent; a docs-only change costs none; a large port costs two, and earns them.

## 5. Spawn

One message, one `Agent` call per sub-agent step 4 selected (`general-purpose`). Each prompt carries the diff command and the commit list, plus the brief below.

Rules that bind every prompt:

- **Commands and file paths, not narrative.** Give each sub-agent the `git` commands to run and the files to read. Never a summary of what the change does, what problem it solved, or which parts are "fine" — a sub-agent told where to look stops looking elsewhere.
- **No justification, pre-emptive or otherwise.** If a test change has a defence, the sub-agent is the one to find it in the diff and the AC. Supplying it up front converts the axis from an audit into a confirmation.
- **Derive scope mechanically.** Test paths for the Green trust axis come from `git diff --name-only` filtered by the repo's test convention, not from a hand-picked list. A narrowed glob is indistinguishable from a hidden finding.

### Sub-agent 1 — AC + Standards

Carries: the confirmed AC list verbatim, the trimmed standards sources from step 3, and **the smell baseline pasted in full** (the sub-agent has no other access to it).

Two axes in one context is a token saving, not a licence to blend them. Bind it with all three of these:

- **Two separate reports, ~400 words each.** Return them under `### AC` and `### Standards`. Never one combined list.
- **Standards first, AC second.** Judge the code against the conventions *before* reading what the AC blessed, so the AC's approval can't retroactively excuse a smell.
- **No cross-suppression.** If the AC asked for something that is also a smell, report it on both axes. "The AC wanted it" is not a Standards defence, and "it's ugly" is not an AC finding.

> **AC.** Report: (a) `machine` AC items that are missing or only partially implemented; (b) behaviour in the diff that no AC item asked for (scope creep); (c) items that look implemented but where the implementation looks wrong. Quote the AC item for each finding. Say nothing about `human` AC items — those are for the developer to judge, not you.
>
> **Standards.** Report, per file or hunk: (a) every place the diff violates a documented standard — cite the standard, file and rule; (b) any baseline smell you spot — name it and quote the hunk. Distinguish hard violations from judgement calls: a documented-standard breach can be hard, a baseline smell never is, and a documented repo standard overrides the baseline. Skip anything tooling enforces.

### Sub-agent 2 — Green trust

Carries: the test-file diff specifically (`git diff <fixed-point>...HEAD -- <test paths>`), the confirmed AC, and the per-file `--stat`. Nothing else — in particular, not the PR description's account of why the tests look the way they do.

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

A missing AC never disables this axis — it needs no AC to function. Only the churn gate in step 4 can skip it.

## 6. Aggregate

Present all three reports verbatim, or lightly cleaned, under `## AC`, `## Standards`, and `## Green trust` — three headings even when two axes came back from one sub-agent. Do **not** merge, rerank, or dedupe across axes.

End with one line per axis: finding count, and the worst finding within that axis. No single verdict across axes — that blend is exactly what the separation exists to prevent.

---

The two-axis structure and the Fowler smell baseline are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see `NOTICE`). Standards is the original axis, AC is the original Spec axis re-pointed at the confirmed AC list, and Green trust is this repo's own.
