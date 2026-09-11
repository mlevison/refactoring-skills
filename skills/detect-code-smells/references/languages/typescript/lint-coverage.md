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

## Mapping
| Smell | Rule that owns the mechanical half | What the rule cannot decide |
| --- | --- | --- |
| Floating Promise | `@typescript-eslint/no-floating-promises`, `no-misused-promises` (type-aware) | Nothing much. The rule is better than reading. |
| Suppressed Diagnostic | `@typescript-eslint/ban-ts-comment` with `minimumDescriptionLength`; ESLint's `--report-unused-disable-directives` | Whether the stated reason is honest, and whether the suppression outlived its cause |
| Barrel File Coupling | `import/no-cycle`, `import/no-self-import` | Whether a large `export *` barrel with no cycle is a public interface or a directory listing |
| Deep Relative Import | `import/no-relative-parent-imports`, or `no-restricted-imports` with path patterns | Whether the import points in the wrong *direction* - a utility reaching up into a feature |
| Unsafe Assertion | `@typescript-eslint/no-non-null-assertion`, `no-unnecessary-type-assertion` | A type predicate whose body checks less than its signature claims. No rule sees this. |
| Any Leakage | `@typescript-eslint/no-explicit-any`, the `no-unsafe-*` family (type-aware) | How far the `any` travelled from the boundary that produced it |
| Sequential Await | `no-await-in-loop` | Whether the iterations are actually independent - the rule cannot tell, and flags legitimate sequential work |
| Enum Over Union | No rule expresses this | All of it |

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

## How to report a gate gap
Once, in its own section, not once per instance. Name the rule, say how many instances reading found, and say that the count is a floor rather than a total - the rule will find the rest, including in the files this run did not read.

Where the rule **is** configured and enabled, report no instances of the mechanical half at all. Say lint owns it, and report only the judgment residue from the right-hand column.

A rule that is configured but disabled inline, or whose findings are suppressed, is a gate gap rather than a gate - see `smells/suppressed-diagnostic.md`.
