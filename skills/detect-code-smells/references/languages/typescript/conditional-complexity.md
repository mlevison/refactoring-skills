# Conditional Complexity in TypeScript

## Thresholds
- **Nesting past 3 levels** of `if` / `for` / `try`.
- **3 or more clauses** in one condition, or any negation applied to a compound expression.
- **Cognitive complexity past ~15** where a tool reports it. `sonarjs/cognitive-complexity` is the one to prefer, for the reason in `lint-coverage.md`: it discounts the flat `switch` this file calls the target shape, and cyclomatic complexity punishes it. Neither is required by this skill.
- **An `else if` chain past 4 arms** that is not a `switch`.

## How it shows up
- **Nested `if` inside `try` inside `for`**, with the work at the bottom.
- **`if (a && b || c && !d)`** - a question nobody has named. Extract Variable gives it one.
- **Repeated type narrowing.** The same `typeof` or `in` check re-done in several branches because the narrowing was lost.
- **`if (x !== undefined && x !== null && x.items.length > 0)`** where `x?.items?.length` says it.
- **Nested ternaries**, especially inside JSX, where indentation cannot help the reader.
- **Branching on a string type code** - `if (user.type === 'admin') ... else if (user.type === 'guest')` - repeated in several files. That is Replace Conditional with Polymorphism, or at least a discriminated union.
- **`try`/`catch` inside a loop inside a conditional**, in async code, where the happy path is invisible.

## TypeScript cases that are fine
- **A `switch` over a discriminated union** with a `never` exhaustiveness check in the default arm. Flat, compiler-verified, and the arms are short. This is the target shape, not a smell.
- **Guard clauses** - several early returns at the top of a function are the fix for nesting, not an instance of it.
- **Optional chaining and nullish coalescing.** `a?.b ?? fallback` is one expression, not three conditionals.
- **Type guards** (`function isUser(x: unknown): x is User`) whose bodies are necessarily a run of checks.
- **Irreducible domain rules** - tax, eligibility, pricing. Naming the condition still helps, but the branch count is the domain's, not the code's.
