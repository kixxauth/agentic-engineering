# Grep Tool Implementation Plan

## Implementation Approach

A `GrepTool` class in `lib/tools/grep-tool.js` follows the same sandboxed-tool pattern as `GlobSearchTool` and `ReadTool`. The public method `grep(searchPattern, target, options)` accepts a literal string or RegExp pattern, a target that may be an absolute file path, an absolute directory path, or a picomatch glob pattern rooted at the sandbox. Matching is always line-by-line (no multi-line mode); binary files detected by a null-byte probe are silently skipped. The return value is a flat list of match objects `{ path, lineNumber, line, contextBefore, contextAfter }` across all matched files, sorted by path then line number, with a `truncated` boolean when the `maxMatches` cap is hit. Picomatch is already vendored from the GlobSearchTool work and is reused here for glob-pattern target resolution.

---

## TODO

- [x] **Create GrepTool class — constructor and defaults**
  - **Story**: Agent can instantiate the tool bound to a sandbox directory
  - **What**: Export a default `GrepTool` class. Constructor accepts `{ sandboxDirectory, maxMatches, maxDepth, maxFileSizeBytes }`. Validate `sandboxDirectory` with `assertNonEmptyString`. Set defaults: `maxMatches = 1000`, `maxDepth = 50`, `maxFileSizeBytes = 1_000_000`. Store all four as private fields. Add a private `#resolveSandboxRealPath()` method that resolves and caches the sandbox realpath on first call.
  - **Where**: `lib/tools/grep-tool.js` (new file)
  - **Acceptance criteria**: Constructor throws `AssertionError` when `sandboxDirectory` is missing or not a string; valid construction succeeds
  - **Depends on**: none

- [x] **Sandbox path helper — shared realpath and within-sandbox check**
  - **Story**: All path validation enforces the sandbox boundary
  - **What**: Add private methods `#sandboxRealPath()` (cached realpath of sandbox) and `#isWithinSandbox(realPath, sandboxRealPath)` (same logic as `ReadTool`: check that `path.relative()` result is empty or does not start with `..`). Add `#assertSafeAbsolutePath(inputPath)` that rejects relative paths with `BadRequestError` and paths that escape the sandbox with `ForbiddenError`, following the deepest-existing-ancestor resolution pattern from `ReadTool`.
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: Paths outside sandbox throw `ForbiddenError`; relative paths throw `BadRequestError`
  - **Depends on**: Create GrepTool class — constructor and defaults

- [x] **Binary file detection**
  - **Story**: Binary files are silently skipped
  - **What**: Add a private `async #isBinaryFile(filePath)` method. Read the first 8 KB of the file with `fsp.read` into a `Buffer`. Return `true` if any byte is `0x00`. Return `false` otherwise. Treat any read error as non-binary so the search attempt surfaces the real error.
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: A file containing a null byte is detected as binary; a UTF-8 text file is not; a file over `maxFileSizeBytes` is skipped before binary detection is reached
  - **Depends on**: Create GrepTool class — constructor and defaults

- [x] **Single-file search — compile pattern and match lines**
  - **Story**: Agent can search one file by literal string or RegExp
  - **What**: Add a private `async #searchFile(filePath, matcher, contextLines, results, maxMatches)` method. Skip files whose `stat.size` exceeds `maxFileSizeBytes`. Skip binary files via `#isBinaryFile`. Read file as UTF-8, split on `\n`. For each line, test with `matcher(line)`. On match, push `{ path: filePath, lineNumber: i + 1, line, contextBefore, contextAfter }` sliced from the lines array, then stop once `results.length >= maxMatches`. The `matcher` is a plain function `(line: string) => boolean` built by the caller.
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: Literal matches found; RegExp matches found; lines beyond `maxMatches` not added; context lines correctly sliced (clamped at file start/end)
  - **Depends on**: Binary file detection

- [x] **Pattern compilation — literal and RegExp modes**
  - **Story**: Agent can choose between literal string search and RegExp search
  - **What**: Add a private `#compileMatcher(searchPattern, isRegex, flags)` method. When `isRegex` is falsy, return a matcher using `String.prototype.includes`. When truthy, compile `new RegExp(searchPattern, flags)` and return a matcher using `RegExp.prototype.test`. Throw `BadRequestError` if the RegExp constructor throws (invalid pattern). The `flags` string defaults to `''`; valid flags are `i`, `m`, `s`, `g` — validate with a RegExp and throw `BadRequestError` on invalid flags. Note: multi-line flag `m` affects `^`/`$` anchors only; matching is still applied per-line.
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: Invalid RegExp throws `BadRequestError`; literal match is case-sensitive; `i` flag makes RegExp case-insensitive
  - **Depends on**: Create GrepTool class — constructor and defaults

- [x] **Target resolution — file, directory, or glob**
  - **Story**: Agent can target a single file, an entire directory tree, or a glob pattern
  - **What**: Add a private `async #resolveTarget(target)` method. If `target` is absolute and `stat` resolves to a file, return `{ mode: 'file', path }`. If it resolves to a directory, return `{ mode: 'directory', path }`. Otherwise (glob characters present, or path does not exist), treat as a picomatch glob matched from the sandbox root using the traversal logic already in `#walkDirectory`, and return `{ mode: 'glob', paths: string[] }`. Reject `..` segments in the target string with `BadRequestError`. Enforce sandbox boundary for absolute path targets via `#assertSafeAbsolutePath`.
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: Existing file path → file mode; existing directory → directory mode; `*.js` glob → glob mode returning matching files; `..` in target throws `BadRequestError`
  - **Depends on**: Sandbox path helper — shared realpath and within-sandbox check

- [x] **Directory traversal for grep**
  - **Story**: Agent's directory search does not follow symlinks and respects depth and size limits
  - **What**: Add a private `async #walkDirectory(dir, currentDepth, fileList)` method. Use `fsp.readdir(dir, { withFileTypes: true })`. Skip symlinks (`isSymbolicLink()`). Recurse into subdirectories only when `currentDepth < this.#maxDepth`. Push absolute file paths for file entries. Stop collecting once the caller's result count reaches `maxMatches` (pass by reference via an object or check a shared accumulator).
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: Symlinks skipped; directories beyond `maxDepth` not entered
  - **Depends on**: Sandbox path helper — shared realpath and within-sandbox check

- [x] **Public `grep()` method — wire everything together**
  - **Story**: Agent calls one method and receives a sorted flat list of matches
  - **What**: Add the public `async grep(searchPattern, target, options)` method. Destructure `options` for `{ isRegex = false, flags = '', contextLines = 0, maxMatches }` (fall back to `this.#maxMatches`). Compile the matcher. Resolve the target. For file mode search the single file; for directory mode walk then search each file; for glob mode search pre-resolved paths from `#resolveTarget`. After collection, sort `results` by `path` then `lineNumber`. Return `{ matches, truncated }` where `truncated` is true when collection stopped at the limit.
  - **Where**: `lib/tools/grep-tool.js`
  - **Acceptance criteria**: Results sorted by path then line number; `truncated: true` when limit reached; all three target modes produce matches
  - **Depends on**: Single-file search — compile pattern and match lines, Pattern compilation — literal and RegExp modes, Target resolution — file, directory, or glob, Directory traversal for grep

- [x] **Tests — constructor validation**
  - **Story**: Misconfigured tool instances fail fast
  - **What**: Test that constructing without `sandboxDirectory` or with a non-string value throws immediately. No filesystem I/O needed.
  - **Where**: `test/lib/tools/grep-tool-test.js` (new file)
  - **Acceptance criteria**: Two `it` cases
  - **Depends on**: Create GrepTool class — constructor and defaults

- [x] **Tests — pattern validation**
  - **Story**: Agents get clear feedback on invalid patterns
  - **What**: Create a temp-dir sandbox fixture with one text file. Test: invalid RegExp string with `isRegex: true` throws `BadRequestError`; invalid `flags` string throws `BadRequestError`; valid literal and valid RegExp patterns resolve without error.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: Three `it` cases
  - **Depends on**: Tests — constructor validation, Pattern compilation — literal and RegExp modes

- [x] **Tests — literal string match in a single file**
  - **Story**: Agent finds all lines containing the search string
  - **What**: Write a fixture file with known content. Assert that `grep('foo', filePath)` returns the correct line numbers, line text, and `truncated: false`. Assert case-sensitive: `grep('FOO', filePath)` returns no matches.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: Two `it` cases
  - **Depends on**: Tests — pattern validation, Single-file search — compile pattern and match lines

- [x] **Tests — RegExp match with flags in a single file**
  - **Story**: Agent finds matches using a regular expression, including case-insensitive
  - **What**: Assert `grep('fo+', filePath, { isRegex: true })` matches; assert `grep('FOO', filePath, { isRegex: true, flags: 'i' })` also matches.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: Two `it` cases
  - **Depends on**: Tests — literal string match in a single file

- [x] **Tests — context lines**
  - **Story**: Agent receives surrounding lines without an extra Read call
  - **What**: Write a 7-line fixture file with a match on line 4. Assert `grep('pattern', filePath, { contextLines: 2 })` returns `contextBefore` with 2 lines and `contextAfter` with 2 lines. Assert a match on line 1 clamps `contextBefore` to an empty array.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: Two `it` cases
  - **Depends on**: Tests — RegExp match with flags in a single file

- [x] **Tests — binary file silently skipped**
  - **Story**: Binary files do not produce errors or spurious matches
  - **What**: Write a binary fixture file (insert a null byte). Write a sibling text file with a match. Assert `grep('match', sandboxDir)` returns only the text file match and no error.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: One `it` case
  - **Depends on**: Tests — context lines, Binary file detection

- [x] **Tests — max file size silently skipped**
  - **Story**: Oversized files do not block the search
  - **What**: Construct the tool with `maxFileSizeBytes: 10`. Write a fixture file with more than 10 bytes. Assert `grep` returns no matches for that file and no error.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: One `it` case
  - **Depends on**: Tests — binary file silently skipped

- [x] **Tests — directory target, symlinks skipped**
  - **Story**: Directory search finds matches across multiple files without escaping via symlinks
  - **What**: Create a sandbox with two text files (each containing a match) and a symlink to an outside file that also contains the pattern. Assert `grep('pattern', sandboxDir)` returns matches from both text files only, symlink excluded. Assert results are sorted by path.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: Two `it` cases (match set, sort order)
  - **Depends on**: Tests — max file size silently skipped, Directory traversal for grep

- [x] **Tests — glob pattern target**
  - **Story**: Agent can restrict search to files matching a file-type pattern
  - **What**: Create a sandbox with `a.js`, `b.ts`, `c.js` each containing the search string. Assert `grep('pattern', '**/*.js', ...)` (glob rooted at sandbox) returns only `a.js` and `c.js`.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: One `it` case
  - **Depends on**: Tests — directory target, symlinks skipped, Target resolution — file, directory, or glob

- [x] **Tests — sandbox enforcement**
  - **Story**: Paths outside the sandbox are rejected
  - **What**: Assert that passing an absolute path outside the sandbox throws `ForbiddenError`. Assert that `..` in the target string throws `BadRequestError`.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: Two `it` cases
  - **Depends on**: Tests — glob pattern target, Sandbox path helper — shared realpath and within-sandbox check

- [x] **Tests — maxMatches truncation**
  - **Story**: Agent knows when the result list is incomplete
  - **What**: Create a fixture with enough matching lines to exceed `maxMatches: 3` (e.g., 5 matching lines across files). Assert `truncated === true` and `matches.length === 3`.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: One `it` case covering both the flag and the count
  - **Depends on**: Tests — sandbox enforcement

- [x] **Tests — maxDepth respected during directory walk**
  - **Story**: Deep directory trees are bounded
  - **What**: Create a fixture with matching files at depth 1, 3, and 6. Construct the tool with `maxDepth: 4`. Assert files at depth > 4 produce no matches.
  - **Where**: `test/lib/tools/grep-tool-test.js`
  - **Acceptance criteria**: One `it` case
  - **Depends on**: Tests — maxMatches truncation
