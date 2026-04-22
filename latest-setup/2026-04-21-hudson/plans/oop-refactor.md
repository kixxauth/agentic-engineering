# OOP Refactor Plan

## Implementation Approach

Move the codebase from its current procedural-with-one-class shape to an object-oriented design in which responsibilities are owned by named entities. Work from the leaves inward: introduce small domain classes in `app/` first (they have no internal dependencies), refactor `commands/app/deploy.js` to compose them, then rework the `hudson.js` harness into collaborating objects, and finally unify the command-module contract. The reference shapes are `app/cloudflare/upload-worker-module.js` (private fields, construct-with-config, action/query methods) and `app/cli/usage-error.js` (typed errors extending `WrappedError`). Cross-cutting concerns: every new class gets a precise name (noun for things, verb phrase for actions), validation and typed errors stay at input boundaries, and the CLI harness must not leak specialized concerns (Cloudflare, config schema) nor should commands reimplement harness concerns (argv, help, secrets). Do not decompose for its own sake — keep closely related code together and only split when information isn't shared or concerns differ.

## User Stories

- **US-1 Secrets ownership**: As a command author, I want a `Secrets` object that validates required keys, so I don't scatter `context.secrets.FOO &&` checks across commands.
- **US-2 Config as a type**: As a command author, I want `ApplicationConfig` to own parsing and main-module resolution, so config shape lives in one place.
- **US-3 Directory as a type**: As a command author, I want `ApplicationDirectory` to own path resolution and existence checks, so commands don't redo `fs.statSync` boilerplate.
- **US-4 Source bundle as a type**: As a command author, I want a `SourceBundle` that discovers and reads `.js` files as one object, so callers iterate contents without knowing how they were discovered.
- **US-5 Deploy as a class**: As a maintainer, I want `DeployCommand` to orchestrate collaborators in a readable `run()` method, so `commands/app/deploy.js` stops mixing five concerns.
- **US-6 Output abstraction**: As a maintainer, I want a `Reporter` (stdout writer) so dry-run output and success messages go through one injectable seam instead of direct `console.log`.
- **US-7 Command registry**: As a harness author, I want `CommandRegistry` to own `commands/` discovery and resolution, so `hudson.js` stops re-deriving paths in four functions.
- **US-8 Argv parser**: As a harness author, I want an `Argv` type that answers `command`/`subcommand`/`remainder`/`helpRequested`, so parsing lives in one place.
- **US-9 Help renderer**: As a harness author, I want `HelpRenderer` as a class so the output stream is injectable and help sections are uniform.
- **US-10 CliContext**: As a command author, I want a `CliContext` class with typed accessors (`positional(i)`, `option(name)`, `.secrets`) so commands don't index into nested plain objects.
- **US-11 Command base contract**: As a maintainer, I want a `Command` base class so every subcommand exposes `static description`, `static options`, and `run(context)` uniformly.
- **US-12 Cli orchestrator**: As a maintainer, I want a `Cli` class whose `run()` replaces the free `main()` so the entry point is just `new Cli(argv).run()`.

## TODO

- [x] **Introduce `Secrets` class**
  - **Story**: US-1
  - **What**: Create a class that loads `.secrets.json` and exposes `.get(key)` and `.require(keys)` (throws `UsageError` listing all missing keys). Move the `ENOENT → warn + empty` behavior from `loadSecrets()` inside the class (static `load(rootDir)` factory). Private field for the underlying map.
  - **Where**: new `app/cli/secrets.js`
  - **Acceptance criteria**: US-1 — `Secrets.load(rootDir)` returns an instance; `.require(['A','B'])` throws one `UsageError` naming every missing key; ENOENT logs a warning and yields an empty instance.
  - **Depends on**: none

- [x] **Promote config loader to `ApplicationConfig` class**
  - **Story**: US-2
  - **What**: Replace the `loadConfig` function with a class. `static load(applicationDir)` returns an instance; expose `.name` and `.mainModule` (the latter absorbing `resolveMainModule` from `deploy.js`, defaulting to `index.js`, validating `.js` suffix). Throw `UsageError` for parse/shape failures as today.
  - **Where**: rewrite `app/load-config.js` → `app/deploy/application-config.js`; delete the old file once no importers remain (done in the deploy-refactor task).
  - **Acceptance criteria**: US-2 — `new ApplicationConfig(...)` is not exported (only the class and its `static load`); `.mainModule` returns `config.main` or `'index.js'` and rejects invalid values with `UsageError`.
  - **Depends on**: none

- [x] **Introduce `ApplicationDirectory` class**
  - **Story**: US-3
  - **What**: Class wrapping an absolute directory path. `static resolve(cwd, positional)` validates the positional, resolves to absolute, and checks it's an existing directory (throws `UsageError` otherwise). Expose `.path` and `.readFile(relPath)` (utf8).
  - **Where**: new `app/deploy/application-directory.js`
  - **Acceptance criteria**: US-3 — constructing via `resolve` with a missing path throws `UsageError`; with a file path throws `UsageError`; `.readFile('index.js')` returns file contents.
  - **Depends on**: none

- [x] **Introduce `SourceBundle` class**
  - **Story**: US-4
  - **What**: Class owning the discover-and-read flow for an application's `.js` files. `static fromDirectory(appDir)` uses `ApplicationDirectory.readFile` to populate an internal list of `{ relativePath, content }`. Expose `.has(relPath)`, `.files` (iterator or array), and `.size`. Throw `UsageError` on empty discovery.
  - **Where**: new `app/deploy/source-bundle.js`
  - **Acceptance criteria**: US-4 — `fromDirectory` throws `UsageError` when no `.js` files found; `.has('index.js')` reflects discovered files; iteration yields `{ relativePath, content }` pairs in deterministic order.
  - **Depends on**: Introduce `ApplicationDirectory` class

- [x] **Introduce `Reporter` abstraction**
  - **Story**: US-6
  - **What**: Tiny class wrapping an output stream (default `process.stdout`). Methods: `.line(msg)`, `.blank()`, `.list(items, format)`. Used by the deploy dry-run branch and success message. No global `console.log` in command code afterward.
  - **Where**: new `app/cli/reporter.js`
  - **Acceptance criteria**: US-6 — `new Reporter()` defaults to stdout; passing a writable stream in tests captures output; `.list` uses a caller-provided formatter.
  - **Depends on**: none

- [x] **Refactor deploy command into `DeployCommand` class**
  - **Story**: US-5
  - **What**: Replace the exported `main` function with a `DeployCommand` class whose `run(context)` composes `ApplicationDirectory`, `ApplicationConfig`, `SourceBundle`, `Secrets.require(...)`, `UploadWorkerModule`, and `Reporter`. Keep the module-level `export const description`/`options` so the existing harness still resolves the command (uniform class contract comes later). The `run` body should read as ~10 lines of collaborator messages; the dry-run branch becomes a `Reporter` call.
  - **Where**: rewrite `commands/app/deploy.js`; remove the now-unused `app/load-config.js` import path (the rewritten file lives at `app/deploy/application-config.js`).
  - **Acceptance criteria**: US-5, US-6 — `deploy.js` contains no direct `fs.*`, no `console.log`, no `resolveApplicationDir`/`resolveMainModule`/`discoverSourceFiles` helpers; `--dry-run` output matches today's format; real deploys still call `UploadWorkerModule.upload()`.
  - **Depends on**: Introduce `Secrets` class, Promote config loader to `ApplicationConfig` class, Introduce `ApplicationDirectory` class, Introduce `SourceBundle` class, Introduce `Reporter` abstraction

- [x] **Introduce `Argv` class**
  - **Story**: US-8
  - **What**: Class constructed from `process.argv`. Exposes `.command`, `.subcommand`, `.remainder`, `.helpRequested`. Absorbs today's `parseArgv()` plus the ad-hoc `--help`/`-h` detection on `hudson.js:158`.
  - **Where**: new `app/cli/argv.js`
  - **Acceptance criteria**: US-8 — `new Argv(['node','hudson.js','app','deploy','--dry-run'])` yields `command='app'`, `subcommand='deploy'`, `remainder=['--dry-run']`, `helpRequested=false`; `--help` anywhere flips `helpRequested`.
  - **Depends on**: none

- [x] **Introduce `HelpRenderer` class**
  - **Story**: US-9
  - **What**: Class wrapping an output stream. `.render(sections)` replaces the free `renderHelp` function. Accept the same `{ usage, description, commands, options }` shape for now.
  - **Where**: new `app/cli/help-renderer.js`
  - **Acceptance criteria**: US-9 — output matches today's formatting byte-for-byte for the three call sites; injectable writer allows test capture.
  - **Depends on**: none

- [x] **Introduce `CommandRegistry` class**
  - **Story**: US-7
  - **What**: Class owning the `commands/` directory. Methods: `listGroups()`, `listSubcommands(group)`, `resolve(group, sub)` (returns the loaded module or throws `UsageError`), `loadGroupIndex(group)`. Absorbs `loadCommandIndex`, `scanCommands`, `scanSubcommands`, `resolveCommand` from `hudson.js`.
  - **Where**: new `app/cli/command-registry.js`
  - **Acceptance criteria**: US-7 — behavior parity with today's four free functions; `resolve()` throws `UsageError` for missing modules; `listGroups()` returns `[{name, description}]`.
  - **Depends on**: none

- [x] **Introduce `CliContext` class**
  - **Story**: US-10
  - **What**: Class built from parsed argv + the command module's options + `Secrets`. Expose `.secrets`, `.command`, `.subcommand`, `.positional(i)`, `.option(name)`. Keep `parseArgs` from `node:util` internal to the constructor.
  - **Where**: new `app/cli/cli-context.js`
  - **Acceptance criteria**: US-10 — `context.option('dry-run')` replaces `context.args.options['dry-run']`; `context.secrets` is a `Secrets` instance (not a plain object); tests for `deploy` pass unchanged structurally.
  - **Depends on**: Introduce `Secrets` class, Introduce `Argv` class

- [x] **Update `DeployCommand` to use `CliContext` accessors**
  - **Story**: US-10
  - **What**: Replace `context.args.positionals[0]`, `context.args.options['dry-run']`, and `context.secrets.FOO` inside `DeployCommand.run` with `context.positional(0)`, `context.option('dry-run')`, and `context.secrets.require([...])`.
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: US-1, US-10 — no string-indexed option access remains; missing-secret path routes through `Secrets.require`.
  - **Depends on**: Refactor deploy command into `DeployCommand` class, Introduce `CliContext` class

- [x] **Introduce `Cli` orchestrator class**
  - **Story**: US-12
  - **What**: Class composing `Argv`, `CommandRegistry`, `HelpRenderer`, `Secrets`, and `CliContext`. `async run()` implements the three help branches, the no-subcommand branch, and the dispatch-to-command branch currently in `main()`. Keep the top-level error handling (`UsageError` vs unknown) where it is.
  - **Where**: rewrite `hudson.js` so the file contains: imports, `class Cli`, and the `if (process.argv[1] === …)` bootstrap that calls `new Cli(process.argv, rootDir).run()`.
  - **Acceptance criteria**: US-7, US-8, US-9, US-12 — `hudson.js` contains no free `parseArgv`/`loadSecrets`/`resolveCommand`/`buildContext`/`renderHelp`/`loadCommandIndex`/`scanCommands`/`scanSubcommands` functions; CLI behavior (help, dispatch, error exit codes) unchanged; manual smoke-test of `node hudson.js app deploy --dry-run some/dir` still works.
  - **Depends on**: Introduce `Secrets` class, Introduce `Argv` class, Introduce `HelpRenderer` class, Introduce `CommandRegistry` class, Introduce `CliContext` class

- [x] **Introduce `Command` base class**
  - **Story**: US-11
  - **What**: Base class declaring `static description`, `static options`, and `async run(context)`. Not abstract-enforced (JS); document the contract in a short jsdoc. Intended as the uniform shape that `CommandRegistry.resolve` returns.
  - **Where**: new `app/cli/command.js`
  - **Acceptance criteria**: US-11 — class exists with jsdoc; default `run()` throws "not implemented".
  - **Depends on**: none

- [x] **Convert `commands/app/deploy.js` to extend `Command`**
  - **Story**: US-11
  - **What**: `DeployCommand extends Command`; move `description` and `options` to static class fields. Change the module default export to the class. Keep `export const description` / `export const options` as re-exports of the static fields until the registry switches to class-based resolution (next task).
  - **Where**: `commands/app/deploy.js`
  - **Acceptance criteria**: US-11 — class extends `Command`; static fields populated; module still resolves under the old registry contract.
  - **Depends on**: Update `DeployCommand` to use `CliContext` accessors, Introduce `Command` base class

- [x] **Convert `commands/test/ping.js` to extend `Command`**
  - **Story**: US-11
  - **What**: Mirror the deploy conversion — `PingCommand extends Command`, static `description`/`options`, class as default export, leave the named exports for registry compatibility.
  - **Where**: `commands/test/ping.js`
  - **Acceptance criteria**: US-11 — `node hudson.js test ping --message=hi` still prints the context JSON.
  - **Depends on**: Introduce `Command` base class

- [x] **Switch `CommandRegistry.resolve` to return `Command` classes and update `Cli`**
  - **Story**: US-11
  - **What**: `CommandRegistry.resolve` reads the module's default export (the class), instantiates it, and returns the instance. `Cli` calls `commandInstance.run(context)` instead of `mod.main(context)`. Remove the transitional named exports from `deploy.js` and `ping.js`. Update `HelpRenderer` call site to pull `description`/`options` from the class's static fields.
  - **Where**: `app/cli/command-registry.js`, `hudson.js` (the `Cli` class), `commands/app/deploy.js`, `commands/test/ping.js`
  - **Acceptance criteria**: US-7, US-11, US-12 — no command module exports a free `main`; `--help` for a subcommand still renders description and options; dispatch still succeeds end-to-end.
  - **Depends on**: Introduce `Cli` orchestrator class, Convert `commands/app/deploy.js` to extend `Command`, Convert `commands/test/ping.js` to extend `Command`
