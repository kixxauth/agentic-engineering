---
name: EditTool and ReadTool implementation plan
description: Plan to implement edit() on EditTool and read() on ReadTool with sandbox safety, input validation, image support, and tests
---

# EditTool and ReadTool Implementation Plan

## Implementation Approach

Both `EditTool` and `ReadTool` share the same sandbox-safety requirement as `TextEditTool`: every `file_path` must be absolute and must resolve (after following symlinks on the deepest existing ancestor) to a path within `#sandboxDirectory`. Rather than sharing a module, each class carries its own private `#assertSafePath` helper — the logic is a direct port from `TextEditTool` so it is easy to verify in isolation. `EditTool.edit` extends `TextEditTool.str_replace` with a `replace_all` flag that skips the single-occurrence requirement and replaces every instance. `ReadTool.read` adds pagination (`limit`/`offset`) and image detection — image files are returned as a raw Base64 string, while text files are returned in `cat -n` format with line numbers that reflect the original file positions (offset shifts the starting line number). Tests mirror the `text-edit-tool-test.js` pattern: a temp-directory sandbox is created per test suite and torn down afterward.

## Tasks

- [x] **Add private path-safety helper to EditTool**
  - **Story**: Sandbox safety for EditTool
  - **What**: Add `#assertSafePath(file_path)` that throws `BadRequestError` when `file_path` is not absolute, and `ForbiddenError` when the resolved real path (via `fsp.realpath` on the deepest existing ancestor) falls outside `#sandboxDirectory`. Return the resolved safe path. Port the logic directly from `TextEditTool#assertSafePath` and its private helpers `#resolveThroughDeepestExistingAncestor` and `#isWithinSandbox`.
  - **Where**: `lib/tools/edit-tool.js`
  - **Acceptance criteria**: Throws `BadRequestError` for relative paths; throws `ForbiddenError` for paths that escape the sandbox via direct traversal or symlink; accepts a path whose last segment does not yet exist.
  - **Depends on**: none

- [x] **Implement `EditTool.edit` for single-replacement mode**
  - **Story**: Replace exact text in a file (single occurrence)
  - **What**: Validate `file_path` via `#assertSafePath`. Assert `old_str` is a non-empty string and `new_str` is a string (empty allowed) and `new_str !== old_str` (throw `BadRequestError` otherwise). Read the file as UTF-8. Count non-overlapping occurrences of `old_str`. When `replace_all` is falsy and the count is 0, throw `BadRequestError`. When `replace_all` is falsy and the count is greater than 1, throw `BadRequestError`. Perform `String.replace` (first occurrence only), write the result as UTF-8, return `true`.
  - **Where**: `lib/tools/edit-tool.js` — `edit()`
  - **Acceptance criteria**: Throws on 0 matches; throws on >1 match when `replace_all` is false; returns `true` and updates the file on success; `new_str === old_str` throws before any I/O.
  - **Depends on**: Add private path-safety helper to EditTool

- [x] **Extend `EditTool.edit` for replace-all mode**
  - **Story**: Replace all occurrences of a string in a file
  - **What**: When `replace_all` is `true`, skip the multiple-occurrence guard. Use `String.replaceAll` (or a global-regex equivalent) to replace every instance of `old_str` with `new_str`. Still throw when the occurrence count is 0.
  - **Where**: `lib/tools/edit-tool.js` — `edit()`
  - **Acceptance criteria**: All occurrences are replaced when `replace_all` is `true`; 0-match still throws; returns `true` on success.
  - **Depends on**: Implement `EditTool.edit` for single-replacement mode

- [x] **Add private path-safety helper to ReadTool**
  - **Story**: Sandbox safety for ReadTool
  - **What**: Add `#assertSafePath(file_path)` with identical semantics to the one in `EditTool`: rejects relative paths with `BadRequestError`, rejects sandbox escapes with `ForbiddenError`, resolves symlinks through the deepest existing ancestor. Return the resolved safe path.
  - **Where**: `lib/tools/read-tool.js`
  - **Acceptance criteria**: Same sandbox-safety guarantees as `EditTool#assertSafePath`.
  - **Depends on**: none

- [x] **Implement `ReadTool.read` for image files**
  - **Story**: Read an image file as Base64
  - **What**: After resolving a safe path, check whether the file's extension (lowercased) is in `SUPPORTED_IMAGE_FILE_EXTENSIONS`. If so, read the file with `fsp.readFile` (no encoding), convert the resulting `Buffer` to a Base64 string via `buffer.toString('base64')`, and return it directly (no line numbers).
  - **Where**: `lib/tools/read-tool.js` — `read()`
  - **Acceptance criteria**: Returns a Base64 string for `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp` files; the decoded Base64 round-trips to the original bytes.
  - **Depends on**: Add private path-safety helper to ReadTool

- [x] **Implement `ReadTool.read` for text files (full read)**
  - **Story**: Read a text file in cat -n format
  - **What**: For non-image files, read as UTF-8, split on `\n`, and format every line with a right-aligned line number (padded to 6 characters) and a tab, starting at line 1 — identical to the `#formatNumberedLines` / `#selectViewLines` pattern in `TextEditTool`.
  - **Where**: `lib/tools/read-tool.js` — `read()`
  - **Acceptance criteria**: Returns all lines numbered from 1; format matches `cat -n` (6-char right-aligned number, then tab, then line content); UTF-8 content round-trips intact.
  - **Depends on**: Add private path-safety helper to ReadTool

- [x] **Implement `ReadTool.read` pagination (offset + limit)**
  - **Story**: Paginate through a large text file
  - **What**: When `offset` is defined, validate it is an integer `>= 0` (throw `BadRequestError` if not); validate it is not greater than `lines.length` (throw `BadRequestError` if so). When `limit` is defined, validate it is a positive integer (throw `BadRequestError` otherwise). Slice `lines` from `offset` to `offset + limit` (or to end when `limit` is absent). Line numbers in the output must reflect original file positions — the first returned line is numbered `offset + 1`.
  - **Where**: `lib/tools/read-tool.js` — `read()`
  - **Acceptance criteria**: `offset=0` returns from the first line; `offset=N` starts at line `N+1`; `limit` caps the number of returned lines; line numbers in output always reflect the original file; `offset > lines.length` throws; `offset < 0` throws; `limit` of 0 or negative throws.
  - **Depends on**: Implement `ReadTool.read` for text files (full read)

- [x] **Tests for EditTool sandbox safety**
  - **Story**: Sandbox safety for EditTool
  - **What**: Test cases confirming that `edit()` rejects relative paths, paths outside the sandbox, and symlinks that escape the sandbox. Use a temp-directory fixture with a sandboxed file, an outside file, and a symlink pointing outside.
  - **Where**: `test/lib/tools/edit-tool-test.js` (new)
  - **Acceptance criteria**: Each escape vector causes a `ForbiddenError` or `BadRequestError`; the fixture is created in setup and removed in teardown.
  - **Depends on**: Implement `EditTool.edit` for single-replacement mode

- [x] **Tests for `EditTool.edit` — single replacement**
  - **Story**: Replace exact text in a file (single occurrence)
  - **What**: Cover: successful replacement returns `true` and mutates the file; `new_str === old_str` throws; `old_str` not found throws; multiple occurrences with `replace_all` falsy throws; non-string `old_str` or `new_str` throws.
  - **Where**: `test/lib/tools/edit-tool-test.js`
  - **Acceptance criteria**: All documented single-replacement behaviors and error cases are exercised.
  - **Depends on**: Tests for EditTool sandbox safety

- [x] **Tests for `EditTool.edit` — replace-all mode**
  - **Story**: Replace all occurrences of a string in a file
  - **What**: Cover: all occurrences replaced when `replace_all=true`; 0-match still throws even when `replace_all=true`; multiple occurrences are all replaced and file content is correct.
  - **Where**: `test/lib/tools/edit-tool-test.js`
  - **Acceptance criteria**: All replace-all behaviors and error cases are exercised.
  - **Depends on**: Tests for `EditTool.edit` — single replacement

- [x] **Tests for ReadTool sandbox safety**
  - **Story**: Sandbox safety for ReadTool
  - **What**: Test cases confirming `read()` rejects relative paths, outside-sandbox paths, and symlinks that escape the sandbox. Reuse the same temp-directory fixture pattern.
  - **Where**: `test/lib/tools/read-tool-test.js` (new)
  - **Acceptance criteria**: Each escape vector is rejected with the appropriate error name; fixture is created and cleaned up per suite.
  - **Depends on**: Implement `ReadTool.read` for text files (full read)

- [x] **Tests for `ReadTool.read` — text file, full read**
  - **Story**: Read a text file in cat -n format
  - **What**: Cover: full file returns all lines with correct cat -n numbering; a single-line file works; a file with no trailing newline works.
  - **Where**: `test/lib/tools/read-tool-test.js`
  - **Acceptance criteria**: All documented full-read behaviors are exercised; line number format matches cat -n exactly.
  - **Depends on**: Tests for ReadTool sandbox safety

- [x] **Tests for `ReadTool.read` — pagination**
  - **Story**: Paginate through a large text file
  - **What**: Cover: `offset` shifts the slice and line numbers; `limit` caps lines returned; `offset + limit` stops before EOF; `offset === lines.length` returns empty (or throws — document which); `offset < 0` throws; `offset > lines.length` throws; non-integer `offset` throws; `limit <= 0` throws.
  - **Where**: `test/lib/tools/read-tool-test.js`
  - **Acceptance criteria**: All pagination edge cases and error paths are covered; returned line numbers always reflect original file positions.
  - **Depends on**: Tests for `ReadTool.read` — text file, full read

- [x] **Tests for `ReadTool.read` — image files**
  - **Story**: Read an image file as Base64
  - **What**: Write a small binary fixture to the sandbox, call `read()` with a `.png` path, assert the return value is a Base64 string that decodes to the original bytes. Also confirm that an unsupported extension (e.g. `.txt`) is not treated as an image.
  - **Where**: `test/lib/tools/read-tool-test.js`
  - **Acceptance criteria**: Base64 result decodes to original bytes; supported extensions are all handled; non-image extension falls through to text path.
  - **Depends on**: Tests for ReadTool sandbox safety

- [ ] **Lint pass**
  - **Story**: Project hygiene (cross-cutting)
  - **What**: Run `node lint.js lib/tools/edit-tool.js lib/tools/read-tool.js test/lib/tools/edit-tool-test.js test/lib/tools/read-tool-test.js` and fix any reported issues. Run `npm test` to confirm all tests pass.
  - **Where**: n/a (verification step)
  - **Acceptance criteria**: `node lint.js` exits clean for all four files; `npm test` passes.
  - **Depends on**: Tests for `EditTool.edit` — replace-all mode, Tests for `ReadTool.read` — image files, Tests for `ReadTool.read` — pagination
