# Floating Promise
**Remedy:** Await the Promise · Handle the Rejection · Make the Fire-and-Forget Explicit

## Lint owns this one
`@typescript-eslint/no-floating-promises` and `no-misused-promises` decide this exactly, using type information, across every file - which is strictly better than reading. See `../lint-coverage.md`.

So: **where those rules are enabled, report no instances.** Say lint owns it and move on. Where they are not - and on plain `recommended` they are not, because they are type-aware - the finding is the missing gate, reported once, with the instance count as a floor rather than a total.

The rest of this file is for judging that gate gap and for the residue below.

## What it is
A promise nobody owns. The call was made, the promise was discarded, and whether it succeeded is now unknowable to the code that started it. If it rejects, the rejection goes to the runtime's unhandled-rejection path, which in Node terminates the process by default and in a browser prints to a console nobody is reading.

The missing `await` is the usual cause. Passing an `async` function where a synchronous callback is expected is the subtler one, because nothing looks wrong at the call site.

## What it costs
Two failures, and they look nothing alike. The work may not have finished when the caller continued - so state is read before it is written, a response is sent before the save completes, a test asserts before the effect lands. Or the work failed and nobody was told, so the system carries on with a silently incomplete operation.

Both produce bugs that reproduce intermittently and depend on timing, which makes them among the most expensive to chase.

## Thresholds
- **Any call to an `async` function whose result is discarded** is worth a look.
- **Inside an `async` function** it is Clear - the `await` was available and omitted.
- **An `async` function passed to something expecting `void`** - `forEach`, an event listener, `setTimeout`, a non-async framework hook - is Clear. The returned promise has nowhere to go.
- **`.then()` with no `.catch()` and no `await`** is Clear.
- **`void someAsyncCall()`** with no `.catch()` is worth a look: the discard is deliberate, the error path still is not.

Rules that cover this, which this skill does not run: `@typescript-eslint/no-floating-promises`, `@typescript-eslint/no-misused-promises`, `@typescript-eslint/await-thenable`, `@typescript-eslint/require-await`.

## Judgment signals
- **`items.forEach(async (item) => { await save(item) })`.** The canonical instance. `forEach` ignores every promise returned, so this returns before any save completes and swallows every failure. `for...of` with `await`, or `Promise.all(items.map(...))`, is the fix depending on whether order matters.
- **An `async` event handler or a fire-and-forget in a constructor.**
- **A logging, metrics or cache-write call left unawaited** because it "doesn't matter". Its rejection still reaches the runtime.
- **A `catch` block whose own async call is unawaited**, so the error handling can fail silently too.
- **A test that does not await the thing it is testing**, which passes regardless of the result.
- **`async` with no `await` in the body.** Not this smell, but usually next to it: the function's signature promises asynchrony it does not have, and callers may have stopped awaiting it.

## When it's fine
- **An explicit, handled discard.** `void track(event).catch(reportError)` - the intent is written down and the failure has somewhere to go.
- **A promise stored and awaited later.** `const pending = fetchAll(); ...; await pending` is not floating; it is deliberate concurrency.
- **A top-level entry point** - a script's main, a server bootstrap - where the runtime's failure behaviour is the intended behaviour, and a `.catch` that logs and exits is usually still better.
- **A framework that takes ownership.** Some routers, queues and test runners await what you return; returning the promise is correct and no `await` is needed.
- **A deliberately detached background task** with its own error handling inside.

## Confidence guidance
**Clear** - a discarded promise inside an `async` function, an `async` callback passed where `void` is expected, or `.then()` with no error path.

**Worth a look** - a discarded promise at a top-level entry point, a `void`-marked call with no `.catch`, or a case where a framework might be taking ownership.

Say which of the two failure modes applies - ordering, or swallowed error - because they need different answers. Check whether the surrounding framework awaits return values before reporting.

## Sources
- typescript-eslint, `no-floating-promises`, `no-misused-promises` - https://typescript-eslint.io/rules/no-floating-promises/
- Node.js unhandled rejection behaviour - https://nodejs.org/api/cli.html#--unhandled-rejectionsmode
