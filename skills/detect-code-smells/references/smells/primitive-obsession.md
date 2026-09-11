# Primitive Obsession
**Refactoring:** Replace Primitive with Object · Replace Type Code with Subclasses · Introduce Parameter Object

## What it is
A bare string, number or boolean standing in for a domain concept that has rules of its own. An email address is not a string: it has a shape, it can be invalid, and comparing two of them is not the same as comparing two strings. Every place that treats it as a string has to remember that.

## What it costs
The rules end up scattered. Validation appears at four call sites and is subtly different at each, because nothing forces them to agree. Two values of the same primitive type that mean entirely different things - a user id and an account id, both strings - can be swapped by mistake and nothing objects.

Money is the canonical case: an amount without a currency is not a quantity, and adding two of them is a bug the type system was never asked to catch.

## Mechanical signals
- Primitive-typed parameters and fields whose names carry the concept the type does not: `emailString`, `userId`, `amountCents`, `isoDate`.
- Validation or parsing of the same primitive in more than one place.
- Constants, prefixes or format strings that encode meaning into a primitive.
- Units in a name - `Cents`, `Ms`, `Kg` - which is a type written in English because it was not written in code.

## Judgment signals
- **The same rule enforced repeatedly.** A regex, a range check, or a normalisation applied wherever the value shows up.
- **Adjacent values that must stay together.** An amount and its currency, a value and its unit, a latitude and a longitude, passed as separate primitives.
- **A string with structure.** A value that is split, parsed, prefixed or pattern-matched is a type wearing a string as a disguise.
- **A set of allowed values enforced by convention.** A status held as a string, with the valid options recorded only in a comment or in the reader's memory.

## When it's fine
- **A primitive that is genuinely a primitive.** A count, a ratio, a name with no rules attached.
- **At a system boundary.** JSON, HTTP and SQL deal in primitives, and code at the edge must too. The question is whether the primitive is converted just inside the boundary or carried all the way to the core.
- **The rules really do live in one place.** A single validator, called once at entry, with the value trusted afterwards, is a workable design even without a type.
- **A small script or a short-lived spike**, where the ceremony would outweigh the value.
- **The language already narrows it.** A union of literal types, an enum, or a branded type is a type - it does not have to be a class.

## Confidence guidance
**Clear** - the same rule enforced in more than one place for the same primitive, or two primitives that must move together being passed separately.

**Worth a look** - a name suggesting a concept, but the rules are enforced in only one place, or the value is genuinely at a boundary.

A naming pattern on its own is not enough. The finding is the scattered rule, not the name.

## Sources
- Martin Fowler, _Refactoring_, 2nd edition, "Primitive Obsession"
- Jerzyk & Madeyski, https://doi.org/10.1007/978-3-031-25695-0_24 - https://www.codesmells.org/smells/primitive-obsession
- [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)
