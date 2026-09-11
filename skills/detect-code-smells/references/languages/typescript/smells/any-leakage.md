# Any Leakage
**Remedy:** Parse at the Boundary · Replace `any` with `unknown` · Narrow with a Type Guard

## What lint owns, and what it doesn't
`@typescript-eslint/no-explicit-any` finds every declared `any`, and the type-aware `no-unsafe-*` family finds the uses. See `../lint-coverage.md`.

Those rules find `any`. **This smell is about distance** - how far an `any` travelled from the boundary that produced it - and no rule measures that. One `any` converted on the next line is the boundary working; the same `any` read three modules later is the finding.

Where the rules are enabled, do not list declarations. Report the journeys.

## What it is
An `any` that travels. One `any` at a boundary is a fact of life - JSON, a third-party library, a database driver. The smell is that `any` being carried inward instead of converted at the edge, so everything downstream is unchecked while still *looking* checked.

`any` is contagious in a way no other type is: a property of an `any` is `any`, the return of a call on an `any` is `any`, and none of it is reported.

## What it costs
Every type in the path becomes a comment. The editor still offers autocomplete, the build still passes, and the guarantee is gone - so the failure arrives at runtime, in production, far from the `any` that permitted it.

It also hides from review. A reviewer reading `user.profile.displayName` cannot tell whether `user` is a `User` or an `any` three function calls deep, and nothing in the diff says which.

## Thresholds
- **Any `any` in a function signature** - parameter or return - is worth a look. It is a contract saying "unchecked".
- **Any `any` more than one call away from the boundary that produced it** is Clear.
- **`as any`** is two smells at once; see also `unsafe-assertion.md`.
- **Implicit `any`** where `noImplicitAny` is off, or in an untyped `catch` binding under older settings.

Rules that cover this, which this skill does not run: `@typescript-eslint/no-explicit-any`, and the `no-unsafe-*` family (`no-unsafe-assignment`, `no-unsafe-argument`, `no-unsafe-call`, `no-unsafe-member-access`, `no-unsafe-return`), which catch the leak rather than the declaration.

## Judgment signals
- **`any` in a return type.** The single worst position: one function's convenience becomes every caller's problem, invisibly.
- **`Record<string, any>` or `{ [key: string]: any }`** used as "an object of some kind". It types the container and abandons the contents.
- **`any[]`**, which promises a list and guarantees nothing about it.
- **A cast chain.** `response as any as Invoice` - two assertions because one was refused.
- **`catch (error: any)`** followed by `error.response.data.message`. Errors are `unknown` for a reason; this walks a shape nothing verified.
- **No boundary in sight.** The `any` originated several modules away and has been passed along since.

## When it's fine
- **At the boundary, converted immediately.** `const raw: any = await res.json()` on one line and a parse into a typed value on the next. That is the boundary doing its job.
- **Genuinely generic utility code** where the type is irrelevant to the operation - though `unknown` plus a generic parameter usually expresses it better and keeps the checking.
- **Declaration files for untyped dependencies.** `*.d.ts` shims for a library with no types are `any` by necessity.
- **Tests that deliberately construct invalid input**, where the point is to pass something the type forbids.
- **Migration in progress**, where `any` is a marked staging post rather than a destination - and marked means a comment or a tracked item, not an intention.

## Confidence guidance
**Clear** - `any` in an exported signature, or an `any` whose members are accessed more than one hop from where it entered, or `catch (error: any)` followed by property access.

**Worth a look** - a single local `any` at a boundary with no visible conversion, a generic utility that could use `unknown`, or an `any` in a test.

Never report an `any` without saying where it entered and how far it travelled. "There is an `any` here" is not a finding; the distance is.

## Sources
- TypeScript Handbook, "unknown" and "any" - https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
- typescript-eslint, `no-explicit-any` and the `no-unsafe-*` family - https://typescript-eslint.io/rules/
- Dan Vanderkam, _Effective TypeScript_, 2nd edition, Chapter 5: "Working with any"
