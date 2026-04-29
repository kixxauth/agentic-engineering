---
name: TextEditTool implementation plan
description: Plan to implement view, str_replace, create, and insert on lib/tools/text-edit-tool.js with sandbox safety, input validation, and tests
---

# TextEditTool Implementation Plan

## Implementation Approach

The four public methods on `TextEditTool` (`view`, `str_replace`, `create`, `insert`) all share two cross-cutting concerns: argument validation and sandbox-safe path resolution. Both are extracted into private helpers (`#assertSafePath`, plus per-method argument assertions) so each method body stays focused on the file-system action it owns. Path safety must defeat both relative paths and symlink escapes, so resolution goes through `fs.realpath` on the parent directory rather than a naive `path.resolve` + `startsWith` check. All file I/O is UTF-8. Tests live in `test/lib/tools/text-edit-tool-test.js` using `kixx-test` + `kixx-assert`, exercising each documented error condition against a temporary sandbox directory created and torn down per test.

## Tasks

- [x] **Add private path-safety helper**
  - **Story**: Sandbox safety (cross-cutting)
  - **What**: Add `#assertSafePath(file_path)` private method that throws when `file_path` is not absolute or resolves (after `fs.realpath` on the deepest existing ancestor) outside `#sandboxDirectory`. Use `path.isAbsolute`, `path.resolve`, and `fsp.realpath` so symlinks that escape the sandbox are rejected. Returns the resolved absolute path for the caller to use.
  - **Where**: `lib/tools/text-edit-tool.js`
  - **Acceptance criteria**: Throws on relative paths; throws when the resolved real path is outside `#sandboxDirectory`; tolerates a `file_path` whose final segment does not yet exist (needed for `create`) by realpath-ing the parent.
  - **Depends on**: none

- [x] **Add range-argument validator for `view`**
  - **Story**: View a file with optional line range
  - **What**: Add a private helper (or inline guard) that, when `range` is provided, asserts it is a 2-element array of integers, that `start >= 1`, that `end === -1` or `end >= start`, and that `range` is not supplied alongside a directory target. Decide and document the error when `start` exceeds the file's line count (throw with a clear message).
  - **Where**: `lib/tools/text-edit-tool.js`
  - **Acceptance criteria**: Rejects non-array, wrong-length, non-integer, reversed, and out-of-bounds ranges; rejects `range` passed with a directory `file_path`.
  - **Depends on**: none

- [x] **Implement `view` for files**
  - **Story**: View a file with optional line range
  - **What**: Read the file as UTF-8, split on `\n`, slice by `range` (1-indexed, `-1` end means "to EOF"), and return the selected lines prefixed with `cat -n`-style line numbers (right-aligned, tab-separated, line numbers reflect the original file, not the slice).
  - **Where**: `lib/tools/text-edit-tool.js` — `view()`
  - **Acceptance criteria**: Returns numbered lines; line numbers match the original file when a range is used; whole-file read returns every line numbered from 1; UTF-8 content round-trips intact.
  - **Depends on**: Add private path-safety helper, Add range-argument validator for `view`

- [x] **Implement `view` for directories**
  - **Story**: List directory contents
  - **What**: When `file_path` is a directory, read its entries with `fsp.readdir`, map each to an absolute path joined under `file_path`, and return them joined by `\n`.
  - **Where**: `lib/tools/text-edit-tool.js` — `view()`
  - **Acceptance criteria**: Returns each entry as an absolute path on its own line; order is stable (sort entries) so callers can rely on it; empty directories return an empty string.
  - **Depends on**: Add private path-safety helper

- [x] **Implement `str_replace`**
  - **Story**: Replace exact text in a file
  - **What**: Read the file as UTF-8, assert `old_str` is a non-empty string and `new_str` is a string and that `new_str !== old_str`, count occurrences of `old_str`, throw if zero matches or more than one match (matching Anthropic's reference text-editor semantics), perform the single replacement, and write the file back as UTF-8.
  - **Where**: `lib/tools/text-edit-tool.js` — `str_replace()`
  - **Acceptance criteria**: Throws on zero matches; throws on multiple matches; throws when `new_str === old_str`; on success, returns `true` and the file content reflects exactly one replacement; original line endings are preserved.
  - **Depends on**: Add private path-safety helper

- [x] **Implement `create`**
  - **Story**: Create a new file with content
  - **What**: Assert `text` is a string (empty allowed), throw if the target already exists (`fsp.stat` then `EEXIST`-style error), `fsp.mkdir(parent, { recursive: true })` to create parents, then `fsp.writeFile` as UTF-8. Return `true`.
  - **Where**: `lib/tools/text-edit-tool.js` — `create()`
  - **Acceptance criteria**: Throws if the file already exists; succeeds when only the parent chain is missing; rejects non-string `text`; written content matches input byte-for-byte.
  - **Depends on**: Add private path-safety helper

- [x] **Implement `insert`**
  - **Story**: Insert text at a specific line
  - **What**: Assert `insert_line` is a non-NaN integer `>= 0`, `insert_text` is a string. Read the file as UTF-8, split on `\n`, validate `insert_line <= lines.length`, splice `insert_text` in after `insert_line` (0 = beginning of file), join with `\n`, ensure the file ends with a single trailing `\n`, and write back. Return `true`.
  - **Where**: `lib/tools/text-edit-tool.js` — `insert()`
  - **Acceptance criteria**: Inserting at 0 prepends; inserting at `lines.length` appends; insertion between lines adds a line return on both sides; the resulting file always ends with exactly one trailing newline; throws on negative or out-of-range `insert_line`.
  - **Depends on**: Add private path-safety helper

- [x] **Tests for path safety**
  - **Story**: Sandbox safety (cross-cutting)
  - **What**: Test cases that confirm relative paths, paths outside the sandbox, and symlinks pointing outside the sandbox are all rejected by every public method.
  - **Where**: `test/lib/tools/text-edit-tool-test.js` (new)
  - **Acceptance criteria**: Each of the four methods rejects each escape vector; tests use a temp directory created in `setup` and removed in `teardown`.
  - **Depends on**: Implement `view` for files, Implement `view` for directories, Implement `str_replace`, Implement `create`, Implement `insert`

- [x] **Tests for `view`**
  - **Story**: View a file with optional line range / List directory contents
  - **What**: Cover full-file read, ranged read (including `-1` end), directory listing (including empty directory), and every range-validation error case.
  - **Where**: `test/lib/tools/text-edit-tool-test.js`
  - **Acceptance criteria**: All documented `view` behaviors and errors are exercised.
  - **Depends on**: Implement `view` for files, Implement `view` for directories

- [x] **Tests for `str_replace`**
  - **Story**: Replace exact text in a file
  - **What**: Cover successful single replacement, zero-match error, multi-match error, `new_str === old_str` error, and non-string argument errors.
  - **Where**: `test/lib/tools/text-edit-tool-test.js`
  - **Acceptance criteria**: All documented `str_replace` behaviors and errors are exercised.
  - **Depends on**: Implement `str_replace`

- [x] **Tests for `create`**
  - **Story**: Create a new file with content
  - **What**: Cover happy path, already-exists error, parent-directory creation, empty-string content, and non-string `text` rejection.
  - **Where**: `test/lib/tools/text-edit-tool-test.js`
  - **Acceptance criteria**: All documented `create` behaviors and errors are exercised.
  - **Depends on**: Implement `create`

- [x] **Tests for `insert`**
  - **Story**: Insert text at a specific line
  - **What**: Cover insert at 0 (prepend), insert at `lines.length` (append), insert between lines, trailing-newline guarantee on a file that did not previously end in `\n`, negative `insert_line` error, out-of-range `insert_line` error, and non-string `insert_text` error.
  - **Where**: `test/lib/tools/text-edit-tool-test.js`
  - **Acceptance criteria**: All documented `insert` behaviors and errors are exercised.
  - **Depends on**: Implement `insert`

- [x] **Lint pass**
  - **Story**: Project hygiene (cross-cutting)
  - **What**: Run `node lint.js lib/tools/text-edit-tool.js test/lib/tools/text-edit-tool-test.js` and resolve any reported issues.
  - **Where**: n/a (verification step)
  - **Acceptance criteria**: `node lint.js` exits clean for both files; `npm test` passes.
  - **Depends on**: All test tasks
