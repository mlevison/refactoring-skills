# Barrel File Coupling
**Remedy:** Import from the Source · Split the Barrel · Move the Shared Type · Invert the Dependency

## Lint owns the cycles
`import/no-cycle` finds import cycles exactly, and traces the full path, which reading cannot do reliably past two hops. See `../lint-coverage.md`.

**Where `import/no-cycle` is enabled, do not hunt cycles by reading** - it has already done it, better. Report the gate gap if it is off.

What remains is the judgment no rule makes: whether a large `export *` barrel with no cycle in it is a deliberate public interface or a directory listing pretending to be one. That question needs a reader.

## What it is
An `index.ts` that re-exports a directory's contents, used as the way in. It looks like encapsulation and behaves like a hub: importing one name from a barrel pulls in every module the barrel mentions, and every module *those* mention.

Where a barrel re-exports a module that imports from the same barrel, the cycle is complete and module initialisation order becomes something nobody specified.

## What it costs
**Cycles.** Barrels are the commonest source of circular imports in TypeScript, and the symptom is not a compile error - it is `undefined` where a class or constant should be, at import time, with a stack trace pointing at the use rather than the cycle.

**Load cost.** One named import reaches the whole graph behind the barrel. Bundlers can often tree-shake this; they frequently cannot when a module in the graph has side effects, and the result is a larger bundle or a test suite that loads the application.

**Lost structure.** When every module imports from `@/lib`, the dependency graph says everything depends on everything, and nobody can see which parts are actually coupled.

## Thresholds
- **Any import cycle** is Clear, barrel or not.
- **A barrel that re-exports a module which imports from that same barrel** is Clear - a cycle waiting for the wrong initialisation order.
- **A barrel re-exporting 20 or more modules** is worth a look: it is a directory, not an interface.
- **A barrel of barrels** - `index.ts` re-exporting other `index.ts` files - is Clear.
- **Importing from a barrel inside the directory that barrel covers** is Clear. A sibling should import its sibling directly.
- **A barrel that re-exports internals** alongside its public surface, so nothing marks the difference.

Rules that cover this, which this skill does not run: `import/no-cycle`, `import/no-self-import`, `import/no-useless-path-segments` from `eslint-plugin-import`.

## Judgment signals
- **`export * from`** rather than named re-exports. Nobody can tell what the module's surface is, including the people maintaining it.
- **A constant or class that is `undefined` at import time**, with an initialisation workaround nearby - a lazy getter, a deferred assignment, a `setTimeout`. Those are cycle symptoms treated at the symptom.
- **Types and runtime values mixed in one barrel**, where `import type` would have broken the cycle for free.
- **A test importing a barrel** and thereby booting half the application.
- **A deep import sitting next to a barrel import** of the same directory, which means someone already found the barrel unusable and worked around it.

## When it's fine
- **A package's public entry point.** `src/index.ts` defining what a library exports is the right use of the pattern - that is an interface, written deliberately.
- **Type-only barrels.** `export type { ... }` has no runtime cost and cannot form a runtime cycle.
- **A small, flat, curated barrel** - a handful of named re-exports over modules that do not import back. This is the working version of the pattern.
- **Framework-required files.** SvelteKit, Next.js and similar define the meaning of certain files; those are conventions, not barrels.
- **Generated index files.**

## Confidence guidance
**Clear** - an actual cycle, a barrel re-exporting a module that imports from it, a barrel of barrels, or an intra-directory import routed through the directory's own barrel.

**Worth a look** - a large `export *` barrel with no cycle visible, or a barrel mixing types with runtime values.

Name the cycle path where there is one - `a → index → b → a` - because the fix depends on which edge is easiest to cut. Without a path, the finding is a guess.

## Sources
- TypeScript Handbook, Modules - https://www.typescriptlang.org/docs/handbook/2/modules.html
- eslint-plugin-import, `no-cycle` - https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-cycle.md
