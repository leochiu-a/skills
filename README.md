# skills

The [Claude Code](https://claude.com/claude-code) skills I use myself. They all revolve around the same idea: **pin down the acceptance criteria (AC) before writing any code, work through them one slice at a time with TDD, then review the diff independently and report back against the AC**.

## Why: Loop Engineering

This whole set exists to make **Loop Engineering** concrete. The unit of work isn't a prompt and a patch — it's a closed loop that an agent can run and that a human can trust the output of:

**Spec → Implement → Verify → Report → back to Spec.**

`grill-me-ac` closes the front of the loop by turning a vague request into criteria that can actually be checked. `tdd` walks those criteria one slice at a time, so every pass through the loop is small enough to reason about. `code-review` closes the back of the loop with an independent check, because a loop that grades its own output drifts. And the report lands back on the same AC the loop started from, so the next iteration begins where the last one measurably ended.

Every design choice below follows from that: the loop has to be able to run unattended, it has to be honest about what it couldn't verify by itself, and its default entry point should be the whole loop rather than one of its steps — with the AC gate available up front when you want to settle the spec before the loop runs.

## Skills

One skill is the way in; the rest are the discipline it runs on. Two of them are also useful on their own.

### The way in

| Skill | What it does |
| --- | --- |
| [`implement`](skills/implement/SKILL.md) | A thin orchestrator: confirm the AC → hand off to TDD → review the diff → report against the AC |

For a feature or a bug fix, `/implement` is the only thing you need to type — it runs the whole loop and hands you back a report against the AC.

### The pieces it composes

| Skill | What it does | When it's reached for |
| --- | --- | --- |
| [`grill-me-ac`](skills/grill-me-ac/SKILL.md) | Pins down the AC with the fewest possible round trips, split into machine-verifiable and human-judgment buckets | Step 1 of `/implement` — **or call it yourself first** (see below) |
| [`tdd`](skills/tdd/SKILL.md) | Red-green-refactor, one vertical slice at a time, testing at the public seam rather than internals | Step 2 of `/implement`, once the AC is confirmed |
| [`code-review`](skills/code-review/SKILL.md) | Reviews the diff along three independent axes: AC fidelity, repo conventions, and whether the green is trustworthy. Three sub-agents run in parallel and their results are never merged | Step 3 of `/implement`, or on any branch/PR you want reviewed |

**Don't call `/tdd` yourself.** It needs a confirmed AC to work from and has no reviewer downstream of it — calling it directly gets you the middle of the loop with neither end attached. It's the one piece that only makes sense inside `/implement`.

**`/grill-me-ac` and `/code-review` do stand alone**, at the two ends of the loop:

- Run `/grill-me-ac` on its own when the AC deserves a look before any code exists — pin it down, have it reviewed by whoever needs to sign off, then start `/implement` from the confirmed list instead of letting the loop settle the AC in passing. This is the recommended path for anything where getting the spec wrong is expensive.
- Run `/code-review` on its own against a branch or PR that didn't come out of this loop at all.

### Day to day

Type `/implement` and describe the requirement as precisely as you would in a ticket. That's it — the more precise the request, the more the AC step collapses into a single confirmation instead of a round of questions. The exceptions:

| Situation | What to type |
| --- | --- |
| Everyday feature work or bug fix | `/implement` |
| Trivial one-liner, typo, rename | Nothing — this flow is skipped by design |
| The spec needs sign-off before any code exists | `/grill-me-ac` first, agree on the AC, then `/implement` |
| Reviewing a branch or PR this loop didn't produce | `/code-review` |

## Design trade-offs

- **Two buckets of AC.** Machine-verifiable items (tests, typecheck, build) can be verified automatically; human-judgment items (UX, copy, business logic, risk) can only be flagged for a person to look at. Keeping them apart avoids the misleading "everything is green so it's done" report.
- **Never ask for AC empty-handed.** Draft one first, ask only about the parts you genuinely can't decide, and ask everything at once instead of one question per turn.
- **Vertical slicing.** One AC item goes red → green before moving to the next, rather than writing every test up front and implementing them together. Refactoring is its own separate pass once everything is green.
- **Test the seam, not the internals.** No mocking internal collaborators, no testing private methods — that's what lets the tests survive later refactors.
- **Green has to be verified independently.** When `/tdd` says "it's green," that's its own claim about its own work, and exit codes, CI, and test reports all sit downstream of the tests — when the tests themselves are modified, those signals confirm each other and then get it wrong together. So `code-review` has one axis that asks nothing but: was this green earned by the implementation, or bent out of the tests?
- **The reviewer can't be the author.** The context that just wrote the code knows why every compromise looked reasonable at the time; it can't review its own diff, and it can't be the one framing the review either — so all three axes run in independent sub-agents that receive only the fixed point and the AC, and re-derive everything else from git. The implementer's account of what it did is never taken at face value.
- **The three axes are never merged or reordered.** A change can follow every convention while building the wrong thing, follow the AC exactly while breaking conventions, or be clean and on-spec while its green came from loosened assertions. Collapsing them into one verdict just lets one axis cover for another.

## Credits

The two-axis structure of `code-review` and the Fowler code smell baseline are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see [NOTICE](NOTICE)). The AC axis and the Green trust axis are this repo's own.

## Installation

Install with the [`skills` CLI](https://github.com/vercel-labs/skills) — it interactively lets you pick which skills to install and for which agent:

```bash
npx skills add leochiu-a/skills
```

Install at the user level (`~/.claude/skills/`) instead of just the current project:

```bash
npx skills add leochiu-a/skills --global
```

Install just one of them, skipping all confirmations:

```bash
npx skills add leochiu-a/skills --skill tdd -y
```

See what skills this repo has first:

```bash
npx skills add leochiu-a/skills --list
```

If you'd rather not use the CLI, copying the folders over works too:

```bash
git clone https://github.com/leochiu-a/skills.git && cp -r skills/skills/* ~/.claude/skills/
```

Once installed, type `/implement` in Claude Code — or just describe what you want built and let it trigger itself. Reach for `/grill-me-ac` first when you want the AC settled and signed off before the loop starts, and `/code-review` on its own for a diff the loop didn't produce.
