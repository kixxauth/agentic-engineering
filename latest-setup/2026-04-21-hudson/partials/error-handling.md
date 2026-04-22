# Error Handling Philosophy

When you write or refactor code in this project, apply the rules below.

## Two Categories of Errors

Every error is either **expected** or **unexpected**. The two categories are handled in completely different ways — do not blur them.

### 1. Expected Operational Errors

Run-time problems that a correctly-written program will eventually encounter. They are *not* bugs. They come from the outside world:

- Network failure, socket hang-up, DNS resolution failure.
- Remote service returned 5xx, 4xx, or timed out.
- Disk full, out of memory, too many open files.
- Invalid user input, invalid configuration.
- A requested resource does not exist.

**Rule:** Expected errors must be *caught, enriched with context, and rethrown* up the stack until a layer with enough context can decide what to do. Only the top-level caller (HTTP handler, CLI entry point) usually has that context.

Use the error classes in `lib/kixx-server-errors/mod.js` to represent expected errors:

- `OperationalError` — generic expected error (I/O, integration, etc.)
- `BadRequestError` (400), `UnauthenticatedError` (401), `ForbiddenError`/`UnauthorizedError` (403), `NotFoundError` (404), `MethodNotAllowedError` (405), `NotAcceptableError` (406), `ConflictError` (409), `UnsupportedMediaTypeError` (415), `ValidationError` (422), `NotImplementedError` (501).
- `WrappedError` — the base class. All of the above set `expected: true` by default and expose `httpStatusCode`, `code`, `name`, and a boolean `httpError`.

Every one of these classes accepts `cause` in its options object. **Always pass the original error as `cause`** when rethrowing, so the context chain is preserved:

```javascript
import { OperationalError, NotFoundError } from '../../lib/kixx-server-errors/mod.js';

try {
    return await fetchFromUpstream(id);
} catch (cause) {
    if (cause.code === 'ENOTFOUND') {
        throw new NotFoundError(`Upstream record ${ id } not found`, { cause });
    }
    throw new OperationalError(`Failed to fetch record ${ id }`, { cause });
}
```

### 2. Unexpected Errors (Programmer Errors / Bugs)

Cases the author did not think about. These cannot be "handled" because, by definition, the code is broken. Examples:

- Reading a property of `undefined`.
- Passing a string where an object was expected.
- Calling a method on a value that should have been a Date.
- An invariant that the surrounding code depends on is violated.

**Rule:** *Crash immediately.* Do not try to recover. A restarter (process supervisor, worker restart) is the correct response — continuing to serve other requests from a process in a broken state risks corrupting shared state (pools, sockets, caches) and returning wrong answers to other callers.

Do not catch-and-swallow. Do not convert a programmer error into an operational error. Let it propagate until the process crashes (or is logged and crashed at the top level).

## Assumptions → Assertions

The way we protect against unexpected errors is to write down our assumptions as **assertions** at the boundaries of our code. An assertion that fails is a loud, immediate crash with a precise message — exactly what you want for a bug.

Use `lib/kixx-assert/mod.js` for this. Pick the relative import path from the file you are editing.

### When to Assert

Assert every non-obvious assumption the function body depends on:

- **Public function entry points**: validate the *shape* of arguments your function needs to use. If you expect a non-empty string, call `assertNonEmptyString(id, 'id')`. If you expect a plain object, assert it.
- **After destructuring from an external source** (config files, JSON payloads, environment): assert that required fields are present and of the expected type.
- **Between layers in a pipeline**: when one module hands data to another, the receiving side can assert the contract.
- **Before operations where a wrong type would silently corrupt state** (writing files, constructing URLs, building SQL): assert first.
- **On values returned from framework/library calls** where your code assumes a specific shape that is not enforced by the type system.

Do **not** assert as a substitute for user input validation. User input is an **expected operational error** (use `ValidationError` / `BadRequestError`), not a programmer error. Assertions are for internal invariants between your own code units — values that a correct program would never produce.

### How to Assert

```javascript
import {
    assertNonEmptyString,
    assertFunction,
    assertPlainObject, // if available — otherwise use the matching is* + assert pattern
    isPlainObject,
    assert,
} from '../../lib/kixx-assert/mod.js';

function registerHandler(args) {
    const { name, handler, options } = args ?? {};

    assertNonEmptyString(name, 'registerHandler: name');
    assertFunction(handler, 'registerHandler: handler');
    assert(isPlainObject(options), 'registerHandler: options must be a plain object');

    // ... from here on, the function body can rely on these facts.
}
```

The message prefix is important. It should name the function and the parameter so that when the assertion fires in production logs, a maintainer can find the call site immediately.

### Prefer `assert*` Helpers Over Hand-Written Checks

Do not write:

```javascript
if (typeof name !== 'string' || name.length === 0) {
    throw new Error('name is required');
}
```

Write:

```javascript
assertNonEmptyString(name, 'context: name');
```

## Wrapping and Rethrowing

When you catch an expected error at a layer that cannot handle it, wrap it with more context and rethrow. Preserve the chain with `cause`:

```javascript
try {
    await loadWranglerConfig(cwd);
} catch (cause) {
    throw new OperationalError(
        `Failed to load wrangler config from ${ cwd }`,
        { cause },
    );
}
```

Rules for wrapping:

- **Always pass `cause`.** Losing the original error destroys the debugging trail.
- **Add information, do not repeat it.** The wrapper message should say what *this layer* was trying to do, not restate the inner message.
- **Do not wrap programmer errors.** If an `AssertionError` (from `kixx-assert`) or a native `TypeError` / `ReferenceError` flies past, let it propagate — it is a bug and should crash.
- **Pick the most specific class.** Use `NotFoundError` when something is missing, `ValidationError` when input fails validation, `BadRequestError` when a request is malformed. Fall back to `OperationalError` only when none fit.

## At the Top of the Stack

The top-level caller (HTTP handler, command entry point) is the one place that decides:

- If `error.expected === true` and `error.httpError === true`: render an HTTP response using `error.httpStatusCode`, `error.code`, and a safe message.
- If the error is unexpected: log the full chain (including `cause`) and let the process crash. Do not try to keep serving.

Commands in `commands/<command>/<subcommand>` should stay thin — they call into `app/` and let errors propagate. Only convert errors to user-facing output (exit codes, stderr messages, HTTP responses) at the outermost layer.

## Quick Checklist

When writing or refactoring code in this project, verify:

- [ ] Every function's assumptions about its arguments are expressed as `assert*` calls at the top of the body.
- [ ] Every `try/catch` either (a) handles the error meaningfully, or (b) rethrows a `kixx-server-errors` class with `{ cause }` set.
- [ ] No `catch` block swallows the error silently.
- [ ] No `catch` block converts a programmer error (TypeError, ReferenceError, AssertionError) into an operational error.
- [ ] Expected errors use the most specific error class available (`NotFoundError`, `ValidationError`, etc.).
- [ ] Error messages describe *what this layer was doing*, not just *what went wrong*.
- [ ] User input validation produces `ValidationError` / `BadRequestError`, not `AssertionError`.
