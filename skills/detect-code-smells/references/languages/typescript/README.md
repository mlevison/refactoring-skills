# TypeScript
The language layer. Thresholds and local forms for the neutral smells, the smells TypeScript owns outright, and what the toolchain can be expected to catch.

## What's in here
- `<smell>.md` - one per neutral smell, carrying the numbers and the TypeScript forms. The idea is in `../../smells/<smell>.md`; only the thresholds and the "cases that are fine" are here.
- `smells/` - the smells with no neutral parent. Self-contained, because there is nothing to translate.
- `remedies.md` - what to name as the fix for those, since Fowler's catalogue does not cover them.
- `lint-coverage.md` - which rule owns which smell, what each rule cannot decide, and the gate check to run once per run.

The workflow that uses all of this is `../../../SKILL.md`. This file does not repeat it.

## The skill does not run any of this
It reads `eslint.config.*`, `tsconfig.json` and `package.json`. It never installs a package, never invokes a linter, and never writes config into the target. Everything below is what the **report recommends to whoever owns the repo** - not a sequence to execute during a run.

The exception is `ast-grep`, which is a read-only sweep over files and is covered by `../../sweeping.md`.

## Recommending tools, in order
Ordered by value over cost, and the first two cost nothing. Recommending rung 4 to a repo that has not done rung 0 is advice nobody acts on.

| Rung | Tool | What it answers | Cost |
| --- | --- | --- | --- |
| 0 | The config already present | Is `typescript-eslint` on `strictTypeChecked` with `parserOptions.projectService`, and is `tsconfig` on `strict`? | Nothing. No install. This is `lint-coverage.md`'s single most valuable check and it outranks every rung below. |
| 1 | `type-coverage` | What percentage of expressions are not `any`. Settles whether Any Leakage is real here before anything is installed. | `npx type-coverage --detail`. Adds nothing to the repo. Gate later with `--at-least`. |
| 2 | `sonarjs/cognitive-complexity` | Conditional Complexity, mechanically, at the threshold this layer uses | One devDependency under LGPL-3.0-only, one rule line, one override for test globs |
| 3 | `dependency-cruiser` | Import cycles, and import *direction* - the gap nothing else fills | A local ruleset, which is the whole cost. See the note below the table. |
| 4 | `knip` | Dead code, unused exports, files and dependencies - what reading one file at a time cannot establish without being dangerous | A config, and a first run that reports a lot |
| 5 | `ast-grep` | The structural half of the sweep | A binary. Plus `sgconfig.yml` where the target is Svelte or Vue, which is why it is last - see `../../sweeping.md`. |

Rungs 0 and 1 measure without committing to anything. Everything from rung 2 is a commitment, and each one inherits the same two conditions: drive the repo to zero before turning the gate on, and put it in the command that actually blocks release rather than an advisory workflow.

## Start at rung 0, because a config can look configured and check nothing
The recommendation to make first is almost always tightening `eslint.config.*`. It costs no install, it blocks the build the same day, and it decides how much of the rest of this catalogue the skill even needs to look at.

What to look for, in rough order of how often it is the answer:

- **Type-aware linting scoped to one block.** `parserOptions.projectService` set only inside a `.svelte` or `.vue` override, while the `.ts` block has no project service at all - so no type-aware rule runs on any TypeScript file. The config reads as configured, `eslint` exits 0, and `no-floating-promises`, the whole `no-unsafe-*` family and `no-non-null-assertion` are all silently off. This is the single most common way the gate is missing.
- **`recommended` rather than `recommendedTypeChecked` or `strictTypeChecked`**, which switches off the same set a different way.
- **`files` globs that miss extensions the repo actually uses** - `.svelte`, `.vue`, `.mts`, `.cts` - or an `ignores` that quietly excludes a whole directory.
- **`disableTypeChecked` applied too widely.** Correct for `**/*.js`. Applied to tests, it removes the rules from the files where unsafe casts collect.
- **Rules present but set to `warn`**, or a lint script without `--max-warnings 0`, or a lint script that the release command never calls.

The decisive check is `npx eslint --print-config path/to/a/real/file.ts`, run once per extension the repo contains. It prints the *effective* config for that file, so a rule that is off shows as off. Reading the config file itself cannot tell you this, because the failure is about which block matched.

### What `strictTypeChecked` already turns on
Most of the first table in `lint-coverage.md`: `no-floating-promises`, `no-misused-promises`, every `no-unsafe-*`, `no-explicit-any`, `no-non-null-assertion`, `ban-ts-comment`, `no-unnecessary-condition`, `no-unnecessary-type-assertion`, `await-thenable`, `require-await`, `no-unsafe-enum-comparison`. It also brings `no-deprecated` and `no-confusing-void-expression`, which are worth having and which nobody asks for by name.

So do not recommend these individually. Recommend the preset plus `projectService`, which is one change, and check it reaches every file type.

### What has to be added on top
None of these are in any preset. Each maps to a smell in `lint-coverage.md`.

| Rule | Why |
| --- | --- |
| `@typescript-eslint/switch-exhaustiveness-check` | Makes the discriminated-union `switch` actually exhaustive - the shape this layer calls the target |
| `no-restricted-syntax` on selector `TSEnumDeclaration` | Enum Over Union. Or `tsc --erasableSyntaxOnly`, which does it at the compiler. |
| `sonarjs/cognitive-complexity`, at 15 | Conditional Complexity |
| `max-params`, at 4 | Long Parameter List |
| `max-lines-per-function`, at 60, skipping blanks and comments | Long Method, at the tier where the legitimate cases are the exception |
| `max-depth`, at 3, and `no-nested-ternary` | Nesting, which this layer weighs above length |
| `no-param-reassign` with `props: true` | The mutation subset of Side Effect |
| `no-await-in-loop` | Sequential Await |
| `import/no-cycle`, `import/no-self-import` | Barrel File Coupling |
| `import/no-relative-parent-imports`, or `no-restricted-imports` with path patterns | Deep Relative Import |
| `@typescript-eslint/prefer-readonly-parameter-types` | Side Effect, where the team will accept it - noisy, so raise it as optional |

Two settings that are not rules and get missed: `linterOptions.reportUnusedDisableDirectives` in flat config, and `ban-ts-comment`'s `minimumDescriptionLength`, which is on by default at a length short enough to pass anything. Both belong to Suppressed Diagnostic.

And in `tsconfig.json`: `strict`, plus `noUncheckedIndexedAccess`, which is not in `strict` and which removes a class of `!` that otherwise looks necessary.

**`dependency-cruiser` does nothing useful out of the box.** `depcruise --init` produces a config that finds cycles and orphans and little else. The value is in `forbidden` rules naming *this* repository's layers and the directions allowed between them - which feature may import which, that shared code never reaches up into a feature, that the domain never imports the UI. Nobody outside the team can write those.

So recommend the ruleset, not the install. A repo that adopts the tool without one has a config file, a green check, and no constraint on the thing the tool exists to constrain - and the passing check makes it harder to raise again.

## One class of finding at a time
Recommend the passes in sequence, never together. Each pass fixes one class, goes in on its own, and turns its rule on at the end - drive to zero, then gate, per class rather than once at the end of everything.

1. **ESLint config.** No source changes. The diff is one file, and it is the change that decides what the next three passes will even see.
2. **Types and `any`.** Everything the config change just switched on: `no-unsafe-*`, `as`, `!`, `any` at boundaries. The largest pass by volume, and the one to keep smallest per commit - it looks mechanical and is not. Removing `map.get(key)!` properly means adding a branch that did not exist, which is a behaviour change wearing a type change's clothes.

   In a few words: `!` becomes restored narrowing or an early return; `as` on anything crossing a boundary becomes a parse at that boundary, once, so the core never asserts; `any` becomes `unknown`, which forces every use to narrow. The named remedies are in `remedies.md`, and the cases where an assertion is genuinely right - `as const`, a tested predicate, a fixture, a DOM node the entry point created - are in `smells/unsafe-assertion.md` and `smells/any-leakage.md`. Deeper than that: the [TypeScript Handbook on narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html), the [typescript-eslint rule docs](https://typescript-eslint.io/rules/), and Vanderkam, _Effective TypeScript_ 2e, "Prefer Type Annotations to Type Assertions".

   Sequencing note: turning on `noUncheckedIndexedAccess` in pass 1 makes this pass larger, because every indexed read starts admitting it can miss. That is the point of the flag, but it is a reason to land it deliberately rather than alongside everything else.
3. **Dependencies.** Cycles and import direction. This one moves and re-points files, so running it before pass 2 would reflow that diff for no reason.
4. **`knip`.** Dead code last, and this ordering is not cosmetic - fixing an `any` or breaking a cycle changes what is actually reachable. Run early, it reports code that is about to become used and misses code about to become unused.

Complexity is deliberately not a pass here. Rung 2's findings are refactorings decided one at a time against **When it's fine**, not a sweep, and batching them is how a complexity gate turns into mechanical function-splitting that makes the code worse.

The reason for the sequence is diff legibility rather than tidiness. A pass that mixes four classes produces a diff where one genuine behaviour change hides among hundreds of mechanical ones, and no reviewer finds it. A pass that contains one class can be reviewed by someone who knows what that class should look like - and reverted alone when it turns out to be wrong.

## Not worth recommending yet
- **`jscpd`** - `shasum` over comparable blocks has been finding the duplication that matters. Worth raising only where a run missed duplication known to be there.
- **Lizard, `rust-code-analysis`** - multi-language metrics, so they earn their place when a second language layer exists, not now. Lizard measures cyclomatic complexity only, which punishes the flat discriminated-union `switch` that `conditional-complexity.md` calls the target shape.
- **Biome, oxlint** - fast, but their type-aware coverage does not replace `typescript-eslint` for the smells in the first table of `lint-coverage.md`. Relevant only to a repo already leaving ESLint. Biome's cognitive-complexity rule defaults to severity `information`, so it prints and the build passes.

## A repository the user does not own
Nothing on the ladder can be installed into a third-party checkout, and the report should not pretend otherwise. What still works: reading the config (rung 0), `grep`, `shasum`, `wc`, and `ast-grep` if it happens to be on the machine. Say which of those were available, and that the counts are floors.
