# Unsafe Assertion
**Remedy:** Narrow with a Type Guard · Replace Assertion with Validation · Replace Assertion with a Discriminated Union

## Lint owns most of this
`@typescript-eslint/no-non-null-assertion` finds every `!`. `no-unnecessary-type-assertion` finds the `as` that were never needed. See `../lint-coverage.md`.

**Where those are enabled, do not list `!` or redundant assertions.** Two things remain, and both need a reader:

- **A lying type predicate.** `function isUser(x: unknown): x is User { return typeof x === 'object' }` - the signature promises far more than the body checks. No rule compares the two, and this is the most dangerous form of assertion precisely because it looks like validation.
- **What the assertion claims about data from outside the program.** A rule can see `as Invoice`; it cannot judge whether the server is entitled to be believed.

## What it is
`as` and `!` do not check anything. They tell the compiler to believe something it could not prove, and then stop looking. Where the belief is wrong, the code fails at the first property access, with a message about the symptom and nothing about the assertion that caused it.

`value as Invoice` is a claim. `value!` is a claim. Neither is a conversion, and neither leaves a trace at runtime.

## What it costs
The compiler's model of the program diverges from what actually runs, and every type downstream of the assertion is trustworthy only if the assertion was right. Nothing re-checks it, so a change to the shape being asserted about breaks at runtime while the build stays green - which is precisely the failure TypeScript was adopted to prevent.

A `!` is worse than it looks, because it is one character. It reads as punctuation and carries the same risk as a cast.

## Thresholds
- **Any `as` on a value crossing a boundary** - parsed JSON, a query result, `localStorage`, an event payload. Clear territory.
- **`as unknown as X`** is always Clear. It exists only because the single assertion was rejected as impossible.
- **Two or more `!` in one expression** - `a!.b!.c` - is Clear.
- **Any `!` on a value from a lookup** - `map.get(key)!`, `arr.find(...)!`, `document.querySelector(...)!` - where nothing guarantees the hit.

Rules that cover this, which this skill does not run: `@typescript-eslint/no-non-null-assertion`, `@typescript-eslint/no-unnecessary-type-assertion`, `@typescript-eslint/consistent-type-assertions`.

## Judgment signals
- **An assertion about external data.** `(await res.json()) as Invoice` claims the server's contract is a fact.
- **A `!` after a lookup.** `users.find(u => u.id === id)!` asserts the user exists. The type said it might not because it might not.
- **Asserting to silence a narrowing failure.** The compiler lost the narrowing across a closure or an `await`, and `as` was used instead of restoring it.
- **A lying type predicate.** `function isUser(x: unknown): x is User { return typeof x === 'object' }` - the signature promises far more than the body checks. This is an assertion in disguise, and harder to spot.
- **Assertion in place of a discriminant.** `shape as Circle` where a `kind` field would let the compiler narrow it.
- **Assertions clustered around one type.** The same `as` in five files means the type is wrong, not that the code needs five assertions.

## When it's fine
- **Narrowing a literal to a const.** `as const`, which adds information rather than discarding checks.
- **A tested type guard.** An assertion inside a predicate whose body genuinely checks every field, with tests covering the negative cases.
- **Test doubles and fixtures.** `{} as User` in a test, where the point is a partial object and the test would fail loudly if the missing fields mattered.
- **A non-null assertion immediately after a check the compiler cannot see through** - though restoring the narrowing is almost always available and better.
- **DOM lookups in code that controls the document** - `querySelector('#root')!` in an entry point that created `#root`. The guarantee is real, just not expressible.
- **Working around a known upstream typing bug**, where a comment names the issue.

## Confidence guidance
**Clear** - an assertion about data from outside the program, `as unknown as`, a `!` on a lookup result, or a type predicate whose body checks less than its signature claims.

**Worth a look** - a local assertion where the invariant is plausible but unwritten, an assertion working around lost narrowing, or one in a test.

Report what the assertion claims and what would happen if the claim were false. An assertion whose failure mode cannot be described is probably fine.

## Sources
- TypeScript Handbook, "Type Assertions" and "Narrowing" - https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- typescript-eslint, `no-non-null-assertion`, `no-unnecessary-type-assertion` - https://typescript-eslint.io/rules/
- Dan Vanderkam, _Effective TypeScript_, 2nd edition, "Prefer Type Annotations to Type Assertions"
