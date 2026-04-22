<user>
This project is a cli tool which implements a development environment for web applications built on Cloudflare workers.

For local development we use a platform called "miniflare" to emulate the Cloudflare workers runtime.

Review the miniflare documentation at documentation/miniflare.md

We need to create a dev server which loads an application and makes it available via HTTP for local development.

I'm thinking we create a new CLI command called "dev" with a sub-command called "server" for this.

An example application is at ../kingston-poc/
</user>

---

<assistant>
Here's the shape I'd propose:

Structure — add commands/dev/index.js + commands/dev/server.js (thin, like app/deploy.js), and a app/dev/miniflare-server.js module that owns the Miniflare lifecycle.

Reuse ApplicationDirectory and ApplicationConfig to resolve the app dir and pull main_module, compatibility_date, compatibility_flags from its config.

Options: --host (default 127.0.0.1), --port (default 8787), probably --live-reload on by default.

Main tradeoff — how to feed the worker to Miniflare:

- (a) Point scriptPath at `<appDir>/src/<main_module>` with modules: true and let Miniflare auto-discover imports. Simplest, matches what miniflare expects, and liveReload "just works."
- (b) Build an explicit modules: [...] array from SourceBundle to mirror what app deploy uploads. More code, but dev mirrors production bundling exactly.

I'd recommend (a) for the POC — it's less code and Miniflare's discovery matches the behavior of Cloudflare's module resolution. We can switch to (b) later if dev/deploy divergence ever bites.

Want me to proceed with (a), or prefer (b)?
</assistant>

---

<user>
Ok, yeah. Let's stick with (a) for now.
</user>

<system>
After reviewing the conversation above, create an implementation plan document.

Think hard to imagine all the user stories which would encapsulate the user prompt above.

Review all user stories you can think of and then plan to implement them cohesively for your implementation plan document.

The plan should begin with a brief Implementation Approach section (3–5 sentences) summarizing the overall strategy and any cross-cutting concerns across the stories.

The rest of the document is a TODO list. Break each user story into discrete technical tasks — one task per file change, component, route, or logical unit of work. Each TODO item must follow this exact format:

```
- [ ] **<Short title>**
  - **Story**: <User story ID or title>
  - **What**: <What to build or change, in concrete terms>
  - **Where**: <File path(s) or module(s) to create or modify>
  - **Acceptance criteria**: <Which AC items this task satisfies>
  - **Depends on**: <Item titles this must come after, or "none">
```

Order items so that dependencies come first. Do not group items by story — sequence them by the order they should be implemented.

When completed, put the plan document in the prompts/plans/ directory.
</system>

