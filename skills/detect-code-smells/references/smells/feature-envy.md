# Feature Envy
**Refactoring:** Move Function · Extract Function · Move Field · Preserve Whole Object

## What it is
A function more interested in another object's data than in its own. It reaches across, pulls out several fields, does arithmetic or decision-making on them, and returns a result that really belonged to the object it was reading from.

## What it costs
The logic sits away from the data it operates on, so a change to that data's rules has to be made in a foreign file - and the person changing the data has no way to know the logic exists. It also tends to duplicate: two callers both envy the same object and both write their own slightly different version of the calculation.

## Mechanical signals
- Count of accesses to another object's members within one function, against accesses to its own.
- Several fields pulled from the same object in one expression.
- A function using none of its own object's state.
- The same combination of another object's fields appearing in more than one place.

## Judgment signals
- **Arithmetic or comparison over foreign fields.** `order.subtotal + order.tax - order.discount` in a formatter is the order's total, computed somewhere else.
- **A decision made on another object's state.** Reading three fields to decide whether something is eligible, active or overdue - a question that object should be able to answer.
- **Repeated prefix.** Every line mentioning the same receiver.
- **The obvious home is available.** The target is a type in this codebase that could hold the method.

## When it's fine
- **The target is not yours to change** - a library type, a generated client, a database row.
- **The pattern is deliberate.** Visitors, serialisers, presenters, validators and mappers exist precisely to keep foreign knowledge out of the domain object. A JSON serialiser reading ten fields is doing its job; moving that into the domain type would put transport concerns into the model.
- **Coordination across several objects.** A function that reads from three collaborators to combine them belongs where it is, even though it uses no state of its own.
- **The target is a deliberately data-only type**, with behaviour held elsewhere by design.
- **Moving it would create a worse dependency** - pointing the domain at the UI, or a low-level type at a high-level one.

## Confidence guidance
**Clear** - a function whose accesses are mostly to one other object, performing a calculation or decision that object could own, where that object is modifiable and is not a deliberate data carrier.

**Worth a look** - heavy foreign access that might be a mapper, a presenter, or a coordinator, or where the target may not be modifiable.

Check the surrounding file's purpose before reporting. In a directory of mappers this smell is the local design, and reporting every function in it is noise.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Feature Envy"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/feature-envy
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
