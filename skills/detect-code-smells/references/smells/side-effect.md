# Side Effect
**Refactoring:** Separate Query from Modifier · Change Function Declaration · Extract Function · Encapsulate Variable

## What it is
A function that does more than its name promises. The glossary this catalogue descends from puts it exactly: *"a `setAmount()` that changes a date field."*

The commonest form is a function that looks like a question but is an instruction: something named `get`, `find`, `check` or `is` that also writes, caches, logs, mutates an argument or advances state.

## What it costs
Callers reason from names. A function whose name says less than it does breaks that reasoning in a way that is invisible at the call site, and the resulting bugs are found far from their cause: state changed twice because someone called the getter twice, or never changed because someone removed a call that looked pure.

It also makes the function unsafe to move, cache, reorder, retry or test, and none of those constraints are written down anywhere.

## Mechanical signals
- Assignment to fields, module-level state or captured variables inside a function whose name suggests a query.
- Mutation of a parameter.
- I/O, logging or network calls inside something named as a computation.
- A function that both returns a value and writes state.
- Name prefixes - `get`, `is`, `has`, `find`, `calculate`, `format`, `validate` - over a body that writes.

## Judgment signals
- **The name is narrower than the body.** `setAmount` writing an audit timestamp. `validateOrder` also saving it.
- **A mutated argument.** The caller passes a collection in and gets it back changed, with nothing in the signature saying so.
- **Order dependence.** Calling two functions in the other order gives a different result, and nothing announces that.
- **Lazy initialisation in a getter.** Common and often intended, but it makes the first call different from the rest.

## When it's fine
Each of these excuses one specific effect, not every effect in the function. A function that qualifies for one of them can still be hiding a second effect that none of them covers, and that second effect is the finding.

- **The name says so.** `saveAndNotify`, `fetchAndCache`, `applyDiscount` - a function that announces both jobs is not hiding one. It is still hiding a third.
- **A command whose effects are the ones its name implies.** `deleteInvoice` writing to the invoice table is its job. `deleteInvoice` also decrementing a counter on the customer is not, and "it's a command" does not cover it.
- **Intentional, documented memoisation**, where the cache is invisible to correctness.
- **A builder or accumulator**, where mutation is the pattern and the type says so.
- **Instrumentation.** Logging, metrics and tracing are side effects everywhere and are not usually this smell - unless the logging call is also doing work.

This section does **not** excuse:
- **A service layer.** Services exist to cause effects, which makes the gap between name and body easier to hide, not less important. "It's a service" is not a counter-case.
- **A mutable codebase.** That a file's neighbours mutate freely says nothing about whether *this* function's name tells the truth. The smell is the gap, not the mutation, so a permissive baseline cannot close it.

## Confidence guidance
**Clear** - a query-named function that writes state a caller would not expect, or a function that mutates a parameter without saying so.

**Worth a look** - lazy initialisation, logging inside a computation, or a function whose extra effect is arguably implied by its name.

The test is the gap between the name and the body, not the presence of an effect. Say what the name promised and what the body also did.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Separate Query from Modifier"; Command-Query Separation, Bertrand Meyer
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/side-effects
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
