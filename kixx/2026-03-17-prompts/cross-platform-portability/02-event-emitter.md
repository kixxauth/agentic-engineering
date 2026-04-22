You may have already gleaned this information from the README.md and CLAUDE.md files, but just to ensure you have the context behind this project:

<background>
The user is the primary maintainer of an open source web development framework called Kixx. The Kixx Framework supports applications built on server-side JavaScript platforms like Node.js, Deno, and Bun. The Kixx Framework provides primitive capabilities for web applications like HTTP routing, HTML templating, configuration, and logging.

The Kixx Framework allows developers to build their applications and deploy them to multiple targets, including Node.js Linux servers, Cloudflare Workers, and AWS Lambda functions, without any changes to the application code. Our objective with the Kixx framework is to accomplish this cross platform portability through the use of the dependency inversion principle: we create concrete implementations of lower level components which are platform specific and then higher level abstractions can use these components to provide a common interface. Only the lower level components are swapped out when the deployment target changes.

**NOTE** that today we only support Node.js running on a Mac, Linux, or Windows system, but have plans to deploy on Cloudflare Workers and AWS Lambda functions.
</background>

Here is a quick summary of the dependency inversion principle:

<dependency_inversion_principle>
The **Dependency Inversion Principle** (DIP) states:

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

## The Problem

Without DIP, code often looks like this:

```javascript
class EmailService {
  send(to, message) {
    // Implementation details
  }
}

class UserRegistration {
  constructor() {
    this.emailService = new EmailService();  // Direct dependency
  }
  
  register(user) {
    // ... registration logic
    this.emailService.send(user.email, 'Welcome!');
  }
}
```

The problem: `UserRegistration` (high-level) directly depends on `EmailService` (low-level). If you want to change email providers or test with a mock service, you have to modify `UserRegistration`.

## The Solution

Instead, both depend on an abstraction (interface/contract):

```javascript
class UserRegistration {
  constructor(notificationService) {
    this.notificationService = notificationService;  // Depends on abstraction
  }
  
  register(user) {
    // ... registration logic
    this.notificationService.notify(user.email, 'Welcome!');
  }
}

// Now you can swap implementations
class EmailService {
  notify(to, message) { /* ... */ }
}

class SlackService {
  notify(userId, message) { /* ... */ }
}

// Use whichever you want
const registration = new UserRegistration(new EmailService());
```

## Key Benefits

- **Loose coupling**: High-level code doesn't care about implementation details
- **Testability**: Easy to inject mock services for testing
- **Flexibility**: Swap implementations without changing the core logic
- **Maintainability**: Changes to low-level code don't break high-level code

The principle is about depending on *what something does* (the interface) rather than *how it does it* (the implementation).
</dependency_inversion_principle>

After doing some exploration and analysis of this codebase you came up with some recommended improvements. Your observation which I'd like to address is:

> `node:events` leaks into platform-agnostic code

> `Config` and `HttpRouter` both directly import `EventEmitter` from `node:events`. This is a Node.js-specific module. On Cloudflare Workers, `EventEmitter` isn't available natively.

> Options:
> - **Inject the emitter** (consistent with your existing DIP pattern)
> - **Use a tiny platform-agnostic EventEmitter** — a simple pub/sub implementation is ~20 lines and has no platform dependencies
> - **Accept it as a known constraint** and polyfill on non-Node platforms

> Fixing this unblocks the core from running on Workers/Lambda where `EventEmitter` isn't available. It also forces you to decide on your eventing contract

So, I'd like you to create a platform agnostic EventEmitter implementation.
