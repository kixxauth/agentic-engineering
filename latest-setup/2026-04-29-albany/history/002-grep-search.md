I want to create a tool in this harness for agents to use to perform grep searches across a directory/file tree. It could be a single file, or recursively through a
directory.

What do you think the use cases and requirements should be?

Don't write any code or a plan yet. Let's think about this first.

<assistant>
⏺ Here's how I'd think through it:

Core use cases

1. Find all files containing a specific string or pattern — the classic "where is this thing used?" query.
2. Search a single known file for a pattern — targeted lookup within a file the agent already has a path to.
3. Find all matches with enough context (surrounding lines) to understand what was found without a separate Read call.
4. Search only files matching a certain type — e.g., only .js files, to avoid noise from binaries or lockfiles.

What the output needs to contain

For each match: the absolute file path, the line number, and the matched line. Context lines (before/after) are high-value — without them the agent almost always has to
follow up with a Read. The tradeoff is verbosity vs. round-trips. I'd lean toward optional context lines (default 0, configurable to 2–3).

Pattern dialect

Two realistic choices: literal string search (fast, no escaping surprises) or a JavaScript RegExp. A regex gives agents real power — case-insensitive flags, word boundaries,
etc. A literal mode could be a convenience wrapper. I'd offer both with an explicit isRegex flag rather than auto-detecting.

Sandboxing — same model as GlobSearchTool

- Paths must resolve within the sandbox.
- No symlink following.
- Reject .. in the provided path.

Limits

- Max matches returned (with a truncated flag, same as glob).
- Max file size to scan — you don't want to read a 500 MB binary into memory. Skip files above the threshold and optionally report them as skipped.
- Max depth when recursing (same as glob).

Open questions worth deciding before planning

1. Should multi-file search accept a directory path, a glob pattern, or both? A directory means "search everything under here"; a glob pattern (reusing picomatch) lets agents
 be selective. Combining both is the most powerful but adds surface area.
2. Should binary files be silently skipped, or reported? Silent skip is the safer default.
3. Is per-file match count reported separately, or is it just a flat list of all matches across all files?

The biggest design choice is #1 — it determines how much this overlaps with GlobSearchTool and whether they should share traversal logic.

</assistant>

I think we should offer both literal string match and RegExp matching.

**Open Questions**

1. We should accept both a directory path or glob pattern for multi file search
2. Binary files should be silently skipped
3. We return a flat list of all matches across all files.

And, should we allow multi-line matching?

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
