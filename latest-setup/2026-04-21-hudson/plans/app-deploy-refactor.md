# Plan: Refactor `hudson.js app deploy` command

## Implementation Approach

The `commands/app/deploy.js` command currently depends on context fields (`applicationDir`) and a hard-coded config import path that no longer reflect how the harness works. We will (1) make `applicationDir` a positional CLI argument resolved inside the command, (2) read `config.json` from that directory at runtime rather than importing a JS module, and (3) relocate `UsageError` out of `hudson.js` into the `app/` directory (app-specific code) where it will extend `WrappedError` from `lib/kixx-server-errors/`. While doing this, we'll also fix a broken import path for the upload worker module, make file discovery use a single `readdir` pass with Dirents, make `main_module` configurable via `config.json`, validate that the configured entry point is present, and tighten a few smaller inconsistencies (spurious `async`, explicit `return 0`, dry-run requiring Cloudflare secrets). The `lib/` directory is reserved for the framework library and must not receive newly factored-out code — all new modules land in `app/`.

## User stories covered

- **US-1**: As a CLI user, I invoke `hudson.js app deploy <applicationDir>` with the application path as a positional argument, so the harness no longer needs to inject `applicationDir`.
- **US-2**: As a CLI user, my `config.json` lives next to my application source in `<applicationDir>/config.json` and is read at runtime, so I can deploy any application without editing harness files.
- **US-3**: As a maintainer, `UsageError` lives in `app/` (app-specific) and extends `WrappedError`, giving it a stable `code`, `expected` flag, and consistent stack capture — and the harness imports it from there.
- **US-4**: As a CLI user, I can configure the Worker entry point via `config.json` (`main` / `main_module`), so my app isn't forced to use `index.js`.
- **US-5**: As a CLI user, I get a clear `UsageError` early when my `applicationDir`, `config.json`, or configured entry point is missing or invalid, before any network call.
- **US-6**: As a CLI user, I can run `--dry-run` without Cloudflare secrets configured, so I can preview a deploy locally.
- **US-7**: As a maintainer, the `deploy` command imports `UploadWorkerModule` from the correct path (`app/cloudflare/upload-worker-module.js`) so the command actually runs.

---

## TODO

- [x] **Create app-specific `UsageError` class**
  - **Story**: US-3
  - **What**: Add a `UsageError` class extending `WrappedError` from `lib/kixx-server-errors/mod.js`. Set `static CODE = 'USAGE_ERROR'`, default `expected: true`, and pass `sourceFunction` through so stack frames are clean. Export as the default export.
  - **Where**: `app/cli/usage-error.js` (new file)
  - **Acceptance criteria**: Instances are `instanceof Error`, `instanceof WrappedError`, and `instanceof UsageError`. `err.code === 'USAGE_ERROR'` by default. `err.expected === true` by default.
  - **Depends on**: none

- [x] **Switch harness to import `UsageError` from `app/`**
  - **Story**: US-3
  - **What**: Remove the inline `export class UsageError extends Error {}` from `hudson.js`. Import `UsageError` from `./app/cli/usage-error.js`. Keep the `instanceof UsageError` check in the top-level catch so usage errors still print just `error.message` (not the stack).
  - **Where**: `hudson.js`
  - **Acceptance criteria**: `hudson.js` no longer defines `UsageError` itself. Running `hudson.js` with no command still prints the usage message and exits 1. Throwing `UsageError` from any command still results in message-only output (no stack trace).
  - **Depends on**: Create app-specific `UsageError` class

- [x] **Update `deploy.js` to import `UsageError` from `app/`**
  - **Story**: US-3
  - **What**: Replace `import { UsageError } from '../../cli.js'` with `import UsageError from '../../app/cli/usage-error.js'`.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: The existing `throw new UsageError(...)` sites still function; no reference to `cli.js` remains.
  - **Depends on**: Create app-specific `UsageError` class

- [x] **Fix `UploadWorkerModule` import path**
  - **Story**: US-7
  - **What**: Change the import from `'../../lib/upload-worker-module.js'` to `'../../app/cloudflare/upload-worker-module.js'` to match the file's actual location.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: Running `node hudson.js app deploy <dir>` resolves the module without `ERR_MODULE_NOT_FOUND`.
  - **Depends on**: none

- [x] **Add an application config loader**
  - **Story**: US-2, US-5
  - **What**: Create a small helper (default export) that takes an absolute `applicationDir`, reads `<applicationDir>/config.json` synchronously, parses it, and returns the parsed object. On `ENOENT`, throw `UsageError('config.json not found at <path>')`. On `SyntaxError`, throw `UsageError('Invalid JSON in <path>: <msg>', { cause: err })`. Validate that the parsed value is a non-null object and that `config.name` is a non-empty string; otherwise throw `UsageError`.
  - **Where**: `app/deploy/application-config.js`
  - **Acceptance criteria**: Returns parsed config for a valid file. Throws `UsageError` (not a generic `Error`) for missing file, invalid JSON, non-object root, or missing/empty `name`. Error messages include the file path.
  - **Depends on**: Create app-specific `UsageError` class

- [x] **Accept `applicationDir` as a positional argument**
  - **Story**: US-1, US-5
  - **What**: In `deploy.js`'s `main`, read `context.args.positionals[0]`. If absent, throw `UsageError('applicationDir positional argument is required')`. Resolve to absolute with `path.resolve(process.cwd(), positional)`. Verify the directory exists via `fs.statSync(...).isDirectory()`; on `ENOENT` or non-directory, throw `UsageError` with the resolved path. Stop reading `context.applicationDir` anywhere in the file (including the JSDoc).
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: `node hudson.js app deploy` (no arg) prints a usage error and exits 1. `node hudson.js app deploy ./nonexistent` prints a usage error. `node hudson.js app deploy <validDir>` proceeds to load config.
  - **Depends on**: Update `deploy.js` to import `UsageError` from `app/cli/`

- [x] **Load `config.json` from `applicationDir` at runtime**
  - **Story**: US-2, US-5
  - **What**: Remove `import config from '../../../config.js'`. After resolving `applicationDir`, call the new config loader to obtain `config`. Use the returned object for `config.name` and the new `main_module` lookup (next task). Remove the separate `if (!config.name)` check — the loader now handles it.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: No static import of a JS config remains. Missing/invalid `<applicationDir>/config.json` surfaces as a `UsageError`. A valid config makes the command proceed to file discovery.
  - **Depends on**: Add a config loader in `app/`, Accept `applicationDir` as a positional argument

- [x] **Make `main_module` configurable via `config.json`**
  - **Story**: US-4
  - **What**: Read `config.main` (fallback: `'index.js'`) and use it as `metadata.main_module`. If present in config, validate it's a non-empty string ending in `.js`; otherwise throw `UsageError`.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: When `config.json` omits `main`, metadata uses `'index.js'`. When it sets `main: 'worker.js'`, metadata uses `'worker.js'`. An invalid value throws `UsageError`.
  - **Depends on**: Load `config.json` from `applicationDir` at runtime

- [x] **Tighten `discoverSourceFiles` using Dirents**
  - **Story**: US-1 (cleanup while we're here)
  - **What**: Drop the `async` keyword (no async I/O inside). Switch to `fs.readdirSync(applicationDir, { recursive: true, withFileTypes: true })`. Filter Dirents with `entry.isFile() && entry.name.endsWith('.js')`. Build the relative path from `entry.parentPath` (or `entry.path`) + `entry.name`, relative to `applicationDir`. Keep the ENOENT→`UsageError` translation and the "no .js files" `UsageError`. Update the caller to no longer `await` it.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: Nested `.js` files are discovered with correct forward-slash relative paths. No extra `statSync` per entry. ENOENT still produces a `UsageError`.
  - **Depends on**: Accept `applicationDir` as a positional argument

- [x] **Validate that the entry-point file is among discovered files**
  - **Story**: US-4, US-5
  - **What**: After discovery, assert that `metadata.main_module` is present in the list of discovered relative paths. If not, throw `UsageError('Entry point <main> not found in <applicationDir>')`.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: A deploy where the configured entry point is missing fails fast with a `UsageError`, before any upload attempt.
  - **Depends on**: Make `main_module` configurable via `config.json`, Tighten `discoverSourceFiles` using Dirents

- [x] **Skip Cloudflare secrets validation for `--dry-run`**
  - **Story**: US-6
  - **What**: Move the `requiredSecrets` check so it runs only when `options['dry-run']` is falsy. Dry-run should still print script name, namespace (or `<not configured>` if secret missing), and discovered files.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: `--dry-run` succeeds without any Cloudflare secrets in `.secrets.json`. A real deploy still fails with `UsageError` when secrets are missing.
  - **Depends on**: Update `deploy.js` to import `UsageError` from `app/`

- [x] **Refresh JSDoc and remove stale context fields**
  - **Story**: US-1
  - **What**: Update the `main` JSDoc: remove `context.applicationDir`; document `context.args.positionals[0]` as the application directory; note that `config.json` is loaded from that directory. Remove the explicit `return 0` noise if trivial (harness coalesces), or leave as-is — at minimum make the doc match reality.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: JSDoc matches the new signature. No mention of `context.applicationDir` remains.
  - **Depends on**: Accept `applicationDir` as a positional argument, Load `config.json` from `applicationDir` at runtime

- [x] **Manual smoke test**
  - **Story**: All
  - **What**: Create a temporary app dir with `config.json` (`{ "name": "hello", "main": "index.js" }`) and an `index.js`. Run: (a) `node hudson.js app deploy` → usage error; (b) `node hudson.js app deploy /bad/path` → usage error; (c) `node hudson.js app deploy <tmp> --dry-run` with no secrets → prints plan and exits 0; (d) with a bad `main` in config → entry-point usage error; (e) with invalid JSON in `config.json` → usage error citing the file path.
  - **Where**: n/a (manual)
  - **Acceptance criteria**: Each scenario produces the expected outcome with a single-line error (no stack) for usage errors.
  - **Depends on**: all prior items
