I want to implement a harness tool which provides a way for agents to perform glob searches across a file tree in a sandbox directory. What do you think the requirements of such a tool should be?

<assistant>
Good thing to think through before designing. Here's how I'd frame the requirements:

Core behavior
- Input: a glob pattern (or patterns?) and a root directory inside the sandbox.
- Output: a list of matching file paths, returned relative to the sandbox root (not absolute) so results are stable and don't leak host paths.
- Glob semantics need to be pinned down: *, **, ?, [...], brace expansion {a,b}, leading ! for negation? Pick a single well-known dialect (e.g. picomatch/minimatch defaults)
rather than rolling your own.

Sandboxing (the part that matters most)
- All paths must be resolved and then checked to live under the sandbox root after symlink resolution — otherwise a symlink inside the sandbox can escape it.
- Decide up front: do you follow symlinks at all? Safest default is no.
- Reject absolute patterns and .. segments in the input pattern itself.

Limits (to keep agents from blowing up the harness)
- Max number of results (truncate with a clear "more results exist" signal).
- Max traversal depth and/or max files visited.
- A wall-clock timeout.
- Default ignores: .git, node_modules, anything in a .gitignore? That last one is a real choice — convenient but couples the tool to git semantics.

Ergonomics for the agent
- Sort order should be deterministic (alpha, or mtime-desc — pick one and document it).
- Return just paths, or also file size / mtime / type? Minimal is better; agents can Read follow-ups.
- Clear, structured errors (pattern_invalid, root_outside_sandbox, limit_exceeded) so the agent can react.

Main tradeoff to decide first: how much does this overlap with grep/Read? If agents already have shell access to the sandbox, a glob tool earns its keep mainly by enforcing the sandbox boundary and the limits — which suggests the design should center on safety guarantees, not on glob features.

Want to start by nailing down the sandboxing model and the limits, then work outward to the pattern dialect?
</assistant>

**Output**

You proposed:

> Output: a list of matching file paths, returned relative to the sandbox root (not absolute) so results are stable and don't leak host paths.

However, for this implementation, since it is contained, we DO want to return absolute file paths.

Let's use the picomatch library for glob matching, since it does not have any dependencies.

**Sandboxing**

- We do not follow symlinks
- Allow absolute patterns
- Reject patterns with .. segments

**Limits**

- We do not need a wall clock timeout for this implementation
- Do not ignore any paths by default

**Ergonomics for the agent**

- Use alpha for sort order
- Returning just paths without metadata will be fine

**Main tradeoff** - This harness does not support bash, so glob features are more important than sandbox safety.

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
