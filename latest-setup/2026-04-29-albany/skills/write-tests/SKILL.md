---
name: write-tests
description: >
  Write automated tests for this project using the kixx-test framework and
  kixx-assert assertion library. Use when asked to add tests, write a test file,
  cover a module with tests, or verify behavior with assertions. Test files live
  in test/ and must end with test.js. Run with: node test.js (all) or
  node test.js test/path/to/file-test.js (single file).
argument-hint: "[file or module to test]"
---

Write tests for this project following the conventions below.

---

## File Conventions

- Test files live under `test/` and must match the pattern `*test.js` (e.g., `test/lib/my-module-test.js`).
- Mirror the source structure: a module at `lib/foo/bar.js` gets tests at `test/lib/foo/bar-test.js`.
- One top-level `describe` block per file, named after the module or class under test.

**Unit vs. integration tests**

- **Unit tests** live anywhere under `test/` *except* `test/integration/`. They must be fast (no real subprocesses, no real filesystem work beyond tmp, no real network, no real databases beyond in-memory SQLite). These run on every code change via `npm run test:unit`.
- **Integration tests** live under `test/integration/`. They may spawn real processes, hit real storage engines, exercise full HTTP request/response cycles, etc. These are slower and run via `npm run test:integration` before opening a pull request.

When adding a new test, decide which bucket it belongs in and place the file accordingly. If a test would slow the unit suite noticeably, it belongs in `test/integration/`.

---

## Imports

Use the `test/deps.js` aggregation file for all test framework and assertion imports:

```javascript
import { describe, assertEqual, assert, assertMatches } from '../deps.js';
```

Adjust the relative path depth to match the file's location inside `test/`. Import only the assertion helpers you actually use.

To use mocking:

```javascript
import { describe, MockTracker } from '../deps.js';
```

---

## Structure

```javascript
describe('ModuleName', ({ before, after, it, describe, xit, xdescribe }) => {

    // Shared state across tests in this suite
    let subject;

    before(() => {
        subject = createSubject();
    });

    after(() => {
        // optional cleanup
    });

    it('does the thing', () => {
        assertEqual('expected', subject.doThing());
    });

    describe('nested behavior group', ({ before, it }) => {
        let extra;

        before(() => {
            extra = setup();
        });

        it('handles edge case', () => {
            assert(subject.edgeCase());
        });
    });
});
```

- Destructure only the helpers you need from the callback argument.
- Use nested `describe` blocks to group related behaviors, especially when they share a `before`/`after` setup.
- `before` and `after` are scoped to their enclosing `describe` block.

---

## Test Functions

Tests can be synchronous, async, or callback-style:

```javascript
// Synchronous
it('computes value', () => {
    assertEqual(42, compute());
});

// Async (return a promise or use async/await)
it('fetches data', async () => {
    const result = await fetch();
    assertEqual('ok', result.status);
});

// Callback-style (done parameter)
it('calls back', (done) => {
    setTimeout(() => {
        assertEqual(true, flag);
        done();
    }, 10);
}, { timeout: 100 });
```

The `{ timeout: ms }` option overrides the default timeout for that test.

---

## Skipping Tests

```javascript
xit('temporarily skipped test', () => { ... });

xdescribe('entire skipped suite', ({ it }) => { ... });

// Or pass { disabled: true } on describe options:
describe('disabled suite', ({ it }) => { ... }, { disabled: true });
```

---

## Assertions (kixx-assert)

All assertions throw an `AssertionError` on failure. An optional message string can be passed as the last argument.

| Assertion | Description |
|---|---|
| `assert(value)` | Truthy check |
| `assertFalsy(value)` | Falsy check |
| `assertEqual(expected, actual)` | Strict equality (`===`), handles `NaN` and `Date` |
| `assertNotEqual(expected, actual)` | Strict inequality |
| `assertMatches(pattern, subject)` | RegExp test or string contains check |
| `assertNotMatches(pattern, subject)` | Inverse of `assertMatches` |
| `assertDefined(value)` | Not `undefined` |
| `assertUndefined(value)` | Is `undefined` |
| `assertNonEmptyString(value)` | Non-empty string |
| `assertNumberNotNaN(value)` | Number that is not NaN |
| `assertArray(value)` | Is an array |
| `assertBoolean(value)` | Is a boolean |
| `assertFunction(value)` | Is a function |
| `assertValidDate(value)` | Is a valid Date instance |
| `assertGreaterThan(control, subject)` | subject > control |
| `assertLessThan(control, subject)` | subject < control |

Most two-argument assertions can be curried:

```javascript
const assertIsOk = assertEqual('ok');
assertIsOk(result);

const assertShortDate = assertMatches(/^\d{4}-\d{2}-\d{2}$/);
assertShortDate(dateString);
```

---

## Mocking with MockTracker

```javascript
describe('Service', ({ it }) => {
    it('records function calls', () => {
        const mock = new MockTracker();
        const fn = mock.fn((a, b) => a + b);

        assertEqual(7, fn(3, 4));
        assertEqual(1, fn.mock.callCount());
        assertEqual(3, fn.mock.getCall(0).arguments[0]);
        assertEqual(7, fn.mock.getCall(0).result);
    });

    it('mocks and restores object methods', () => {
        const mock = new MockTracker();
        const obj = { multiply: (a, b) => a * b };

        mock.method(obj, 'multiply', (a, b) => a * b * 10);
        assertEqual(120, obj.multiply(3, 4));

        mock.restoreAll();
        assertEqual(12, obj.multiply(3, 4));
    });
});
```

---

## Running Tests

```bash
# Run lint + unit tests (fast; use after every code change)
npm run test:unit

# Run integration tests only (slow; use before opening a PR)
npm run test:integration

# Run lint + full suite (unit + integration)
npm test

# Run all tests directly
node test.js

# Run a single file
node test.js test/lib/my-module-test.js

# Run all tests in a subdirectory
node test.js test/lib/

# Skip one or more subdirectories (repeatable)
node test.js --skip test/integration
```

---

## Writing Good Tests

- Each `it` block tests one behavior. Name it as a statement: `'returns null when input is empty'`, not `'test 1'`.
- Use `before` to set up shared state; reset it there rather than relying on test order.
- Test the observable behavior (return values, thrown errors, state after mutation), not implementation details.
- When testing error cases, catch and assert on the error. Use `assertEqual` on `error.name` to assert the error type — do NOT use `instanceof`, as it breaks when the same library exists at multiple module paths:

```javascript
it('throws on invalid input', () => {
    let caught = null;
    try {
        subject.process(null);
    } catch (error) {
        caught = error;
    }
    assert(caught, 'expected an error to be thrown');
    assertEqual('TypeError', caught.name);
    assertMatches('Expected non-null', caught.message);
});
```

- For async error cases:

```javascript
it('rejects on failure', async () => {
    let caught = null;
    try {
        await subject.fetch('bad-id');
    } catch (error) {
        caught = error;
    }
    assert(caught, 'expected an error to be thrown');
    assertEqual('NotFoundError', caught.name);
});
```
