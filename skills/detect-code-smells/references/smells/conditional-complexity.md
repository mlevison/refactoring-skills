# Conditional Complexity
**Refactoring:** Decompose Conditional · Replace Nested Conditional with Guard Clauses · Consolidate Conditional Expression · Replace Conditional with Polymorphism

## What it is
Nested branches, chains of `else if`, or conditions built from several clauses joined by and/or. The reader has to hold the accumulated state of every enclosing test to understand any line inside.

The glossary entry this catalogue descends from makes the point on itself: *"Nested switch or if statements; a series of successive if statements. This preceding statement is itself an example."*

## What it costs
Nesting multiplies the paths through a function faster than anyone can enumerate them, so some paths are never tested and some are never reachable. Compound conditions are where off-by-one and inverted-logic bugs live, because nothing in `!(a || b) && c` tells the reader what question is being asked.

## Mechanical signals
- Nesting depth of conditionals.
- Cyclomatic or cognitive complexity, where a tool is available.
- Number of clauses in a single condition.
- Length of an `else if` chain, and whether it branches on the same value throughout.
- Negations, particularly a negation applied to a compound expression.

## Judgment signals
- **The condition has no name.** A multi-clause test that a reader must evaluate rather than read. `if (user.age >= 18 && user.country === 'CA' && !user.suspended)` is asking one question that has not been named.
- **Arrow-shaped code.** Indentation marching right and then back, with the real work at the deepest point.
- **The same test repeated.** The same condition checked at several levels, or re-checked in a branch where it is already known.
- **Branching on type.** A chain dispatching on what something *is* rather than what it should *do*.
- **A far-away `else`.** The `else` so distant from its `if` that the reader has lost which condition it belongs to.

## When it's fine
- **A flat dispatch table.** A `switch` over an enum with a short arm each, no nesting, is a lookup table written in control flow, and it reads well.
- **Guard clauses at the top.** Several sequential early returns are not nesting; they are the fix for nesting.
- **Genuinely irreducible business rules.** Tax, eligibility and pricing rules are complicated because the domain is. The smell is complexity the code added, not complexity it inherited - though a named condition still helps.
- **Exhaustiveness the compiler checks.** Where the language verifies that every case is handled, the chain is safer than it looks.

## Confidence guidance
**Clear** - deep nesting, or a compound unnamed condition, or a type-dispatch chain repeated in more than one place.

**Worth a look** - a long but flat chain, a condition that is complicated but does have a name, or domain rules whose shape may be irreducible.

Complexity produced by a legitimate domain rule is Worth a look at most. Complexity produced by structure is Clear.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Repeated Switches", "Decompose Conditional"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/conditional-complexity
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
