# Duplicated Code
**Refactoring:** Extract Function · Pull Up Method · Extract Class · Parameterize Function · Combine Functions into Class

## What it is
The same idea expressed more than once. Fowler calls it the worst of the smells, and the glossary this catalogue descends from agrees: *"Possibly the worst smell."*

Four kinds, and they are not equally findable:

1. **Identical** - the same text, perhaps reindented or with renamed locals.
2. **Near-identical** - the same structure with small deliberate differences, which are usually the parameters the extracted version would take.
3. **Structural** - the same algorithm with statements reordered, different control-flow shapes, different names.
4. **Semantic** - the same outcome by different means, with no textual resemblance at all.

## What it costs
A change has to be made everywhere, and nothing tells you where everywhere is. The bug fixed in one copy survives in the other three, which is the single most common way a fixed defect comes back.

Duplication also multiplies the reading cost of every future change: before editing one copy, someone has to work out whether the others are deliberately different or accidentally drifted. That question often has no answer left.

And it hides design. Three copies of an orchestration are one orchestration nobody has named yet - the duplication is the evidence that the abstraction exists and is missing.

## Mechanical signals
- Identical or near-identical token runs, and how many lines each spans.
- Count of occurrences, which matters more than the length of any one of them.
- The same expression repeated - an idiom like an error-narrowing ternary appearing dozens of times is duplication even though each instance is one line.
- Files whose size is similar and whose import lists match.
- Repeated literal values, format strings and magic numbers.

## Judgment signals
- **Parallel file structures.** Four list pages with the same sequence of steps in the same order. The steps are the duplication, not the lines.
- **Copy-paste with a rename.** Identical structure where only identifiers differ; the differing identifiers are the parameters.
- **Sibling subclasses or sibling routes** with the same method implemented identically.
- **A repeated idiom.** `err instanceof Error ? err.message : String(err)` fifty times is one function that does not exist. Each instance is trivial; the aggregate is not.
- **Drifted copies.** Two near-identical blocks with one small difference, where nobody can say whether the difference is intentional. That is worse than exact duplication, because the behaviours have already diverged.
- **The same comment twice.** Duplicated prose usually sits above duplicated code.

## When it's fine
- **Tests.** Explicit, repetitive setup is often clearer than a shared helper, and a test that shares its fixture with twenty others fails for reasons nobody can localise. Repetition in tests is a deliberate trade, not an oversight.
- **Coincidental similarity.** Two blocks that look alike but answer to different requirements will need to change independently. Merging them creates a coupling between unrelated things, which is worse than the duplication - this is the mistake that produces a parameter-riddled helper nobody can read.
- **Two lines.** Not every repetition is worth a name. The threshold is whether a change would have to be made in both places for the same reason.
- **Across a deliberate boundary.** Duplication between two services, or either side of a module boundary kept decoupled on purpose, may be the price of that independence. Say it is there; do not assume it should be removed.
- **Generated code.**
- **A deliberate inline for performance**, where a comment says so.

## Confidence guidance
**Clear** - identical or near-identical blocks in two or more places, with a count and locations; or a repeated idiom whose occurrence count is high enough to state.

**Worth a look** - structurally similar code that might be coincidental, or duplication across a boundary that may be deliberate.

State the occurrence count and every location. "There is duplication here" is not actionable; "this 28-line block appears in these two files" is.

## What this misses
Reading finds kinds 1 and 2 reliably, and kind 3 only when the resemblance is visible. **Kind 4 is invisible to it, and so is kind 3 at any scale beyond a few files.** Say so when reporting, so a clean result is not read as "no duplication".

A clone detector that hashes and compares does this properly, and the product backlog carries it. Until then, the absence of a Duplicated Code finding means nothing was obvious, not that nothing is there.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Duplicated Code"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/duplicated-code
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
