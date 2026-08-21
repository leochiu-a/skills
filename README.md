# skills

The [Claude Code](https://claude.com/claude-code) skills I use myself. They all revolve around the same idea: **pin down the acceptance criteria (AC) before writing any code, work through them one slice at a time with TDD, then review the diff independently and report back against the AC**.

## Skills

| Skill | What it does | When it triggers |
| --- | --- | --- |
| [`implement`](skills/implement/SKILL.md) | A thin orchestrator: confirm the AC → hand off to TDD → review the diff → report against the AC | A request to implement a feature or fix a bug that isn't a one-line change |
| [`grill-me-ac`](skills/grill-me-ac/SKILL.md) | Pins down the AC with the fewest possible round trips, split into machine-verifiable and human-judgment buckets | Whenever the AC isn't explicit yet — usable on its own, and also called by `/implement` |
| [`tdd`](skills/tdd/SKILL.md) | Red-green-refactor, one vertical slice at a time, testing at the public seam rather than internals | You ask for a TDD implementation, or `/implement` hands off after confirming the AC |
| [`code-review`](skills/code-review/SKILL.md) | Reviews the diff along three independent axes: AC fidelity, repo conventions, and whether the green is trustworthy. Three sub-agents run in parallel and their results are never merged | You ask to review a branch/PR/uncommitted changes, or `/tdd` has just finished |

All four work standalone, and they also chain — `/implement` is just the line that strings the other three together.

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

Once installed, invoke them in Claude Code with `/implement`, `/grill-me-ac`, `/tdd`, or just describe what you need and let them trigger themselves.
