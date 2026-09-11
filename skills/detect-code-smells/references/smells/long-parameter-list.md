# Long Parameter List
**Refactoring:** Introduce Parameter Object · Preserve Whole Object · Replace Parameter with Query · Combine Functions into Class

## What it is
A function taking so many arguments that calling it correctly requires checking the signature every time. The same human limit as Long Method applies: a caller has to hold every argument and its position in mind at the call site.

## What it costs
Callers make positional mistakes that the language may not catch, particularly where several parameters share a type. Every new requirement adds another parameter, so the signature grows and every existing caller changes with it. And a long list is usually evidence of a second problem: the parameters that always travel together are a concept nobody has named.

## Mechanical signals
- Count of parameters.
- Count of parameters sharing a type, which is where positional errors live.
- The same group of parameters appearing in more than one signature - that is a Data Clump, and it is the reason this smell is worth fixing rather than tolerating.
- Parameters that are only passed through to another call untouched.

## Judgment signals
- **Parameters that always travel together.** Three of them appear in this order in four functions. They are an unnamed object.
- **Passing pieces of something the callee could take whole.** Handing over a start date and an end date pulled out of a booking, when the function could take the booking.
- **A parameter derivable from another.** Passing both a collection and its length, or an object and one of its fields.
- **Booleans and nulls at the call site.** Callers writing `null, null, true` are describing a signature that does not fit what they need.

## When it's fine
- **A constructor or factory assembling a value.** Something has to take the pieces. Grouping them into an object is only progress if that object means something.
- **Genuinely independent parameters.** A function taking four unrelated things, all required, none derivable from the others, is honest about its inputs. Bundling them into an argument bag named `options` hides the requirement without removing it.
- **Named or keyword arguments at every call site.** Where the language lets callers name each argument, the positional risk is gone and the cognitive cost is much lower.
- **An interface fixed by something outside the code** - a framework signature, a protocol, an ABI.

## Confidence guidance
**Clear** - a long list where a subgroup recurs across signatures, or where the caller could pass one object it already holds.

**Worth a look** - a long list of parameters with no visible grouping, or a constructor whose object might legitimately have that many pieces.

A count on its own is Worth a look at best. The finding is the unnamed concept, not the arithmetic.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Long Parameter List"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/long-parameter-list
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
