# Glob Search Tool Implementation Plan

## Implementation Approach

A `GlobSearchTool` class in `lib/tools/glob-search-tool.js` follows the same sandboxed-tool pattern as `ReadTool`, `WriteTool`, and `EditTool`. The tool accepts a glob pattern and a root directory, traverses the directory tree using `node:fs/promises` (without following symlinks), tests each discovered path against the pattern using the vendored picomatch library, and returns a sorted list of absolute paths. Path validation rejects patterns containing `..` segments and enforces that the traversal root lies within the sandbox. Traversal is bounded by a configurable max depth and a configurable max results count; when the results cap is hit the return value signals truncation so callers can react.

---

## TODO

- [x] **Vendor picomatch**
  - **Story**: Agent can match files using standard glob syntax
  - **What**: Copy the picomatch package source into the project's vendored dependencies. Picomatch is a zero-dependency glob library. Vendor the files needed for the default export: `lib/picomatch.js`, `lib/parse.js`, `lib/utils.js`, `lib/constants.js`, and a minimal `index.js` re-export.
  - **Where**: `lib/vendor/picomatch/` (new directory)
  - **Acceptance criteria**: `import picomatch from '../vendor/picomatch/index.js'` resolves and `picomatch('**/*.js')('/foo/bar.js')` returns `true`
  - **Depends on**: none

- [x] **Create GlobSearchTool class — constructor and sandbox validation**
  - **Story**: Agent can instantiate the tool with a sandbox directory; invalid instantiation throws immediately
  - **What**: Export a default `GlobSearchTool` class. Constructor accepts `{ sandboxDirectory, maxResults, maxDepth }`. Validate `sandboxDirectory` with `assertNonEmptyString`. Set defaults: `maxResults = 1000`, `maxDepth = 50`. Store all three as private fields. Implement a private `#sandboxRealPath()` method that resolves and caches the realpath of the sandbox once.
  - **Where**: `lib/tools/glob-search-tool.js` (new file)
  - **Acceptance criteria**: Constructor throws `AssertionError` when `sandboxDirectory` is missing or not a string; valid construction succeeds
  - **Depends on**: none

- [x] **Pattern validation — reject `..` segments**
  - **Story**: Agent receives a clear error when an escape-attempt pattern is supplied
  - **What**: Add a private `#validatePattern(pattern)` method. Split on `/` and `\`, reject if any segment is `..`. Reject if `pattern` is not a non-empty string. Throw `BadRequestError` on any violation.
  - **Where**: `lib/tools/glob-search-tool.js`
  - **Acceptance criteria**: `glob('../secret', root)` rejects with `BadRequestError`; `glob('**/../etc', root)` also rejects
  - **Depends on**: Create GlobSearchTool class — constructor and sandbox validation

- [x] **Root directory validation — must be within sandbox**
  - **Story**: Agent cannot search outside the sandbox by passing a root that escapes it
  - **What**: Add a private `#assertSafeRoot(root)` method. Require `root` to be an absolute path (`BadRequestError` if relative). Resolve its realpath (or deepest existing ancestor as in `ReadTool`). Check that it starts with the sandbox realpath using the same `#isWithinSandbox` helper. Throw `ForbiddenError` when it escapes.
  - **Where**: `lib/tools/glob-search-tool.js`
  - **Acceptance criteria**: Passing a root outside the sandbox throws `ForbiddenError`; passing a root equal to or inside the sandbox succeeds
  - **Depends on**: Create GlobSearchTool class — constructor and sandbox validation

- [x] **Directory traversal — recursive walk without following symlinks**
  - **Story**: Agent's search does not escape the sandbox through symlinks; deep trees are bounded
  - **What**: Add a private `async #walk(dir, currentDepth, results, limit)` method. Use `fsp.readdir(dir, { withFileTypes: true })`. For each entry: skip if `isSymbolicLink()`. Recurse into subdirectories only when `currentDepth < maxDepth`. Push absolute file paths for file entries. Stop recursing and pushing once `results.length >= limit`.
  - **Where**: `lib/tools/glob-search-tool.js`
  - **Acceptance criteria**: Symlinks are skipped; directories deeper than `maxDepth` are not entered; traversal stops collecting once `limit` is hit
  - **Depends on**: Create GlobSearchTool class — constructor and sandbox validation

- [x] **Glob matching and sorted return value**
  - **Story**: Agent receives alphabetically sorted absolute paths that match the pattern
  - **What**: Add the public `async glob(pattern, root)` method. Validate the pattern, validate and resolve the root, walk from root, test each candidate path against the compiled picomatch matcher. Sort results with `Array.prototype.sort()` (default locale-independent alpha). Return `{ paths: string[], truncated: boolean }` where `truncated` is `true` when the walker hit `maxResults`.
  - **Where**: `lib/tools/glob-search-tool.js`
  - **Acceptance criteria**: Results are alphabetically sorted; only matching paths are included; `truncated: true` appears when results exceed `maxResults`
  - **Depends on**: Pattern validation — reject `..` segments, Root directory validation — must be within sandbox, Directory traversal — recursive walk without following symlinks, Vendor picomatch

- [x] **Tests — constructor validation**
  - **Story**: Confidence that misconfigured tool instances fail fast
  - **What**: Test that constructing without `sandboxDirectory`, or with a non-string value, throws immediately.
  - **Where**: `test/lib/tools/glob-search-tool-test.js` (new file)
  - **Acceptance criteria**: Two `it` cases; no filesystem I/O needed
  - **Depends on**: Create GlobSearchTool class — constructor and sandbox validation

- [x] **Tests — pattern validation**
  - **Story**: Agents get clear feedback on invalid patterns
  - **What**: Create a temp-dir sandbox fixture (same pattern as `read-tool-test.js`). Test: `..` in pattern rejects with `BadRequestError`; empty string rejects; a valid pattern resolves.
  - **Where**: `test/lib/tools/glob-search-tool-test.js`
  - **Acceptance criteria**: Three `it` cases
  - **Depends on**: Tests — constructor validation, Pattern validation — reject `..` segments

- [x] **Tests — root outside sandbox**
  - **Story**: Sandbox boundary is enforced on the search root
  - **What**: Using the same fixture (sandbox dir + sibling outside dir), verify that passing the outside dir as root rejects with `ForbiddenError`. Verify that passing the sandbox dir itself succeeds.
  - **Where**: `test/lib/tools/glob-search-tool-test.js`
  - **Acceptance criteria**: Two `it` cases
  - **Depends on**: Tests — pattern validation, Root directory validation — must be within sandbox

- [x] **Tests — symlinks are not followed**
  - **Story**: Traversal cannot escape through symlinks
  - **What**: In the fixture, create a symlink inside the sandbox pointing to the outside dir. Run `glob('**/*', sandboxDir)`. Assert the symlink path does not appear in results.
  - **Where**: `test/lib/tools/glob-search-tool-test.js`
  - **Acceptance criteria**: One `it` case
  - **Depends on**: Tests — root outside sandbox, Directory traversal — recursive walk without following symlinks

- [x] **Tests — maxDepth limit**
  - **Story**: Deep trees are bounded
  - **What**: Create a fixture with files at depth 1, 3, and 6. Construct the tool with `maxDepth: 4`. Run `glob('**/*', sandboxDir)`. Assert files at depth > 4 do not appear.
  - **Where**: `test/lib/tools/glob-search-tool-test.js`
  - **Acceptance criteria**: One `it` case
  - **Depends on**: Tests — symlinks are not followed

- [x] **Tests — maxResults truncation**
  - **Story**: Agent knows when results are incomplete
  - **What**: Create a fixture with more files than `maxResults` (e.g., 5 files, `maxResults: 3`). Assert `truncated === true` and `paths.length === 3`.
  - **Where**: `test/lib/tools/glob-search-tool-test.js`
  - **Acceptance criteria**: One `it` case covering both `truncated` flag and result count
  - **Depends on**: Tests — maxDepth limit

- [x] **Tests — glob pattern matching and sorted output**
  - **Story**: Agent gets correct matches in alpha order
  - **What**: Create a fixture with `a.js`, `b.ts`, `c.js`, `sub/d.js`. Run `glob('**/*.js', sandboxDir)`. Assert results equal `[absPath(a.js), absPath(c.js), absPath(sub/d.js)]` in that order and `truncated === false`.
  - **Where**: `test/lib/tools/glob-search-tool-test.js`
  - **Acceptance criteria**: One `it` case; verifies extension filtering and sort order
  - **Depends on**: Tests — maxResults truncation, Glob matching and sorted return value
