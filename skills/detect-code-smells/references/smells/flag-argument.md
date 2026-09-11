# Flag or Boolean Argument
**Refactoring:** Remove Flag Argument · Extract Function · Replace Parameter with Query

## What it is
A parameter whose job is to select which behaviour the function performs. `render(report, true)` tells the reader nothing at all; they have to open the function to learn what `true` meant.

The flag need not be a boolean. An enum or a string used to switch between whole behaviours - not to configure one behaviour - is the same smell.

## What it costs
The call site stops being readable. `true` and `false` carry no meaning at the point where a reader most needs it, so understanding any caller means reading the callee too.

Underneath that, the flag is usually evidence that one function is doing two jobs. The body has a branch near the top, and the two arms share little. Every subsequent change has to be made with both arms in mind, and it is easy to change one and forget the other.

## Mechanical signals
- Boolean literals at call sites.
- A parameter used only as the condition of a top-level branch in the body.
- Multiple flags on one signature - two booleans mean four behaviours, and usually only two of them were ever intended.

## Judgment signals
- **The branch is the whole function.** `if (flag) { ...one thing... } else { ...another... }` with nothing shared but the signature.
- **The name reveals it.** Parameters called `isX`, `shouldY`, `force`, `dryRun`, `verbose` used to pick behaviour rather than to configure it.
- **Callers cluster.** Every caller in one file passes `true`, every caller in another passes `false`. Those are two functions with two sets of users.

## When it's fine
- **A genuine configuration flag that does not branch behaviour**, only adjusts it - a formatting option threaded through to a library, for instance.
- **A named argument at every call site.** Where the language supports it, `render(report, { includeDrafts: true })` restores the meaning the bare literal lost. The two-jobs problem may still be there, but the readability cost is gone.
- **A boundary the code does not control** - implementing an interface, a callback signature, or an API defined elsewhere.
- **The value is data, not a switch.** A boolean passed through and stored, never branched on, is a field being set.

## Confidence guidance
**Clear** - a boolean parameter that selects between two behaviours in the body, passed as a bare literal at call sites.

**Worth a look** - a flag that configures rather than switches, or one always passed as a named argument, or one whose two branches share most of their work.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Remove Flag Argument"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/flag-argument
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/) - which notes boolean flags are common enough to need more than one source to shift
- [Are Boolean Flags on Methods a Code Smell?](https://ardalis.com/are-boolean-flags-on-methods-a-code-smell/) - Steve Smith
- [Clean code: the curse of a boolean parameter](https://medium.com/@amlcurran/clean-code-the-curse-of-a-boolean-parameter-c237a830b7a3)
