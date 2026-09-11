# Suppressed Diagnostic
**Remedy:** Declare the Suppression · Fix the Underlying Type · Narrow with a Type Guard

## Lint owns the counting
`@typescript-eslint/ban-ts-comment` finds these exactly, and can require a description of a minimum length. `--report-unused-disable-directives` finds the ones that are no longer needed. See `../lint-coverage.md`.

**Where `ban-ts-comment` is enabled, do not list suppressions.** What remains is the part no rule can judge: whether a stated reason is *honest*, and whether a suppression has outlived the problem it was added for. Those are findings. A count of `@ts-ignore` is not.

## What it is
A comment that switches off a compiler or linter error instead of answering it: `@ts-ignore`, `@ts-expect-error` with no reason, `eslint-disable-next-line` on a type-safety rule, or a whole-file `/* eslint-disable */`.

The error was information. Suppressing it keeps the information out of the build and out of the next reader's way.

## What it costs
`@ts-ignore` suppresses whatever error happens to be on the next line - including an error that arrives later, for an entirely unrelated reason. It is a permanent, silent exemption granted to a line of code on the strength of one day's problem.

More to the point, it is unfalsifiable. Nothing ever tells you the underlying issue was fixed, so the suppression outlives it, and removing it years later means reconstructing why it was added. A file-level disable extends that to every line in the file, including lines written afterwards by people who never saw the comment.

## Thresholds
- **Any `@ts-ignore`** is worth a look. `@ts-expect-error` is strictly better: it errors when the suppressed problem goes away.
- **Any `@ts-expect-error` with no description** following it is worth a look. With a reason and a link, it is a documented decision.
- **A file-level `/* eslint-disable */` or `// @ts-nocheck`** is Clear. The scope is unbounded and grows with the file.
- **`eslint-disable` on a type-safety rule** - the `no-unsafe-*` family, `no-floating-promises`, `no-explicit-any` - is Clear. Those are the rules that catch the expensive mistakes.
- **Two or more suppressions in one file** suggests a wrong type rather than several awkward lines.

Rules that cover this, which this skill does not run: `@typescript-eslint/ban-ts-comment` (which can require descriptions and forbid `@ts-ignore`), and ESLint's `--report-unused-disable-directives`.

## Judgment signals
- **No reason given.** A bare suppression is a decision with its reasoning deleted.
- **A suppression over an import.** Usually a missing or wrong `@types` package - a solvable problem, solved by silence.
- **A suppression in a test** covering up that the production type is wrong, rather than that the test is unusual.
- **Copy-paste clusters.** The same suppression on five consecutive lines is one type error, five times hidden.
- **A suppression older than the problem.** Where git blame is available, a suppression whose surrounding code has since been rewritten is probably unnecessary now.
- **`@ts-nocheck` at the top of a file** that is otherwise actively maintained.

## When it's fine
- **`@ts-expect-error` with a reason and a link**, particularly pointing at an upstream issue. It documents the constraint and it fails when the constraint lifts - the opposite of a stale suppression.
- **Deliberate negative tests.** Asserting that invalid code is rejected requires writing invalid code; `@ts-expect-error` is the correct tool and its presence is the assertion.
- **Generated files**, where the suppression belongs to the generator.
- **A genuine upstream bug** with no local fix, named in the comment.
- **A marked migration step** with a tracked item, not an intention.

## Confidence guidance
**Clear** - a file-level disable or `@ts-nocheck` in maintained code, a suppression of a type-safety rule, or a bare `@ts-ignore` over an expression that is doing real work.

**Worth a look** - an undescribed `@ts-expect-error`, a single `@ts-ignore` with a plausible cause, or a suppression in a test.

Say what error is being suppressed, where that can be determined, and what would have to change to remove it. A suppression nobody can describe the removal of is the one most likely to be permanent.

## Sources
- TypeScript Handbook, `@ts-expect-error` vs `@ts-ignore` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-9.html
- typescript-eslint, `ban-ts-comment` - https://typescript-eslint.io/rules/ban-ts-comment/
