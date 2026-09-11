# Enum Over Union
**Remedy:** Replace Enum with Literal Union

## What it is
A TypeScript `enum` where a literal union would do the same job. `enum Status { Draft = 'draft', Sent = 'sent' }` against `type Status = 'draft' | 'sent'`.

The union is checked identically, disappears at compile time, and needs no import to write a value. The enum emits a runtime object, has to be imported wherever a member is named, and introduces nominal behaviour in a language that is otherwise structural.

This is the weakest of the TypeScript-native smells, and the one most likely to be a deliberate house choice. Report it at most once per file, and never as a reason to rewrite working code.

## What it costs
**A runtime artifact for a compile-time idea.** The enum object exists in the bundle, and cannot be fully erased.

**An import for every use.** `status === Status.Draft` requires importing `Status`; `status === 'draft'` requires nothing. Over a codebase this is hundreds of imports carrying no information, and it makes the enum's module a hub.

**Surprising assignability.** A numeric enum accepts any number at some positions, which is the opposite of what the declaration looks like it promises. A string enum is nominal - a matching string literal is not assignable to it - which surprises everyone exactly once per codebase.

**`const enum` is worse**: it breaks under `isolatedModules`, which most modern toolchains require.

## Thresholds
- **Any numeric `enum`** is worth a look - the implicit values are positional, so reordering members changes the data.
- **Any `const enum`** is Clear under `isolatedModules` or a transpile-only pipeline (esbuild, swc, Vite, Babel).
- **A string enum whose members map to their own names** - `Draft = 'Draft'` - is worth a look: ceremony for no gain.
- **An enum used only in type positions**, never as a value, is Clear - nothing needs the runtime object.
- **An enum crossing a serialisation boundary** - written to a database, a URL or JSON - is worth a look, since the stored value is a string or number and the enum adds a translation layer over it.

Rules that cover this, which this skill does not run: `@typescript-eslint/prefer-literal-enum-member`, `@typescript-eslint/no-unsafe-enum-comparison`, `@typescript-eslint/prefer-enum-initializers`.

## Judgment signals
- **Only ever compared, never enumerated.** No `Object.values(Status)`, no iteration - so the runtime object is unused.
- **String members equal to their keys.**
- **A union elsewhere for the same concept.** The codebase has both, so the convention is already mixed and the enum is the outlier.
- **Assertions to get in or out of it.** `value as Status` on a string from an API, which is what nominal string enums force.
- **Mixed initialised and uninitialised members**, where some values are explicit and the rest are implicit numbers.

## When it's fine
- **The runtime object is used.** Iterating members to build a dropdown, validate input, or seed a database needs a real object. A union needs a separate `as const` array; an enum already has one.
- **Reverse mapping.** Numeric enums map value back to name, which a union cannot do.
- **A declared house convention.** Where a codebase uses enums consistently, consistency is worth more than this smell. Say it once; do not report every enum.
- **Generated code** - Prisma, GraphQL codegen, protobuf.
- **A shared contract with non-TypeScript consumers**, where the enum mirrors something defined elsewhere.
- **`enum` as a nominal brand**, where preventing a bare string from being assignable is the point.

## Confidence guidance
**Clear** - a `const enum` in a transpile-only pipeline, or an enum used only in type positions with no runtime use anywhere in the target.

**Worth a look** - a numeric enum, a string enum mirroring its keys, or an enum crossing a serialisation boundary.

Check whether the enum object is used at runtime before reporting. Where the codebase uses enums throughout, make the observation once for the target and move on - this smell is a preference, and a report full of it reads as pedantry and will get the skill turned off.

## Sources
- TypeScript Handbook, Enums - https://www.typescriptlang.org/docs/handbook/enums.html
- TypeScript docs on `isolatedModules` and `const enum` - https://www.typescriptlang.org/tsconfig#isolatedModules
- Dan Vanderkam, _Effective TypeScript_, 2nd edition, "Prefer Unions of Literals to Enums"
