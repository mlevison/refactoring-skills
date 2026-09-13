# Lint Coverage
Where a lint rule decides a question mechanically and exactly, lint is the better answer: it runs in CI, it blocks the build, it is deterministic, and it does not need a model. Reporting instances of something a configured rule already catches is noise, and reporting them to someone who has *not* configured the rule is forty findings where one config line would do.

So for the smells below, the skill's job is the **gate**, not the instances.

## What to read, once per run
- `eslint.config.{js,mjs,cjs,ts}` - flat config, current
- `.eslintrc.{js,cjs,json,yml,yaml}` or `package.json` → `eslintConfig` - legacy config
- `tsconfig.json` and anything it `extends`
- `package.json` - is `typescript-eslint` / `@typescript-eslint/*` even installed, and is there a `lint` script

Absent any lint config at all, say that once, as the finding. It is the highest-value thing in the report.

## The single most valuable check
`typescript-eslint`'s plain `recommended` config does **not** enable the type-aware rules. Those need `recommendedTypeChecked` or `strictTypeChecked` plus `parserOptions.projectService`. A repo on plain `recommended` has `no-floating-promises` and the whole `no-unsafe-*` family switched off and usually does not know it.

Check for that before anything else. It is one change that turns on most of this table.

## Mapping: the TypeScript-native smells
| Smell | Rule that owns the mechanical half | What the rule cannot decide |
| --- | --- | --- |
| Floating Promise | `@typescript-eslint/no-floating-promises`, `no-misused-promises` (type-aware) | Nothing much. The rule is better than reading. |
| Suppressed Diagnostic | `@typescript-eslint/ban-ts-comment` with `minimumDescriptionLength`; ESLint's `--report-unused-disable-directives` | Whether the stated reason is honest, and whether the suppression outlived its cause |
| Barrel File Coupling | `import/no-cycle`, `import/no-self-import` | Whether a large `export *` barrel with no cycle is a public interface or a directory listing |
| Deep Relative Import | `import/no-relative-parent-imports`, or `no-restricted-imports` with path patterns | Whether the import points in the wrong *direction* - a utility reaching up into a feature |
| Unsafe Assertion | `@typescript-eslint/no-non-null-assertion`, `no-unnecessary-type-assertion` | A type predicate whose body checks less than its signature claims. No rule sees this. |
| Any Leakage | `@typescript-eslint/no-explicit-any`, the `no-unsafe-*` family (type-aware) | How far the `any` travelled from the boundary that produced it |
| Sequential Await | `no-await-in-loop` | Whether the iterations are actually independent - the rule cannot tell, and flags legitimate sequential work |
| Enum Over Union | `tsc --erasableSyntaxOnly` (TypeScript 5.8+) errors on every non-`declare` enum; `no-restricted-syntax` on selector `TSEnumDeclaration` does the same in lint | Nothing, once either is on. Both are exact and neither has a false positive. |

## Mapping: Fowler's smells
Weaker coverage. A rule here produces a candidate list, and the **When it's fine** section still decides - so these stay reported as findings, with the rule named as the cheaper way to keep finding them.

| Smell | Rule that narrows it | What the rule cannot decide |
| --- | --- | --- |
| Conditional Complexity | `sonarjs/cognitive-complexity` - see below. Plus `max-depth`, `no-nested-ternary`, `@typescript-eslint/switch-exhaustiveness-check` | Whether the branch count is the domain's. Tax, eligibility and pricing score high and are still right. |
| Long Method | `max-lines-per-function` at the 60-line tier, `max-statements` | That a flat 200-line mapper is still a Long Method at a cognitive score of nearly zero, and that JSX is not body |
| Long Parameter List | `max-params` at 4 | Adjacent same-type parameters - the real risk, and `max-params` cannot see types. An 8-property options bag scores 1. |
| Duplicated Code | `sonarjs/no-identical-functions`, `sonarjs/no-duplicate-string` | The same steps written differently, which is what token comparison misses |
| Side Effect | `no-param-reassign` with `props: true`, `@typescript-eslint/prefer-readonly-parameter-types` | The smell itself - the gap between what the name promises and what the body does. No rule reads names. |
| Flag Argument | `no-restricted-syntax` on `CallExpression > Literal[value=true]` | Configure from switch. The selector flags every `disabled` prop and every library callback. |
| Message Chain | `no-restricted-syntax` on member-expression depth | Navigation from operation. The selector cannot tell a domain chain from a query builder. |
| Feature Envy | None | All of it |
| Primitive Obsession | None | All of it. `noUncheckedIndexedAccess` and branded types are the remedy, not a detector. |

## Cognitive complexity, where a complexity rule is wanted
Prefer `sonarjs/cognitive-complexity` over ESLint's cyclomatic `complexity`, and over line count as a gate. It discounts a flat `switch` and charges for nesting, which is what this layer's own thresholds say - `conditional-complexity.md` calls the discriminated-union `switch` the target shape, and cyclomatic complexity punishes exactly that.

`eslint-plugin-sonarjs` is an npm package under LGPL-3.0-only. It needs no SonarQube server and no account.

```js
{ rules: { 'sonarjs/cognitive-complexity': ['error', 15] } }
```

Two things it does not replace. **Long Method survives it** - a long straight-line mapper scores near zero and a person still cannot hold it in mind, so line count stays a reading prompt even where this rule is the gate. And **test files inflate**, because a `describe`/`it` tree scores the framework's nesting rather than the test's logic; an override for test globs is part of recommending it.

Biome's `complexity/noExcessiveCognitiveComplexity` is the same algorithm under MIT, for a repo that cannot take LGPL. It is not in Biome's recommended set and its default severity is `information`, so it needs `"level": "error"` explicitly or it prints and the build passes.

## Minimum worth recommending
Where nothing is configured, recommend this rather than a list of individual rules. One block, and it subsumes most of the table.

```js
// eslint.config.js
import tseslint from 'typescript-eslint';

export default tseslint.config(
  ...tseslint.configs.strictTypeChecked,
  { languageOptions: { parserOptions: { projectService: true } } },
);
```

And in `tsconfig.json`, `"strict": true` - which turns on `noImplicitAny` and `strictNullChecks` together. `noUncheckedIndexedAccess` is worth naming separately; it is not in `strict` and it catches a class of `!` that otherwise looks necessary.

Per your own quality-gate principle: drive the repo to zero before turning a rule on, and put it where release is actually blocked. A rule added to an advisory workflow that nothing waits on is decoration - so recommending a rule means recommending the gate it belongs in.

This config block is rung 0 and rung 2 of the ladder in `README.md`, which has the rest of the tools and the order to adopt them in. Recommend from there rather than assembling an order here.

## How to report a gate gap
Once, in its own section, not once per instance. Name the rule, say how many instances reading found, and say that the count is a floor rather than a total - the rule will find the rest, including in the files this run did not read.

Where a rule from the **first** table is configured and enabled, report no instances of the mechanical half at all. Say lint owns it, and report only the judgment residue from the right-hand column.

The second table does not work that way. A configured `max-params` does not settle Long Parameter List, because the rule cannot see the thing the smell is actually about. Report those findings as normal, and name the rule as the cheaper way to keep finding them.

A rule that is configured but disabled inline, or whose findings are suppressed, is a gate gap rather than a gate - see `smells/suppressed-diagnostic.md`.
