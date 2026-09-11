# Primitive Obsession in TypeScript

## Thresholds
There is no count here. The signal is a rule about a primitive enforced in **two or more places**.

- **Two or more validations of the same primitive** across the codebase.
- **Two `string` parameters** in one signature that mean different things.
- **A `string` field whose allowed values live in a comment** rather than in a type.

## How it shows up
- **Ids as bare strings.** `userId: string` and `accountId: string` are the same type to the compiler. Transposing them compiles and ships.
- **Money as `number`.** Floating point plus an implicit currency. `amountCents: number` puts the unit in the name because it is not in the type.
- **Dates as strings.** `createdAt: string` with `new Date(...)` parsing scattered wherever it is read, and the ISO format assumed everywhere.
- **Status as `string`.** The valid values known only by convention, so a typo is a runtime bug rather than a compile error.
- **Structured strings** - slugs, keys, paths, emails - that get `split`, `startsWith`, or a regex applied at several call sites.
- **`any` and `unknown` carried past the boundary** instead of being parsed into a type at the edge.

## TypeScript cases that are fine
TypeScript gives cheaper answers than a class, and code already using them is not a finding.

- **Literal unions.** `type Status = 'draft' | 'sent' | 'paid'` is a type. The compiler checks it exhaustively in a `switch`.
- **Branded types.** `type UserId = string & { readonly __brand: unique symbol }` makes two strings non-interchangeable at zero runtime cost. Look for this before reporting id-as-string.
- **A parsed boundary.** A zod or similar schema at the edge, converting primitives into typed values once, with the core trusting them afterwards. This is the design working.
- **Genuine primitives** - counts, ratios, free text, display names.
- **`Date`, `URL`, `Intl` types** already doing the job.
- **Scripts, migrations and short-lived code**, where a branded type is ceremony.
