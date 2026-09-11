# Feature Envy in TypeScript

## Thresholds
- **3 or more accesses to the same foreign object** in one function, where its own state is barely touched.
- **Foreign accesses outnumbering local ones** by a clear margin.
- **The same combination of another type's fields** computed in 2 or more places.

## How it shows up
TypeScript's structural typing and its preference for free functions make this smell look different from the classic OO version. Much of what would be Feature Envy in Java is idiomatic here.

- **Arithmetic over another object's fields.** `order.subtotal + order.tax - order.discount` computed in a component, a formatter and a report. That is `orderTotal(order)`, and it should live next to the order type.
- **A decision made on foreign state.** Reading three fields of `subscription` to decide whether it is active, in a file that is not about subscriptions.
- **A React component computing from props** what the domain should have provided - deriving status, totals or eligibility inline in JSX.
- **A util module that only ever touches one type.** `utils/orderHelpers.ts` operating exclusively on `Order` is the `Order` module, misfiled.
- **Repeated destructuring of the same object** to feed a calculation.

## TypeScript cases that are fine
- **Free functions over data types.** A codebase built on `type Order = {...}` plus module functions has no class to move the method into - moving means moving to the *module*, which is a filing decision, not a design flaw. Only report where the function is in the wrong module.
- **Mappers, serialisers, presenters and DTO converters.** Reading every field is the job. In a directory of these, this smell is the local design.
- **Validators and schemas** - zod, class-validator - which necessarily read the whole shape.
- **Redux selectors and reducers**, which exist to read state they do not own.
- **Coordinators** pulling from several collaborators to combine them.
- **Types owned by a library, generated client or database layer**, which cannot take a method.
