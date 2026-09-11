# Refactoring Names
Names come from Martin Fowler, _Refactoring_, 2nd edition (2018). Where the 1st edition used a different name, it is given as an alias, because the older name is the one most search results return and the one most people learned.

A finding names the refactoring and stops there. Performing it is a separate job.

| Refactoring | What it does | 1st edition alias |
| --- | --- | --- |
| Extract Function | Move a fragment into its own named function | Extract Method |
| Parameterize Function | Merge near-identical functions by passing the difference as an argument | Parameterize Method |
| Pull Up Method | Move a method duplicated in siblings to their common parent | |
| Inline Function | Replace a call with the body, where the body says it better than the name | Inline Method |
| Extract Variable | Name an intermediate value so the expression reads | Introduce Explaining Variable |
| Change Function Declaration | Rename a function, or change its parameters | Rename Method, Add/Remove Parameter |
| Combine Functions into Class | Gather functions that share data into a class | |
| Split Phase | Separate two stages that got mixed into one function | |
| Introduce Parameter Object | Replace a recurring group of parameters with an object | |
| Preserve Whole Object | Pass the object rather than several values pulled out of it | |
| Replace Parameter with Query | Derive a parameter inside the function instead of passing it | Replace Parameter with Method |
| Remove Flag Argument | Split a function whose boolean parameter selects behaviour | Replace Parameter with Explicit Methods |
| Replace Primitive with Object | Give a bare string or number a type that carries its rules | Replace Data Value with Object |
| Replace Type Code with Subclasses | Replace a type flag with distinct types | |
| Introduce Special Case | Replace repeated checks for a special value with an object that handles it | Introduce Null Object |
| Decompose Conditional | Extract the condition and each branch into named functions | |
| Consolidate Conditional Expression | Merge conditions that lead to the same result | |
| Replace Nested Conditional with Guard Clauses | Return early on the exceptional cases, leaving the main path unindented | |
| Replace Conditional with Polymorphism | Let types carry the branching | |
| Move Function | Move a function to the class or module whose data it uses | Move Method |
| Move Field | Move data to where it is actually used | |
| Hide Delegate | Give the client one call instead of a chain through intermediaries | |
| Remove Middle Man | Let the client talk to the delegate directly | |
| Encapsulate Variable | Route access to shared data through functions | Encapsulate Field |
| Encapsulate Collection | Return a copy or a read-only view instead of the live collection | |
| Separate Query from Modifier | Split a function that both returns a value and changes state | |
| Extract Class | Split a class carrying two responsibilities | |
| Extract Superclass, Extract Interface | Name what two types share | |
| Slide Statements | Move related lines together before extracting them | |
| Split Loop | Separate two jobs sharing one loop | |
| Replace Loop with Pipeline | Express a loop as map/filter/reduce | |
| Split Variable | Give each use of a reused variable its own name | |
| Rename Variable, Rename Field | Say what it is | |

Full catalogue: https://refactoring.com/catalog/
