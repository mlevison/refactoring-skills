# Long Parameter List in TypeScript

## Thresholds
- **4 or more positional parameters**, look closer.
- **2 or more adjacent parameters of the same type** is the real risk, at any count - TypeScript cannot catch a transposed pair of `string`s.
- **An object parameter with 8+ properties** is the same smell wearing a different hat, and destructuring it in the signature does not change that.

## How it shows up
- **Positional same-type runs.** `fn(userId: string, accountId: string, regionId: string)` - three strings, any order compiles.
- **Optional tails.** `fn(a, b?, c?, d?)` with callers writing `fn(a, undefined, undefined, true)`. The optionals are a signature that stopped fitting.
- **An `options` bag that is really two concepts.** One object holding both what to fetch and how to render it.
- **Props drilling.** A React component taking twelve props and passing eight of them straight down. Those eight are one object, or they belong in context.
- **A constructor taking every dependency separately** where several always arrive together.
- **Passing pieces of an object the callee could take whole** - `user.id, user.email, user.locale` rather than `user`. Preserve Whole Object, unless the narrower signature is a deliberate dependency reduction.

## TypeScript cases that are fine
- **A single object parameter with named properties.** `createInvoice({ customer, lines, issuedOn })` - callers name every argument, so the positional risk is gone and the readability cost with it. This is the idiomatic fix, and an already-converted signature is not a finding.
- **Destructured object parameters in the signature**, for the same reason.
- **Branded or distinct types on adjacent parameters.** `UserId` and `AccountId` as branded types cannot be transposed; the compiler catches it.
- **React props.** A component with many props is often honest - the alternative is an object that means nothing. Look for props that always travel together instead.
- **Overload signatures**, and framework or library callback shapes.
- **Curried or partially applied functions**, where the staging is the design.
