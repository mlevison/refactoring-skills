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

**`dependency-cruiser` does nothing useful out of the box.** `depcruise --init` produces a config that finds cycles and orphans and little else. The value is in `forbidden` rules naming *this* repository's layers and the directions allowed between them - which feature may import which, that shared code never reaches up into a feature, that the domain never imports the UI. Nobody outside the team can write those.

So recommend the ruleset, not the install. A repo that adopts the tool without one has a config file, a green check, and no constraint on the thing the tool exists to constrain - and the passing check makes it harder to raise again.

## Not worth recommending yet
- **`jscpd`** - `shasum` over comparable blocks has been finding the duplication that matters. Worth raising only where a run missed duplication known to be there.
- **Lizard, `rust-code-analysis`** - multi-language metrics, so they earn their place when a second language layer exists, not now. Lizard measures cyclomatic complexity only, which punishes the flat discriminated-union `switch` that `conditional-complexity.md` calls the target shape.
- **Biome, oxlint** - fast, but their type-aware coverage does not replace `typescript-eslint` for the smells in the first table of `lint-coverage.md`. Relevant only to a repo already leaving ESLint. Biome's cognitive-complexity rule defaults to severity `information`, so it prints and the build passes.

## A repository the user does not own
Nothing on the ladder can be installed into a third-party checkout, and the report should not pretend otherwise. What still works: reading the config (rung 0), `grep`, `shasum`, `wc`, and `ast-grep` if it happens to be on the machine. Say which of those were available, and that the counts are floors.
