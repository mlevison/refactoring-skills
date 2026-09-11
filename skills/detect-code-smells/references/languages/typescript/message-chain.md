# Message Chain in TypeScript

## Thresholds
- **3 or more navigation steps** through distinct domain types.
- **The same chain, or its prefix, in 2 or more places.**
- **`?.` repeated 3 or more times in one expression** - `a?.b?.c?.d` is a chain admitting it might not exist at any point.

Count navigations, not method calls. This is the distinction that decides most findings.

## How it shows up
- **`order.customer.address.country.code`** - four types to fetch one string.
- **Optional-chaining sprawl.** `session?.user?.profile?.settings?.theme`. Each `?.` is a step the caller was made to know about, and a place the shape could change.
- **A chain split across temporary variables.** Three assignments walking the same path is the same coupling, just less visible.
- **Reaching through a store or context.** `state.entities.users.byId[id].preferences.locale` in a component. A selector hides the shape; the component should not know it.
- **Deep imports.** `import { x } from '../../../billing/internal/detail/thing'` is the module-level version of the same problem.
- **Chained getters on an ORM entity**, walking relations to reach a scalar.

## TypeScript cases that are fine
These are chains of *operations*, not navigations through an object graph. Reporting them will get the skill switched off.

- **Array pipelines.** `items.filter(...).map(...).reduce(...)`.
- **Promise chains.** `fetch(...).then(...).then(...).catch(...)`.
- **Fluent builders and query builders.** `qb.select(...).where(...).orderBy(...)` - each call returns the same conceptual thing.
- **Test and assertion DSLs.** `expect(x).toHaveBeenCalledWith(...)`, `z.string().email().optional()`.
- **A single `?.` over one genuinely nullable step**, where the alternative is nested null checks.
- **Navigation within the module that owns those types**, where knowing the shape is the point.
