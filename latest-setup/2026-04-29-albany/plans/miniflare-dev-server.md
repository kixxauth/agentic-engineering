# Plan: Add `hudson.js dev server` for Miniflare-backed local development

## Implementation Approach

The smallest coherent implementation is a thin `dev server` command that reuses the existing application-loading path (`ApplicationDirectory` and `ApplicationConfig`) and delegates Miniflare runtime concerns to a dedicated `app/dev/miniflare-server.js` module. That keeps CLI parsing, validation, and user-facing reporting separate from the mechanics of constructing Miniflare options, waiting for readiness, and disposing the server cleanly. The worker is fed to Miniflare via `scriptPath` + `modules: true` with `modulesRoot` set to the app's `src/` directory, so Miniflare auto-discovers imports exactly the way the production runtime resolves modules. Cross-cutting concerns are fail-fast validation of the application path and entry module, deterministic host/port handling, live-reload of HTML responses for a pleasant dev loop, and a clean shutdown path on `SIGINT`/`SIGTERM` so local development does not leak long-running processes. The plan also includes targeted automated coverage plus a final smoke test against `../kingston-poc` so the command is verified end to end with a real application.

- [x] **Create `dev` command group metadata**
  - **Story**: DS-1 Discover the local-development command
  - **What**: Add a new command group index that exposes `dev` as a top-level CLI group and advertises the `server` subcommand so it shows up in help and command listings.
  - **Where**: `commands/dev/index.js`
  - **Acceptance criteria**: DS-1.AC1 (`node hudson.js --help` lists `dev`), DS-1.AC2 (`node hudson.js dev --help` lists `server`)
  - **Depends on**: none

- [x] **Build the Miniflare lifecycle wrapper**
  - **Story**: DS-2 Start a local HTTP server for a Worker app; DS-3 Reuse application config in development; DS-4 Control the network bind; DS-6 Shut the server down cleanly
  - **What**: Create a dedicated module that owns Miniflare construction, startup, readiness, and disposal. It should accept the resolved `scriptPath` under `<applicationDir>/src/<main_module>`, enable `modules: true`, set `modulesRoot` to the app's `src/` directory for clean module resolution and stack traces, enable `liveReload` so HTML responses pick up worker changes, and pass through `compatibility_date` and `compatibility_flags` from the app config along with the chosen host and port.
  - **Where**: `app/dev/miniflare-server.js`
  - **Acceptance criteria**: DS-2.AC1 (the wrapper starts Miniflare from the configured entry module), DS-3.AC2 (`main_module`, `compatibility_date`, and `compatibility_flags` reach Miniflare), DS-4.AC1 (default bind settings can be applied by the caller), DS-6.AC1 (the wrapper exposes readiness and disposal cleanly)
  - **Depends on**: none

- [x] **Implement the `dev server` command**
  - **Story**: DS-2 Start a local HTTP server for a Worker app; DS-3 Reuse application config in development; DS-4 Control the network bind; DS-5 Surface actionable local-dev errors; DS-6 Shut the server down cleanly
  - **What**: Add the new CLI subcommand as a thin command class. It should resolve the application directory, load app config, validate that `src/` exists and that the configured `main_module` exists beneath it, parse `--host`, `--port`, and `--no-live-reload`, start the Miniflare wrapper, print a ready message with the resolved URL, and keep the process alive until interrupted while ensuring `dispose()` runs in a `finally` path. `SIGINT`/`SIGTERM` handlers must trigger shutdown and resolve the run promise with exit code 0.
  - **Where**: `commands/dev/server.js`
  - **Acceptance criteria**: DS-2.AC2 (`node hudson.js dev server <applicationDir>` starts the server and reports its URL), DS-2.AC3 (HTTP requests are served by the Worker), DS-3.AC1 (the command reuses `ApplicationDirectory` and `ApplicationConfig` instead of duplicating app config parsing), DS-4.AC1 (defaults to `127.0.0.1:8787`), DS-4.AC2 (`--host` and `--port` override the defaults), DS-4.AC3 (invalid port values fail fast), DS-5.AC1 (missing app path, `src/`, or entrypoint throws `UsageError` before startup), DS-5.AC2 (startup failures such as bind conflicts surface actionable output), DS-6.AC2 (`SIGINT`/`SIGTERM` disposes the server and exits cleanly)
  - **Depends on**: Create `dev` command group metadata, Build the Miniflare lifecycle wrapper

- [x] **Add metadata coverage for the new command group**
  - **Story**: DS-1 Discover the local-development command
  - **What**: Add a focused test that imports the real `commands/dev/index.js` module and asserts the exported group description and `server` subcommand metadata, so discoverability is verified without relying on manual help checks.
  - **Where**: `test/commands/dev/index-test.js`
  - **Acceptance criteria**: DS-1.AC1 (the `dev` group is exposed), DS-1.AC2 (the `server` subcommand is advertised with the expected description)
  - **Depends on**: Create `dev` command group metadata

- [x] **Add unit tests for the Miniflare wrapper**
  - **Story**: DS-2 Start a local HTTP server for a Worker app; DS-3 Reuse application config in development; DS-4 Control the network bind; DS-6 Shut the server down cleanly
  - **What**: Add wrapper tests that verify the Miniflare constructor receives the expected options (including `scriptPath`, `modules: true`, `modulesRoot` set to the app's `src/`, `liveReload`, and the pass-through compatibility fields) and that the wrapper exposes readiness and disposal correctly. Keep the tests deterministic by using a fake Miniflare implementation or similarly isolated test double rather than a long-lived real process.
  - **Where**: `test/app/dev/miniflare-server-test.js`
  - **Acceptance criteria**: DS-2.AC1 (wrapper maps the resolved script path into Miniflare), DS-3.AC2 (compatibility settings are passed through), DS-4.AC1 (host and port are forwarded), DS-6.AC1 (`dispose()` is called exactly once on shutdown)
  - **Depends on**: Build the Miniflare lifecycle wrapper

- [x] **Add command-level tests for startup, validation, and shutdown**
  - **Story**: DS-2 Start a local HTTP server for a Worker app; DS-4 Control the network bind; DS-5 Surface actionable local-dev errors; DS-6 Shut the server down cleanly
  - **What**: Add tests for the `dev server` command covering default host/port behavior, CLI overrides, `--no-live-reload` handling, startup reporting, missing `src/` or entry module errors, invalid port handling, and the shutdown path. Use injected test doubles for the server wrapper and reporter so the tests can assert orchestration without binding a real TCP port.
  - **Where**: `test/commands/dev/server-test.js`
  - **Acceptance criteria**: DS-2.AC2 (the command starts the wrapper and reports the ready URL), DS-4.AC1 (defaults are applied when options are omitted), DS-4.AC2 (host/port overrides are honored), DS-4.AC3 (bad port values raise `UsageError`), DS-5.AC1 (invalid app layout fails before startup), DS-6.AC2 (the command disposes the wrapper on shutdown)
  - **Depends on**: Implement the `dev server` command

- [x] **Document `dev server` usage**
  - **Story**: DS-7 Make the new workflow understandable to CLI users
  - **What**: Update the project README so the new command appears in the CLI overview and command reference, including the `<applicationDir>` argument, default host/port behavior, live-reload behavior, and an example invocation against a worker app.
  - **Where**: `README.md`
  - **Acceptance criteria**: DS-7.AC1 (README documents `node hudson.js dev server <applicationDir> [--host <host>] [--port <port>] [--no-live-reload]`), DS-1.AC1 (top-level command overview includes `dev`)
  - **Depends on**: Create `dev` command group metadata, Implement the `dev server` command

- [x] **Run an end-to-end smoke test against `../kingston-poc`**
  - **Story**: DS-7 Verify the real developer workflow end to end
  - **What**: Start the new dev server with the example application, wait for the ready URL, make an HTTP request to the local server, verify the response body matches the sample worker output, and then stop the process to confirm cleanup.
  - **Where**: manual verification using `../kingston-poc/`
  - **Acceptance criteria**: DS-7.AC2 (`node hudson.js dev server ../kingston-poc` serves the example app over HTTP), DS-2.AC3 (the response body matches `Hello World` returned by `src/index.js`), DS-6.AC2 (the process exits cleanly after interruption)
  - **Depends on**: Add unit tests for the Miniflare wrapper, Add command-level tests for startup, validation, and shutdown, Document `dev server` usage
