---
name: document-code
description: >
  Add or improve JSDoc block comments and inline comments to JavaScript code.
  Use when asked to document a file, class, function, or module — or to review
  existing documentation for completeness and correctness. Follows project
  conventions: JSDoc for the public API contract (types, params, returns,
  throws, events) and inline comments for the "why" behind non-obvious logic.
  Invoke with a file path or symbol name: `/document-code src/foo.js`
argument-hint: "[file or symbol]"
---

Document JavaScript code following the two-layer convention used in this project:

- **JSDoc block comments** — the formal API contract: types, parameters, return values, errors, and events. Consumed by editors, documentation generators, and future readers of the public interface.
- **Inline comments** — the narrative layer explaining *why*, constraints, workarounds, and non-obvious decisions. Cannot be expressed by types or names alone.

JSDoc answers "what does this do and how do I call it?" Inline comments answer "why does it work this way?"

---

## JSDoc Block Comments

JSDoc blocks reduce cognitive load by answering three questions without reading the implementation: "What does this do?", "How do I use it?", and "What can go wrong?"

### Supported JSDoc Tags

- **@async**: mark a function or method as asynchronous when it does not use the `async` keyword.
- **@readonly**: mark a symbol to be readonly, meaning that it cannot be overwritten.
- **@module**: defined on a top-level JSDoc comment, treats that comment as being for the file instead of the subsequent symbol. A value can be specified to use as an identifier for the module (i.e., for default exports).
- **@see**: define an external reference related to the symbol.
- **@callback**: define a callback.
- **@property**: define a property on a symbol.
- **@typedef**: define a type.
- **@param**: define a parameter on a function.
- **@emits**: denote an event which an object or class emits.
- **@returns**: define the return type and/or comment of a function.
- **@throws**: define what a function throws when called.
- **@enum**: define an object to be an enum.
- **@extends**: define a type that a function extends on.
- **@name**: explicitly define the name of a symbol which may otherwise be difficult to discern.
- **@type**: define the type of a symbol.
- **@default**: define the default value for a variable, property or field.

### Add Value Beyond the Name

Write concise descriptions that add value beyond the name of the thing being documented. Keeping it concise helps documentation remain relevant longer.

**Good:**
```javascript
/**
 * Retrieves user data from the database with role information populated
 */
function getUserById() {}
```

**Bad:**
```javascript
/**
 * This function takes a user ID parameter and returns user data
 */
function getUserById() {}
```

### Document the Contract, Not the Implementation

Focus on *what* the function does and *how* to use it, not *how* it works internally.

```javascript
/**
 * Calculates the total price including tax and discounts
 * @param {number} basePrice - The original price before adjustments
 * @param {number} taxRate - Tax rate as a decimal (e.g., 0.08 for 8%)
 * @param {number} [discount=0] - Discount amount to subtract
 * @returns {number} The final price after tax and discount
 */
function calculateTotal(basePrice, taxRate, discount = 0) {}
```

### Specify Types Precisely

Be specific about object shapes, array contents, and union types. Use `@typedef` blocks to document complex data structures:

```javascript
/**
 * @typedef {Object} UserProfile
 * @property {string} id - Unique user identifier
 * @property {string} email - User's email address
 * @property {string[]} roles - Array of role names
 */

/**
 * @param {String} id - The UUID for the user.
 * @returns {UserProfile} The full UserProfile object.
 */
function getUser(id) {}
```

Show the relative JSDoc import path to external type definitions when they exist:

```javascript
/**
 * @typedef {import('../config/config.js').default} Config
 */
```

Only include the import path like `import('../config/config.js').default` if the file path can be positively located. If not, use just the name (e.g., `Config`).

Use dotted `@param` notation for function and method arguments/options. Do **not** introduce a `@typedef` just to describe an `options` argument shape — document it inline with dotted params instead:

```javascript
/**
 * @param {Object} options - Context initialization options
 * @param {AppRuntime} options.runtime - Runtime configuration
 * @param {Config} options.config - Application configuration manager instance
 * @param {Logger} options.logger - Logger instance for application logging
 */
createContext({ runtime, config, logger }) {}
```

### Document Error Conditions and Edge Cases

```javascript
/**
 * Reads and parses a JSON configuration file
 * @param {string} filePath - Path to the JSON file
 * @returns {Promise<Object>} Parsed configuration object
 * @throws {Error} When file doesn't exist or contains invalid JSON
 * @throws {TypeError} When filePath is not a string
 */
```

### Know When to Skip JSDoc

A well-named private function with obvious parameters sometimes needs no JSDoc at all. If the function is *not* public and the JSDoc would just restate the function name and parameter names, leave it out.

```javascript
// No JSDoc needed — the name and signature say it all
function isEven(n) {
    return n % 2 === 0;
}
```

### Document Async Behavior

Be explicit about what Promises resolve to. Use `Promise<void>` (not `Promise<undefined>`) for functions that don't resolve to a meaningful value.

Use `@async` on methods and functions that return a Promise but do not use the `async` keyword. If the `async` keyword is present, `@async` is redundant and should be omitted.

```javascript
/**
 * @async
 * @param {number} milliseconds
 * @returns {Promise<void>}
 */
function delay(milliseconds) {
    return new Promise((resolve) => setTimeout(resolve, milliseconds));
}

/**
 * @param {string} userId
 * @returns {Promise<UserProfile|null>} Resolves to user profile or null if not found
 * @throws {DatabaseError} When database connection fails
 */
async function getUser(userId) {}
```

### Document Events

Document events using the `@emits` tag. Use `@typedef` blocks to document event object structures:

```javascript
/**
 * @typedef {Object} FileChangeEvent
 * @property {string} filepath - Absolute path to the changed file
 * @property {string} eventType - Type of change ('rename' or 'change')
 */

/**
 * Monitors a directory for file changes using glob patterns to filter events.
 * @extends EventEmitter
 * @emits FileWatcher#change - Emits a FileChangeEvent when a matching file changes
 * @emits FileWatcher#error - Emits a WrappedError when the underlying fs.watch fails
 */
export default class FileWatcher extends EventEmitter {}
```

### Document Classes

- Use `@name` on members defined via `Object.defineProperties()` or `Object.defineProperty()` to give them an explicit name.
- Do *not* add the `@private` tag to private members — JavaScript's `#private` syntax already communicates visibility.
- Sparse documentation is acceptable for private methods and members — a brief description is sufficient.
- **Do not include a description for `constructor` JSDoc blocks.** Only document `@param` tags (and `@throws` if relevant).

### Using @name with Object.defineProperties

When properties are defined via `Object.defineProperties()` or `Object.defineProperty()`, add a JSDoc block with `@name` so the property is discoverable. Put the description first, then `@name`, then `@type`:

```javascript
constructor({ runtime, config, paths, logger }) {
    Object.defineProperties(this, {
        /**
         * Runtime configuration indicating whether the application is running as a CLI command or server.
         * @name runtime
         * @type {AppRuntime}
         */
        runtime: { value: runtime },
    });
}
```

For properties created dynamically by a setter method, place a standalone JSDoc block near the top of the class:

```javascript
export default class Context {

    /**
     * The root user with permission to perform any operation in the app.
     * @name rootUser
     * @type {User}
     */

    /**
     * Sets the root user instance for the context, creating a read-only rootUser property.
     * @param {User} user - Root user instance with elevated privileges
     * @throws {TypeError} When rootUser has already been set (operation cannot be repeated)
     */
    setRootUser(user) {
        Object.defineProperty(this, 'rootUser', { value: user });
    }
}
```

### Use @see for Cross-References

```javascript
/**
 * Validates and normalizes user input before persisting.
 * @param {UserProfile} profile - Raw user profile data
 * @returns {UserProfile} Normalized profile ready for storage
 * @see Context#registerCollection for how collections are registered
 */
function normalizeProfile(profile) {}
```

---

## Inline Code Comments

JSDoc describes the *what* and *how* at the public API level. Inline comments fill the gap by explaining *why* — intent, constraints, and context that the code itself cannot express.

### Explain the "Why," Not the "What"

Focus on why the code exists and why it does what it does, especially when it seems counterintuitive or requires domain knowledge:

```javascript
// Increment DB counter BEFORE processing to ensure we don't
// get stuck on the same database if we hit the time limit
currentDb = (currentDb + 1) % totalDatabases;
```

### Avoid Trivial Comments

```javascript
// Bad
user.name = 'John'; // Set the user name to John

// Good
user.name = sanitizeInput(rawName); // Remove potential XSS vectors
```

### Use Guide Comments to Break Up Complex Logic

```javascript
async function processPayment(order, paymentMethod) {
    // Validate payment details and customer eligibility
    await validatePaymentMethod(paymentMethod);
    await checkCustomerCredit(order.customerId);

    // Calculate final amounts including taxes and fees
    const taxAmount = calculateTax(order);
    const finalAmount = order.total + taxAmount + calculateFee(paymentMethod, order.total);

    // Process payment and update order status
    const transaction = await chargePayment(paymentMethod, finalAmount);
    await updateOrderStatus(order.id, 'paid', transaction.id);

    return transaction;
}
```

### Document State Transitions and Side Effects

```javascript
// After this call, the connection state changes to 'authenticating'
// and subsequent messages will be queued until auth completes
await connection.startAuthentication(credentials);
```

### Use "Teacher Comments" for Domain Knowledge

```javascript
// JWT exp claim uses NumericDate format (seconds since epoch)
// JavaScript Date.now() returns milliseconds, so we divide by 1000
const expiry = Math.floor(Date.now() / 1000) + (60 * 60 * 24); // 24 hours
```

### Document Workarounds and Hacks

```javascript
// Workaround: Some legacy clients send timestamps as strings
// TODO: Remove this once all clients upgrade to v2.0+
const timestamp = typeof data.timestamp === 'string'
    ? parseInt(data.timestamp, 10)
    : data.timestamp;
```

### Explain Performance or Memory Considerations

```javascript
// Pre-allocate buffer to avoid multiple reallocations
// during high-frequency writes (saves ~40% memory churn)
const buffer = Buffer.allocUnsafe(expectedSize);

// Process in chunks to avoid blocking the event loop
for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);
    await processChunk(chunk);

    // Yield control back to event loop between chunks
    await setImmediate();
}
```

### Flag Coordinated Change Points

```javascript
const EVENT_TYPES = {
    USER_LOGIN: 'user:login',
    USER_LOGOUT: 'user:logout',
    // WARNING: When adding event types here, also update:
    // - src/analytics/event-handlers.js
    // - tests/fixtures/events.json
};
```

**Priority order:** Prefer "why" comments over "what" comments. When both are obvious, don't comment at all.
