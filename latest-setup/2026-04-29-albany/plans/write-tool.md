# WriteTool Implementation Plan

## Implementation Approach

`WriteTool.write()` creates or replaces a file within a sandboxed directory, following the same path-safety pattern established by `ReadTool` and `EditTool`. The three private helpers — `#assertSafePath`, `#resolveThroughDeepestExistingAncestor`, and `#isWithinSandbox` — are duplicated verbatim from those sibling classes (no shared base class exists yet). The write path must create any missing intermediate directories before writing so agents can create files in new subdirectories. The method accepts a UTF-8 string and writes it atomically via `fsp.writeFile`. Tests mirror the fixture/teardown pattern already used in `edit-tool-test.js` and `read-tool-test.js`.

---

## TODO

- [x] **Add imports and path-safety helpers**
  - **Story**: Sandbox enforcement
  - **What**: Add `fsp`, `path`, `BadRequestError`, and `ForbiddenError` imports. Implement the three private methods `#assertSafePath`, `#resolveThroughDeepestExistingAncestor`, and `#isWithinSandbox` — identical in structure to `EditTool` but with "Write" in error messages.
  - **Where**: `lib/tools/write-tool.js`
  - **Acceptance criteria**: Relative paths throw `BadRequestError`; paths outside the sandbox (including symlinks that escape) throw `ForbiddenError`; paths inside the sandbox resolve to their real absolute path.
  - **Depends on**: none

- [x] **Implement `write()` method body**
  - **Story**: Create or replace a file
  - **What**: Resolve the safe path, create any missing parent directories with `fsp.mkdir({ recursive: true })`, then write `content` as UTF-8 with `fsp.writeFile`. Return `true` on success.
  - **Where**: `lib/tools/write-tool.js`
  - **Acceptance criteria**: A new file is created with the given content; an existing file has its content fully replaced; parent directories are created when they do not exist; the method returns `true`.
  - **Depends on**: Add imports and path-safety helpers

- [x] **Write path-safety tests**
  - **Story**: Sandbox enforcement
  - **What**: Create a test file with a `createSandboxFixture` / `removeFixture` helper (temp dir, sandbox subdir, outside subdir, symlink escaping the sandbox). Add `describe('path safety')` cases: rejects relative paths (`BadRequestError`), rejects paths outside the sandbox (`ForbiddenError`), rejects symlinks that resolve outside the sandbox (`ForbiddenError`), accepts paths inside the sandbox.
  - **Where**: `test/lib/tools/write-tool-test.js`
  - **Acceptance criteria**: All three rejection cases throw the correct named error; the acceptance case does not throw.
  - **Depends on**: Implement `write()` method body

- [x] **Write file-creation and replacement tests**
  - **Story**: Create or replace a file
  - **What**: Add `describe('write')` cases: creates a new file with correct UTF-8 content, replaces the full content of an existing file, creates intermediate parent directories when they do not exist, returns `true` on success.
  - **Where**: `test/lib/tools/write-tool-test.js`
  - **Acceptance criteria**: File content after `write()` matches the supplied string exactly; missing parents are created; return value is `true`.
  - **Depends on**: Write path-safety tests
