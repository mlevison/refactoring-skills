# Long Method
**Refactoring:** Extract Function · Split Phase · Slide Statements · Replace Loop with Pipeline

## What it is
A function long enough that a person cannot hold it in mind at once. The working limit is human, not numeric: by the time the reader reaches the bottom they have lost the top, so they scroll back, and each scroll costs them the thread.

The rule of thumb is a screen. A function taller than the screen it is read on has already asked the reader to remember something they can no longer see.

## What it costs
Every later change to it starts with reconstructing what it does. That reconstruction is where bugs enter, because the reader forms a model of the function that is close to right rather than right. Long functions also resist testing: the more a function does, the more setup a test needs, and the fewer of its paths anyone bothers to cover.

## Mechanical signals
Countable, and none of them decide the question on their own:
- Length in lines, measured against a screen rather than a constant.
- Nesting depth, and how far the deepest line sits from the left margin.
- Number of local variables live at once.
- Number of distinct responsibilities suggested by blank-line-separated blocks or by comments introducing sections.

## Judgment signals
- **Comments acting as section headings.** A comment introducing the next ten lines is naming a function that has not been extracted yet.
- **The name needs "and".** If describing the function honestly requires a conjunction, it is doing two things.
- **A change of altitude.** The function opens on policy - what should happen - and descends into mechanism, byte-shuffling and null-handling, then climbs back.
- **Reused variables.** A variable assigned early, used, then reassigned for a different purpose later, is two functions sharing a scope.

## When it's fine
- **A flat sequence with no branching.** A long mapping between two shapes, a configuration block, a switch over an enum where every arm is one line. Length here carries no complexity, and breaking it up makes it harder to read, not easier.
- **Generated code**, or a file whose header says it is generated.
- **A test.** Arrange-act-assert is deliberately linear, and extracting the arrange step into a helper often hides what the test is really doing.
- **The alternative is worse.** Extraction that produces functions taking eight parameters, or functions only meaningful in one order, has moved the problem rather than solved it. If no honest name exists for the fragment, the fragment is not a function yet.

## Confidence guidance
**Clear** - long, and carrying more than one responsibility, with at least one judgment signal present. Length plus section-heading comments is the archetype.

**Worth a look** - long, but the responsibilities are not obviously separable, or the length comes from a shape that might be one of the legitimate cases above.

Length alone is never Clear. A function that is only long is a fact, not a finding.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Long Function"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/long-method
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
