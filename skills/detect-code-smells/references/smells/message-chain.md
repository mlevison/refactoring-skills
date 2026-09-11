# Message Chain
**Refactoring:** Hide Delegate · Move Function · Extract Function

## What it is
A caller navigating through a sequence of intermediaries to reach what it actually wants: `order.getCustomer().getAddress().getCountry().getCode()`. To get one value, the caller has been made to know about four types and the relationships between them.

## What it costs
Every link is a dependency the caller did not ask for. Change any relationship in the middle of the chain - the address moves onto the account, the country becomes a region - and every caller that walked through it has to change, even though none of them cared about the middle.

The caller is also coupled to a shape it has no business knowing. It wanted a country code; it was made to learn the object graph.

## Mechanical signals
- Length of the call chain, counting navigations rather than transformations.
- The same chain prefix appearing in more than one place.
- A chain crossing a module or package boundary partway along.
- Chains in which the intermediate values are never otherwise used.

## Judgment signals
- **The intermediates are strangers.** The caller names types it has no other dealings with.
- **The chain repeats.** The same walk, or its prefix, in several methods. That is a method waiting to be added to the object at the head of the chain.
- **Temporary variables hiding the chain.** Splitting it across three assignments makes it look shorter without reducing what the caller must know.
- **Only the end of the chain is used.** Nothing between the first call and the last is needed for its own sake.

## When it's fine
- **A fluent interface or builder.** `query.select(...).where(...).limit(10)` is a chain by design, and each call returns the same conceptual thing rather than navigating deeper.
- **A transformation pipeline.** `items.filter(...).map(...).reduce(...)` chains operations, not relationships. Nobody is being made to learn an object graph.
- **Optional-chaining over one nullable step**, where the alternative is nested null checks that read worse.
- **Inside the object graph's own module**, where the types are meant to know each other.
- **Configuration and test setup**, where the shape is being deliberately described.

## Confidence guidance
**Clear** - three or more navigation steps through distinct domain types, the intermediates unused, and the chain appearing more than once.

**Worth a look** - a shorter navigation, or one whose intermediates are used, or one that stays inside a single module.

Distinguish navigation from transformation before reporting anything. A long pipeline is not this smell, and reporting it as one will get the skill dismissed.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Message Chains"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/message-chain
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
