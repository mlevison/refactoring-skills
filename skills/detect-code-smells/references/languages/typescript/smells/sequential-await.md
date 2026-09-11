# Sequential Await
**Remedy:** Parallelise with `Promise.all` · Collect with `Promise.allSettled`

## Why lint cannot settle this one
ESLint's `no-await-in-loop` finds every `await` in a loop - including all the ones that are correct. It cannot tell whether iteration *n* needs iteration *n-1*, which is the entire question, so enabling it on a real codebase produces mostly false positives and then gets disabled.

That makes this a reading job, not a gate. Judging independence is the work; finding the loop is not. See `../lint-coverage.md`.

## What it is
`await` inside a loop, where each iteration's work does not depend on the last. The code waits for the first request before starting the second, so total time is the sum of every wait rather than the longest one.

A hundred items at 50ms each is five seconds of doing nothing, where it could be 50ms.

## What it costs
Latency that scales with input size and is invisible in development, where the collection has three items and the network is local. It appears in production, under real data, as a timeout - and the code looks perfectly reasonable, which is why it survives review.

Where the awaited call is a database query, it is also an N+1: the loop is asking the database the same question repeatedly instead of once.

## Thresholds
- **Any `await` inside `for`, `for...of`, `for...in`, `while`, or a `.reduce` accumulating promises**, where the awaited call does not use the previous iteration's result - worth a look at minimum.
- **Over a collection whose size comes from input or a query** - Clear. The cost is unbounded.
- **Two or more independent `await`s in sequence** outside a loop - `const a = await getA(); const b = await getB();` where `b` does not need `a`. Smaller, very common, and free to fix.
- **A loop body whose only statement is an `await`** - Clear, and usually a missing `Promise.all`.

Rules that cover this, which this skill does not run: ESLint's core `no-await-in-loop`.

## Judgment signals
- **Independence is visible.** The loop variable is the only thing that changes between iterations, and nothing from iteration *n* is read in *n+1*.
- **The awaited call is a query.** `for (const id of ids) { users.push(await findUser(id)) }` is N+1, and the better fix may be one query rather than parallel queries.
- **Results pushed into an array** and used only after the loop - exactly what `Promise.all(ids.map(...))` produces.
- **Unbounded collection size.** `await` in a loop over a paginated result set or a user-supplied list.
- **Sequential independent awaits** at the top of a function, often a handler gathering several things before it can respond.

## When it's fine
These are the cases where sequence is the point, and reporting them is noise.

- **Each iteration depends on the last** - pagination by cursor, a retry with backoff, a state machine step, anything consuming the previous result.
- **Order of effects matters.** Writes that must land in sequence, migrations, an append-only log.
- **Deliberate rate limiting.** Sequential calls to an API with a quota, or to avoid exhausting a connection pool. Parallelising here trades a slow function for a failing one - and with a large collection, `Promise.all` on unbounded concurrency is a worse bug than the latency it fixed.
- **A bounded, tiny collection** where the latency is irrelevant and the sequential version reads better.
- **Transactional work** that must stay inside one connection.
- **Scripts and migrations**, where wall-clock time does not matter and a predictable order helps when one fails.

## Confidence guidance
**Clear** - independent awaits in a loop over a collection whose size is not fixed and small, with nothing in the body suggesting rate limiting or ordering.

**Worth a look** - a loop over a small fixed collection, two independent sequential awaits outside a loop, or a case where a quota or a connection limit might be the reason.

Name the dependency you checked for and did not find, and note the concurrency limit question: the remedy for a thousand-item loop is a bounded pool, not `Promise.all`.

## Sources
- ESLint, `no-await-in-loop` - https://eslint.org/docs/latest/rules/no-await-in-loop
- MDN, `Promise.all` and `Promise.allSettled` - https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
