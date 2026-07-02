---
name: setup-sui-contracts
description: "Set up a Sui Move smart contract project with OpenZeppelin Contracts for Sui. Use when users need to: (1) install the Sui CLI and Move toolchain, (2) create a new Sui Move package, (3) discover and add OpenZeppelin Sui dependencies to Move.toml via the Move Registry (MVR), (4) learn the integration pattern from the library's examples and API docs before implementing, or (5) understand Sui Move import conventions and build/test commands for OpenZeppelin."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Sui Setup

## Prerequisites

Install the Sui CLI by following the [Sui installation guide](https://docs.sui.io/getting-started/onboarding/sui-install). The CLI bundles the Move toolchain and the Move Registry (MVR) resolver, so no separate Move install is needed.

Any time you need to investigate the library, start from its AI discovery entry point, [`llms.txt`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/llms.txt) — it maps the library's content (the architecture doc, the `contracts/` and `math/` package catalogs, each package's README/examples/API reference, and audits). Follow its links rather than guessing paths.

OpenZeppelin Contracts for Sui pins a specific Sui CLI version in its top-level [README](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/README.md) (this is repo metadata, not linked from `llms.txt`) — read the required version there and check your local install against it. The pinned version is the tested one and safest to match; a newer patch/minor generally works too, so treat a small drift as a warning, not a blocker:

```bash
sui --version
```

## Create a Project

Initialize a new Move package (only if starting a new project):

```bash
sui move new my_project
cd my_project
```

This creates `Move.toml`, a `.gitignore`, and commented-out stub modules `sources/my_project.move` and `tests/my_project_tests.move`, using Move `edition = "2024"`. The stubs are inert placeholders (fully wrapped in `/* … */`); replace or delete them when you add your own modules.

## OpenZeppelin Dependencies

OpenZeppelin packages are published to the **Move Registry (MVR)** and added to `Move.toml` under `[dependencies]` with the `r.mvr` format, mapping the Move package name to its MVR slug:

```toml
[dependencies]
<move_package_name> = { r.mvr = "@openzeppelin-move/<slug>" }
```

Do **not** rely on a memorized package list — it drifts as the library evolves. Discover what is available from the library's own metadata, which is the single source of truth:

1. Start — as always — at the AI discovery entry point, [`llms.txt`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/llms.txt).
2. Follow its links to **every package catalog it lists** — `llms.txt` is the authority on which catalogs exist, so read the set from there rather than assuming a fixed one (new top-level catalogs get added over time). Each catalog table lists the MVR slug, the Move package name, docs, and highlights.
3. Read the individual package's `README.md` for the exact `r.mvr` install snippet and its module list. Confirm the slug resolves — either by looking it up on [moveregistry.com](https://www.moveregistry.com) or, definitively, by running the build (below), which resolves every slug against the MVR.

> **Catalog paths are relative to the catalog file's own directory.** A catalog at `<catalog-dir>/README.md` lists each package by a `Path` relative to `<catalog-dir>/`, so the package's raw README is `.../main/<catalog-dir>/<Path>/README.md` — resolve `<Path>` against the directory of the catalog that links it, not the repo root (the bare `.../main/<Path>/README.md` 404s). When in doubt, list the tree: `gh api repos/OpenZeppelin/contracts-sui/git/trees/main?recursive=1`.

> The Move package name (used in `use` statements) differs from the MVR slug — slug `@openzeppelin-move/integer-math` is Move package `openzeppelin_math`. Only add the packages the project actually uses.

> **A package usually contains several modules.** The catalog lists *packages*; the building block you want is often one module among several inside a package. Read the package README's module list and its `examples/` to see what a package actually exposes — don't assume one package equals one component, and don't rely on a fixed mental catalog: the set of packages and modules grows over time.

### Resolving version conflicts (diamond dependencies)

Two OZ packages can pull *different revisions* of a shared transitive dependency — for example, if one package embeds its own revision of a math package that you also depend on directly. When that happens the build aborts:

```
Package depends on multiple versions of the package with ID 0x…
```

Resolve it by adding `override = true` to the directly-declared dependency, which forces every consumer onto that revision:

```toml
[dependencies]
openzeppelin_math = { r.mvr = "@openzeppelin-move/integer-math", override = true }
openzeppelin_fp_math = { r.mvr = "@openzeppelin-move/fixed-point-math" }
```

## Study the Examples and Docs Before Implementing

Do this before writing any integration code — most users don't know the full catalog, and it grows over time, so let the library show you what it provides and how it's meant to be composed. For each package you plan to use:

1. **Study its `examples/`.** Every package ships compilable, CI-tested integration examples — these are the canonical composition recipes and the fastest way to see the real usage pattern (initialization, capability/witness flow, object ownership, tests). Prefer copying and adapting an example into `sources/` over writing from scratch.
2. **Read the docs.** Start at the [documentation site](https://docs.openzeppelin.com/contracts-sui) for concepts and guides, then the generated API reference — reached via the `Docs` link in each catalog table (it points at the correct version), path `.../contracts-sui/<major>.x/api/<package>` where `<major>` matches your contracts-sui release and `<package>` is the short catalog name, not the Move package name — for exact signatures, parameters, and abort conditions.
3. **Read the code docs.** The API reference is generated from the modules' doc-comments, so the installed source is the ground truth. Generate the same docs locally with `sui move build --doc --build-env testnet` (the `--build-env` is required whenever there are MVR deps) — this also emits the docs for every OZ dependency under `build/<package>/docs/dependencies/<move_package_name>/` — or just read the doc-comments in the installed source.

Discover the available examples from `llms.txt` and each package README, then browse them in the [contracts-sui repo](https://github.com/OpenZeppelin/contracts-sui).

## Import Conventions

Import modules through the Move package name (the left side of the `Move.toml` entry) followed by the module name — `use <move_package_name>::<module>;` — then follow the usage patterns from the package's `examples/` and API docs.

> Some operations are exposed as **method syntax** (`value.op(...)`) via `public use fun` re-exports, so the underlying function may live in a different module than the type. If a free function you expect isn't found, check the package source for the `public use fun` lines and call it as a method.

## Generate Agent Guidance

Create an `AGENTS.md` at the project root so any agent that later opens this project knows it is built on OpenZeppelin Contracts for Sui and where the sources of truth live. Follow the same philosophy as the [contracts-sui `AGENTS.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/AGENTS.md): a lean pointer file that does **not** restate rules — it links to the authoritative sources. Also add a `CLAUDE.md` containing only `@AGENTS.md` so Claude Code picks it up (mirroring how contracts-sui itself is set up).

```markdown
# AGENTS.md

This is a Sui Move project built on **OpenZeppelin Contracts for Sui**.

## Sources of truth — read these first
- AI discovery entry point: https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/llms.txt
  (points to the package catalogs, each package's `examples/`, the generated API reference, and audits)
- Docs: https://docs.openzeppelin.com/contracts-sui
- API reference: https://docs.openzeppelin.com/contracts-sui/<major>.x/api/<package> (`<major>` matches your contracts-sui release; the catalog's `Docs` links carry the correct version)
- Code docs (local): `sui move build --doc --build-env testnet` → `build/<package>/docs/` (includes OZ dependencies)

## Conventions
- Before implementing, study each package's `examples/` (compilable, CI-tested composition recipes) and its API docs; prefer audited library components over custom logic.
- Import modules via the Move package name (`use <package>::<module>;`); a package typically exposes several modules — check its README and `examples/`.
- MVR-backed builds require a build env: `sui move build --build-env testnet`. If two OZ deps pull different revisions of a shared transitive package, add `override = true` to the directly-declared one.
- For fungible-asset custody, consider Sui [Address Balances](https://docs.sui.io/onchain-finance/asset-custody/address-balances/) (SIP-58) over summing `Coin` objects where it fits.
- Code style follows the OpenZeppelin Sui conventions — see Code Quality below.

## Code quality
The library's conventions are the single source of truth for Move style:
- Rules: https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/STYLEGUIDE.md
- Design rationale: https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/ARCHITECTURE.md
- Review procedure (reads STYLEGUIDE.md, reports/fixes violations): https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/.claude/commands/code-quality.md

## MCP (optional)
An OpenZeppelin MCP endpoint (https://mcp.openzeppelin.com) can be added for in-editor access to OZ knowledge.
```

## Build & Test

Because dependencies resolve through MVR, builds and tests must specify a build environment. Use `testnet` for development (`mainnet` is also available for release builds):

```bash
sui move build --build-env testnet
sui move test --build-env testnet
```

> Without `--build-env`, the CLI cannot resolve `r.mvr` dependencies and errors with "Could not determine the correct dependencies to use for `local`". A green `build` + `test` on a freshly scaffolded project is the acceptance bar.

Add `--lint --warnings-are-errors` once the project has real code to catch style and dead-code issues early:

```bash
sui move test --build-env testnet --lint --warnings-are-errors
```

### Testing patterns to know

A couple of Sui conventions recur when testing code built on these packages — the library's own `examples/` tests are the canonical reference:

- **One-Time Witness (OTW) in tests.** Modules that take an OTW consume it in `init`, and it can't be constructed at runtime. To exercise such a module in a unit test, expose a `#[test_only]` initializer that builds the OTW (its struct *is* constructible inside a `#[test_only]` context):

  ```move
  #[test_only]
  public fun init_for_testing(ctx: &mut TxContext) { init(MY_OTW {}, ctx) }
  ```

- **Clock.** Modules that depend on time take a `sui::clock::Clock`. In tests use `create_for_testing`, `increment_for_testing`, and `destroy_for_testing`.

- **Assertions.** Under `--warnings-are-errors`, a literal abort code (`assert!(cond, 0)`) trips the `abort_without_constant` lint. In tests, use `assert_eq!` (from `std::unit_test`) or a bare `assert!(cond)`; in non-test code, assert against a named error constant rather than a literal.

> **Composition constraint:** a Sui `init` function receives no `Clock` (nor other runtime arguments). A component whose constructor needs a `Clock` therefore can't be fully built in `init` — create/share the object in `init`, then complete setup in a later entry function that takes the `Clock`. The package's `examples/` show the intended two-step pattern.

## Code Quality

Align the project's Move style with OpenZeppelin's own conventions so integrator code stays consistent with the library. As with dependencies, don't restate the rules — point at the library's single sources of truth:

- **[`STYLEGUIDE.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/STYLEGUIDE.md)** — the codified rules (Naming, Module & Package, Section ordering, Imports, Structs, Functions, Idiomatic Move 2024, Collections & object size, Testing, Lint suppression, Documentation).
- **[`ARCHITECTURE.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/ARCHITECTURE.md)** — the design rationale those rules reference (capability-based access, owned vs. shared objects, PTB composability, bounded state, upgrade safety).
- **[`.claude/commands/code-quality.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/.claude/commands/code-quality.md)** — the review procedure: it reads `STYLEGUIDE.md` and checks each `.move` file against it, then reports or fixes violations. The procedure is plain markdown, so any agent can follow it even though the slash command is Claude-specific.

Record these under a **Code quality** heading in the generated `AGENTS.md` (see above) so downstream agents follow the same conventions on clone.

### Formatting

contracts-sui formats Move with the [`@mysten/prettier-plugin-move`](https://github.com/MystenLabs/sui/blob/main/external-crates/move/tooling/prettier-move/README.md) Prettier plugin — match it so integrator code lays out identically. Install it and add a `.prettierrc` matching the library's:

```bash
npm i -D prettier @mysten/prettier-plugin-move
```

```json
{
  "tabWidth": 4,
  "printWidth": 100,
  "useModuleLabel": true,
  "autoGroupImports": "module",
  "plugins": ["@mysten/prettier-plugin-move"]
}
```

Then format from the repo root (Prettier silently skips `.move` files if the plugin isn't resolvable, so keep it in `devDependencies`):

```bash
npx prettier --write "**/*.move"
```

> `autoGroupImports: "module"` normalizes `use` grouping, so run `--write` to let it format — don't hand-tune imports and expect `--check` to pass first.

The `code-quality.md` review procedure runs this same formatting pass after applying fixes.
