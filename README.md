# skills

The [Claude Code](https://claude.com/claude-code) skills I use myself. They all revolve around the same idea: **pin down the acceptance criteria (AC) before writing any code, work through them one slice at a time with TDD, then review the diff independently and report back against the AC**.

## Why: Loop Engineering

This whole set exists to make **Loop Engineering** concrete. The unit of work isn't a prompt and a patch — it's a closed loop that an agent can run and that a human can trust the output of:

**Spec → Implement → Verify → Report → back to Spec.**

`grill-me-ac` closes the front of the loop by turning a vague request into criteria that can actually be checked. `tdd` walks those criteria one slice at a time, so every pass through the loop is small enough to reason about. `code-review` closes the back of the loop with an independent check, because a loop that grades its own output drifts. And the report lands back on the same AC the loop started from, so the next iteration begins where the last one measurably ended.

Two things follow, and the rest of [DESIGN.md](DESIGN.md) follows from them: the loop has to run unattended, and it has to be honest about what it couldn't verify by itself.

## Skills

| Skill | What it does | Who calls it |
| --- | --- | --- |
| [`implement`](skills/implement/SKILL.md) | Thin orchestrator: confirm the AC → hand off to TDD → review the diff → report against the AC | **You.** This is the entry point |
| [`grill-me-ac`](skills/grill-me-ac/SKILL.md) | Pins down the AC in the fewest round trips, split into machine-verifiable and human-judgment buckets | Step 1 of `/implement` |
| [`tdd`](skills/tdd/SKILL.md) | Red-green-refactor, one vertical slice at a time, testing at the public seam rather than internals | Step 2 of `/implement` |
| [`code-review`](skills/code-review/SKILL.md) | Three independent axes — AC fidelity, repo conventions, and whether the green is trustworthy — reported side by side, never merged | Step 3 of `/implement` |

### What to type

**`/implement`**, and describe the requirement as precisely as you would in a ticket. The more precise the request, the more the AC step collapses into a single confirmation instead of a round of questions. Three exceptions:

- You want the AC settled and agreed with someone else before any code exists → **`/grill-me-ac`**, then `/implement`.
- You're reviewing someone else's PR, or any diff this loop didn't produce → **`/code-review`**.
- It's a one-liner, a typo, a rename → just make the change.

Never `/tdd` on its own: it takes a confirmed AC from upstream and gets audited downstream, so calling it directly hands you the middle of the loop with neither end attached. And "this one matters, I should be careful" isn't a reason to start at `/grill-me-ac` — `/implement` runs it as step 1 anyway.

## Credits

`code-review`'s two-axis structure and its Fowler code smell baseline are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see [NOTICE](NOTICE)). The version here runs three axes: Standards is the original one, AC is the original Spec axis re-pointed at the confirmed AC list, and Green trust is this repo's own.

## Installation

Install with the [`skills` CLI](https://github.com/vercel-labs/skills) — it interactively lets you pick which skills to install and for which agent:

```bash
npx skills add leochiu-a/skills
```

Take all four: `/implement` is a thin orchestrator and does nothing without the three skills it calls. Useful flags — `--global` installs to `~/.claude/skills/` instead of the current project, `--list` shows what's in here, `--skill code-review -y` takes the one skill that's useful alone.

If you'd rather not use the CLI, copying the folders over works too:

```bash
git clone https://github.com/leochiu-a/skills.git && cp -r skills/skills/* ~/.claude/skills/
```

Then type `/implement` in Claude Code, or just describe what you want built and let it trigger itself.
