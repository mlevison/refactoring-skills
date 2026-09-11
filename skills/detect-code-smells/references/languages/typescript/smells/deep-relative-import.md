# Deep Relative Import
**Remedy:** Introduce a Path Alias · Move the Shared Type · Import from the Source

## Lint owns the depth
`import/no-relative-parent-imports`, or `no-restricted-imports` with path patterns, counts `../` segments exactly and across every file. See `../lint-coverage.md`.

**Where either is enabled, do not report depth.** Counting `../` is the definition of a job for a linter.

What remains is **direction**, which no rule can see: a utility reaching up into a feature, or one feature reaching sideways into another, is a dependency-inversion finding that happens to be written as a path. Report that. Depth alone, with a rule in place, is not a finding.

## What it is
`import { formatMoney } from '../../../lib/utils/currency'`. The path describes where the file sits on disk relative to this one, which means it encodes a fact about the filesystem into source code that otherwise does not care.

Two or more `../` is the line. Below that the import is local; above it, the path is navigation.

## What it costs
Moving a file rewrites every relative import in it, and every relative import pointing at it. That cost falls precisely when refactoring - the moment the code most needs to be rearranged is the moment rearranging is most expensive - so directories stop being reorganised and the structure ossifies.

The path is also unreadable as documentation. `../../../lib/utils/currency` tells a reader nothing about *where* that module lives in the project, only how far up to climb, so the dependency cannot be judged at a glance.

And the direction hides. `../../../` reaching from a low-level utility up into a feature module is a dependency pointing the wrong way, which a path alias would have made obvious.

## Thresholds
- **3 or more `../` segments** is Clear, where the project has path aliases configured.
- **2 `../` segments** is worth a look.
- **Any `../` crossing a feature or layer boundary** - out of one feature directory and into another, or from a shared utility up into a feature - is Clear regardless of depth. That is a dependency-direction finding wearing an import path.
- **A mix of aliased and deep-relative imports for the same target** in one file, or across a directory, means the convention exists and was not followed.

Rules that cover this, which this skill does not run: `import/no-relative-parent-imports`, `import/no-useless-path-segments`, and `no-restricted-imports` with path patterns.

## Judgment signals
- **Aliases already configured.** `paths` in `tsconfig.json`, or a bundler alias, means the project decided this and the file did not follow. Check for this before reporting anything, since it decides the tier.
- **`../` climbing out of a feature into a sibling feature.** Two features coupled directly, where the shared thing belongs somewhere both can see.
- **A low-level module importing upward.** `lib/utils/x.ts` reaching `../../features/billing/` is inverted: the utility now depends on the feature it serves.
- **Long paths in test files**, reaching from a parallel test tree into source. Common, and usually the easiest to fix.
- **`../../../index`** - a deep import *into a barrel*, combining this smell with the one next door.

## When it's fine
- **One `./` or one `../`.** A sibling or a parent is genuinely local, and an alias would be worse - it would hide that the module is right there.
- **No aliases configured.** Where the project has no `paths` and no bundler alias, a deep relative path is the only option available. Report it as a project-level observation at most, once, not as a finding per import.
- **Monorepo package imports** that are already package-scoped - `@scope/pkg` is an alias.
- **Generated files**, and framework-generated route trees.
- **Test fixtures deliberately loaded by path**, where the path is the point.
- **A single deep import in an otherwise aliased file** may be reaching something the alias config does not cover - worth noting, not worth a Clear.

## Confidence guidance
**Clear** - three or more `../` where aliases are configured, or any `../` crossing a feature boundary or pointing from a lower layer to a higher one.

**Worth a look** - two `../`, or a deep path in a project with no alias configuration.

Check `tsconfig.json` for `paths` once per run, not once per finding. Where no aliases exist, say so and make the observation once - twenty findings for a missing config line is noise, and the config line is the actual remedy.

## Sources
- TypeScript Handbook, `paths` and module resolution - https://www.typescriptlang.org/tsconfig#paths
- eslint-plugin-import, `no-relative-parent-imports` - https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-relative-parent-imports.md
