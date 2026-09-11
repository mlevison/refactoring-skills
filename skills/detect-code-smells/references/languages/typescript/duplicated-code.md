# Duplicated Code in TypeScript

## Thresholds
Occurrence count matters more than block length. A one-line idiom repeated fifty times is a worse finding than a twenty-line block repeated once.

- **6 or more identical lines in 2 or more places.**
- **Any repeated expression appearing 5 or more times** across the target.
- **3 or more files with the same sequence of steps**, however differently written.
- **2 near-identical blocks with one unexplained difference** - report at any length, because drift has already happened.

## How it shows up
- **The error-narrowing idiom.** `err instanceof Error ? err.message : String(err)` is the commonest duplicated expression in TypeScript codebases. Repeated dozens of times, it is one `toMessage(err)` that was never written.
- **Parallel route or page modules.** Several pages each doing load, guard, fetch, map, handle-error in the same order. The orchestration is duplicated even where no two lines match.
- **Component markup.** The same form, card or table written twice - frequently near-identical except for indentation, which makes it easy to miss in review and easy to find by reading.
- **Repeated `zod` or validation schemas** describing the same shape in two modules.
- **`try`/`catch` blocks with identical bodies** across sibling handlers.
- **Reimplemented local helpers.** The same small function defined several times in one file, or once per module, because nobody knew it already existed. A helper defined more than once *within a single file* is Clear with no further analysis.
- **Duplicated type declarations.** The same object shape written as a `type` in two places instead of imported - which drifts silently, since nothing compares them.
- **Copy-pasted `useEffect` or lifecycle bodies** across components.

## TypeScript cases that are fine
- **Tests.** `describe` blocks repeating arrange steps on purpose. Judge a shared fixture harder than the repetition it would replace.
- **Generated output** - `*.d.ts`, GraphQL codegen, Prisma client, API clients, route manifests.
- **Barrel files**, whose content is repetitive by definition.
- **Config and constant declarations** that look alike because they describe parallel things.
- **Structural similarity across a deliberate boundary** - two packages in a monorepo kept independent on purpose. Report it as present, not as wrong.
- **Two components that look alike today and answer to different designers tomorrow.** Merging these produces a component with nine boolean props, which is a worse file than the two it replaced.

## A note on `.svelte`, `.vue` and `.tsx`
Duplication in markup reads differently from duplication in logic: it is usually longer, more obvious, and more often legitimate. Before reporting duplicated markup, check whether the two copies are diverging designs or one component. Where they are byte-identical apart from whitespace, they are one component.
