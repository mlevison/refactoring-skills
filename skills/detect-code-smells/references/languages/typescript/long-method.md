# Long Method in TypeScript

## Thresholds
Rules of thumb, not rules. The neutral file's test - what a person can hold in mind - still decides.

- **Past ~30 lines of function body**, look closer.
- **Past ~60 lines**, the legitimate cases are the exception rather than the rule.
- **Nesting past 3 levels** matters more than length.

## How it shows up
- **React components.** A component mixing data fetching, derived state, event handlers and a large JSX tree. The usual extraction is a custom hook for the logic, leaving the component as markup, rather than splitting the JSX.
- **JSX is not body.** A component that is 80 lines of which 60 are a flat JSX tree is a template, not a long function. Count the logic.
- **`useEffect` bodies** that grew into several unrelated concerns. Two effects, each with its own dependency array, read better than one doing both.
- **Express and API handlers** that validate, authorise, query, transform and respond in one function. Those are five phases, and Split Phase applies.
- **`async` functions** where each `await` begins a new stage, each with its own error handling.
- **Promise chains with long inline callbacks**, where each `.then` holds a block.

## TypeScript cases that are fine
- **Discriminated-union `switch`es** with one short arm per variant, including the `never` exhaustiveness check. Length here buys compiler-verified completeness.
- **Type guards and parsers** at a boundary - zod schemas, validation functions, DTO mappers. Long, flat, no branching.
- **Configuration objects and route tables** exported as one declaration.
- **Tests.** `describe` blocks are long by nature; judge the individual `it`, and remember that extracting the arrange step into a helper often hides what the test is proving.
- **Generated files** - `*.d.ts`, API clients, Prisma output, GraphQL codegen.
