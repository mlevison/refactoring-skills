# TypeScript Remedies
Fowler's catalogue covers smells that exist in any language. The smells in `smells/` exist because TypeScript exists, so they need their own remedies.

Same rule as `references/refactorings.md`: a finding names one of these and stops. Performing it is a separate job.

| Remedy | What it does |
| --- | --- |
| Parse at the Boundary | Validate untyped input into a typed value once, at the edge, so the core never sees `any` or `unknown` |
| Replace `any` with `unknown` | Keep the value opaque so every use has to narrow it, rather than trusting it everywhere |
| Narrow with a Type Guard | Replace an assertion with a predicate that actually checks, so the compiler's belief matches runtime |
| Replace Assertion with Validation | Swap `as X` for a check that can fail, at the one place the value enters |
| Replace Assertion with a Discriminated Union | Let the type carry the variant so no assertion is needed to pick it |
| Declare the Suppression | Convert `@ts-ignore` to `@ts-expect-error` with a reason and a link, so it fails when the underlying problem is fixed |
| Fix the Underlying Type | Correct the type or the dependency's types rather than silencing the diagnostic |
| Await the Promise | Await it, or return it, so the caller owns the outcome |
| Handle the Rejection | Attach an error path to a promise nobody awaits, rather than leaving it to the process |
| Make the Fire-and-Forget Explicit | `void promise.catch(handler)`, so the intent is written down rather than implied by omission |
| Parallelise with `Promise.all` | Replace sequential awaits over independent work with one concurrent wait |
| Collect with `Promise.allSettled` | Replace a loop that stops at the first failure, where the rest still matter |
| Replace Enum with Literal Union | Use `'a' \| 'b'` so the values are structural, erasable, and need no runtime object |
| Import from the Source | Replace a barrel import with the module that actually defines the thing |
| Split the Barrel | Separate the public surface from the internals it currently re-exports |
| Introduce a Path Alias | Replace `../../../` with a configured alias, so a moved file does not rewrite its imports |
| Move the Shared Type | Break a cycle by extracting what both modules need into a module that depends on neither |
| Invert the Dependency | Break a cycle by pointing the lower-level module at an interface rather than at the higher-level one |

## On mechanical signals
Each smell file names the `typescript-eslint` or `eslint-plugin-import` rule covering it, where one exists. **This skill does not run them.** They are named so a future tool-assisted phase knows where to attach, and so a reader can check a finding against a rule they may already have configured.

A rule being disabled in a repo is not evidence the smell is absent. It is often evidence of the opposite.

## Sources
- TypeScript Handbook - https://www.typescriptlang.org/docs/handbook/
- typescript-eslint rules - https://typescript-eslint.io/rules/
- Dan Vanderkam, _Effective TypeScript_, 2nd edition (O'Reilly, 2024)
