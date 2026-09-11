# Flag or Boolean Argument in TypeScript

## Thresholds
- **Any bare `true` or `false` literal at a call site** is worth a look.
- **Two or more boolean parameters** on one signature is Clear territory - four behaviours from one function, and usually only two were intended.
- **A `string` parameter used only in a `switch`** that selects whole behaviours is the same smell.

## How it shows up
- **`getUser(id, true)`** where `true` meant `includeDeleted`. Nothing at the call site says so.
- **`isX` / `shouldY` / `force` / `dryRun`** parameters whose only use is the condition of a top-level `if`.
- **Optional booleans defaulting to `false`** - `fn(a, b = false)` - where passing `true` takes a different path through the body.
- **React props that switch rendering wholesale.** `<Button isLink>` that returns an `<a>` instead of a `<button>` is two components.
- **A boolean that should be a discriminated union.** `render(data, isCompact)` where the two modes take different data is `render({ mode: 'compact', ... } | { mode: 'full', ... })`.

## TypeScript cases that are fine
- **A named property on an options object.** `getUser(id, { includeDeleted: true })` restores the meaning the bare literal lost. The two-jobs problem may remain, but the readability cost does not - report this as Worth a look at most.
- **A literal union instead of a boolean.** `setSort('asc' | 'desc')` is not a flag argument; it is a value with a name.
- **React boolean props that configure rather than switch** - `disabled`, `readOnly`, `autoFocus`. These match the DOM and the ecosystem, and changing them would be worse.
- **Interface, callback and library signatures** the code does not own - `Array.prototype.sort` comparators, `addEventListener` options, test framework hooks.
- **A boolean stored as data**, threaded through and written to state without ever being branched on.
