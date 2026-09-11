# Side Effect in TypeScript

## Thresholds
No count. The measure is the gap between what the name promises and what the body does.

- **Any write to state inside a function named `get*`, `is*`, `has*`, `find*`, `calculate*`, `format*` or `select*`.**
- **Any mutation of a parameter** not declared `readonly`.
- **Any I/O, `fetch`, `console` or storage write** inside something named as a computation.

## How it shows up
- **Mutating an argument.** `function normalize(items: string[]) { items.sort(); ... }` - `sort` and `reverse` mutate in place. So do `push`, `splice` and `Object.assign` onto a parameter. The caller's array changed and the signature never said so.
- **A getter that writes.** Lazy initialisation, cache population, or an access counter inside `get value()`.
- **Module-level mutable state** written from a function that reads as pure. A `let` at module scope is the TypeScript form of global data.
- **`useEffect` doing two unrelated jobs**, or a render body that mutates a ref or a module variable.
- **A reducer that mutates.** Writing to `state` directly outside Immer's producer.
- **`async` functions that both return a value and write to a store**, where the write is the part callers do not expect.
- **Validation that also saves.** `validateOrder(order)` returning errors and persisting the valid ones.

## TypeScript cases that are fine
- **The name says both.** `saveAndNotify`, `fetchAndCache`, `applyDiscount`.
- **Commands** returning `void` or a receipt - their purpose is the change.
- **Immer producers and builder patterns**, where mutation is the declared idiom.
- **`useEffect` and `useRef`**, which exist to hold effects and mutable state. The smell is an effect in the wrong place, not an effect.
- **Logging, metrics and tracing**, unless the logging call is also doing work.
- **A parameter typed `readonly` or `Readonly<T>`** and honoured - the contract is written down and the compiler checks it.

A `*Service.ts` file is not a counter-case. A service layer is where effects belong and therefore where an undeclared one is hardest to spot: `ExpenseService.update` also writing an audit row, recalculating a cached total, or firing a notification is the smell whether or not the file is called a service.
