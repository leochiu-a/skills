# Design trade-offs

Why the loop is shaped the way it is. See the [README](README.md) for what the skills are and how to use them.

- **Two buckets of AC.** Machine-verifiable items (tests, typecheck, build) can be verified automatically; human-judgment items (UX, copy, business logic, risk) can only be flagged for a person to look at. Keeping them apart avoids the misleading "everything is green so it's done" report.
- **Never ask for AC empty-handed.** Draft one first, ask only about the parts you genuinely can't decide, and ask everything at once instead of one question per turn.
- **Vertical slicing.** One AC item goes red → green before moving to the next, rather than writing every test up front and implementing them together. Refactoring is its own separate pass once everything is green.
- **Test the seam, not the internals.** No mocking internal collaborators, no testing private methods — that's what lets the tests survive later refactors.
- **Green has to be verified independently.** When `/tdd` says "it's green," that's its own claim about its own work, and exit codes, CI, and test reports all sit downstream of the tests — when the tests themselves are modified, those signals confirm each other and then get it wrong together. So `code-review` has one axis that asks nothing but: was this green earned by the implementation, or bent out of the tests?
- **The reviewer can't be the author.** The context that just wrote the code knows why every compromise looked reasonable at the time; it can't review its own diff, and it can't be the one framing the review either — so all three axes run in independent sub-agents that receive only the fixed point and the AC, and re-derive everything else from git. The implementer's account of what it did is never taken at face value.
- **The three axes are never merged or reordered.** A change can follow every convention while building the wrong thing, follow the AC exactly while breaking conventions, or be clean and on-spec while its green came from loosened assertions. Collapsing them into one verdict just lets one axis cover for another.
