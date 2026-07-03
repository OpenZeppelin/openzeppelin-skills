---
name: develop-secure-contracts
description: "Develop secure smart contracts using OpenZeppelin Contracts libraries. Use when users need to integrate OpenZeppelin library components — including token standards (ERC20, ERC721, ERC1155), access control (Ownable, AccessControl, AccessManager), security primitives (Pausable, ReentrancyGuard), governance (Governor, timelocks), or accounts (multisig, account abstraction) — into existing or new contracts. Covers pattern discovery from library source, CLI contract generators, and library-first integration. Supports Solidity, Cairo, Stylus, Stellar, and Sui Move."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Develop Secure Smart Contracts with OpenZeppelin

## Core Workflow

### Understand the Request Before Responding

For conceptual questions ("How does Ownable work?"), explain without generating code. For implementation requests, proceed with the workflow below.

### CRITICAL: Always Read the Project First

Before generating code or suggesting changes:

1. **Search the user's project** for existing contracts (`Glob` for `**/*.sol`, `**/*.cairo`, `**/*.rs`, `**/*.move`, etc.)
2. **Read the relevant contract files** to understand what already exists
3. **Default to integration, not replacement** — when users say "add pausability" or "make it upgradeable", they mean modify their existing code, not generate something new. Only replace if explicitly requested ("start fresh", "replace this").

If a file cannot be read, surface the failure explicitly — report the path attempted and the reason. Ask whether the path is correct. Never silently fall back to a generic response as if the file does not exist.

### Fundamental Rule: Prefer Library Components Over Custom Code

Before writing ANY logic, search the OpenZeppelin library for an existing component:

1. **Exact match exists?** Import and use it directly — inherit, implement its trait, compose with it. Done.
2. **Close match exists?** Import and extend it — override only functions the library marks as overridable (virtual, hooks, configurable parameters).
3. **No match exists?** Only then write custom logic. Confirm by browsing the library's directory structure first.

**NEVER copy or embed library source code into the user's contract.** Always import from the dependency so the project receives security updates. Never hand-write what the library already provides:
- Never write a custom `paused` modifier when `Pausable` or `ERC20Pausable` exists
- Never write `require(msg.sender == owner)` when `Ownable` exists
- Never implement ERC165 logic when the library's base contracts already handle it

### Methodology

The primary workflow is **pattern discovery from library source code**:

1. Inspect what the user's project already imports
2. Read the dependency source and docs in the project's installed packages
3. Identify what functions, modifiers, hooks, and storage the dependency requires
4. Apply those requirements to the user's contract

See [Pattern Discovery and Integration](#pattern-discovery-and-integration) below for the full step-by-step procedure.

### CLI Generators as Reference

Use `npx @openzeppelin/contracts-cli` to generate reference implementations for pattern discovery:
generate a baseline to a file, generate with a feature enabled to another file, diff them, and apply the changes to the user's code. The CLI output is the canonical correct integration — use it as the source of truth for what imports, inheritance, storage, and overrides a feature requires.

See [CLI Generators](#cli-generators) for details on the generate-compare-apply workflow.

If no CLI command exists for what's needed, use the generic pattern discovery methodology from [Pattern Discovery and Integration](#pattern-discovery-and-integration). The absence of a CLI command does not mean the library lacks support — it only means there is no generator.

## Pattern Discovery and Integration

Procedural guide for discovering and applying OpenZeppelin contract integration patterns
by reading dependency source code. Works for any ecosystem and any library version.

**Prerequisite:** Always follow the library-first decision tree above
(prefer library components over custom code, never copy/embed source).

### Step 1: Identify Dependencies and Search the Library

1. Search the project for contract files: `Glob` for `**/*.sol`, `**/*.cairo`, `**/*.rs`,
   `**/*.move`, or the relevant extension from the lookup table below.
2. Read import/use statements in existing contracts to identify which OpenZeppelin components
   are already in use.
3. Locate the installed dependency in the project's dependency tree:
   - Solidity: `node_modules/@openzeppelin/contracts/` (Hardhat/npm) or
     `lib/openzeppelin-contracts/` (Foundry/forge)
   - Cairo: resolve from `Scarb.toml` dependencies — source cached by Scarb
   - Stylus: resolve from `Cargo.toml` — source in `target/` or the cargo registry cache
     (`~/.cargo/registry/src/`)
   - Stellar: resolve from `Cargo.toml` — same cargo cache locations as Stylus
   - Sui Move: resolve from `Move.toml` `[dependencies]` (`<move_package_name> = { r.mvr = "@openzeppelin-move/<slug>" }`);
     the Move Registry source is cached under `~/.move/` after a build
     (`sui move build --build-env <env>` — the build env is required whenever there are MVR deps), and mirrored per-dependency in the project's
     `build/<project_package>/sources/dependencies/<move_package_name>/` — read `.move` sources there for exact
     signatures. Generate readable code docs with `sui move build --doc --build-env <env>`; dependency
     docs land under `build/<project_package>/docs/dependencies/<move_package_name>/`.
4. Browse the dependency's directory listing to discover available components. Use `Glob`
   patterns against the installed source (e.g., `node_modules/@openzeppelin/contracts/**/*.sol`).
   Do not assume knowledge of the library's contents — always verify by listing directories.
5. If the dependency is not installed locally, clone or browse the canonical repository
   (see lookup table below).

### Step 2: Read the Dependency Source and Documentation

1. Read the source file of the component relevant to the user's request.
2. Look for documentation within the source: NatSpec comments (`///`, `/** */`) in Solidity,
   doc comments (`///`) in Rust and Cairo, and README files in the component's directory.
3. Determine the integration strategy using the decision tree from the Critical Principle:
   - If the component satisfies the need directly → import and use as-is.
   - If customization is needed → identify extension points the library provides (virtual
     functions, hook functions, configurable constructor parameters). Import and extend.
   - Only if no component covers the need → write custom logic.
4. Identify the **public API**: functions/methods exposed, events emitted, errors defined.
5. Identify **integration requirements** — this is the critical step:
   - Functions the integrator MUST implement (abstract functions, trait methods, hooks)
   - Modifiers, decorators, or guards that must be applied to the integrator's functions
   - Constructor or initializer parameters that must be passed
   - Storage variables or state that must be declared
   - Inheritance or trait implementations required (always via import, never via copy)
6. Search for example contracts or tests in the same repository that demonstrate correct
   usage. Look in `test/`, `tests/`, `examples/`, or `mocks/` directories.

### Step 3: Extract the Minimal Integration Pattern

From Step 2, construct the minimal set of changes needed:

- **Imports / use statements** to add
- **Inheritance / trait implementations** to add (always via import from the dependency)
- **Storage** to declare
- **Constructor / initializer** changes (new parameters, initialization calls)
- **New functions** to add (required overrides, hooks, public API)
- **Existing functions to modify** (add modifiers, call hooks, emit events)

If the contract is upgradeable, any of the above may affect storage compatibility. Consult the relevant upgrade skill before applying.

Do not include anything beyond what the dependency requires. This is the minimal diff
between "contract without the feature" and "contract with the feature."

### Step 4: Apply Patterns to the User's Contract

1. Read the user's existing contract file.
2. Apply the changes from Step 3 using the `Edit` tool. Do not replace the entire file —
   integrate into existing code.
3. Check for conflicts: duplicate access control systems, conflicting function overrides,
   incompatible inheritance. Resolve before finishing.
4. Do not ask the user to make changes themselves — apply directly.

### Repository and Documentation Lookup Table

| Ecosystem | Repository | Documentation | File Extension | Dependency Location |
|-----------|-----------|---------------|----------------|-------------------|
| Solidity | [openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) | [docs.openzeppelin.com/contracts](https://docs.openzeppelin.com/contracts) | `.sol` | `node_modules/@openzeppelin/contracts/` or `lib/openzeppelin-contracts/` |
| Cairo | [cairo-contracts](https://github.com/OpenZeppelin/cairo-contracts) | [docs.openzeppelin.com/contracts-cairo](https://docs.openzeppelin.com/contracts-cairo) | `.cairo` | Scarb cache (resolve from `Scarb.toml`) |
| Stylus | [rust-contracts-stylus](https://github.com/OpenZeppelin/rust-contracts-stylus) | [docs.openzeppelin.com/contracts-stylus](https://docs.openzeppelin.com/contracts-stylus) | `.rs` | Cargo cache (`~/.cargo/registry/src/`) |
| Stellar | [stellar-contracts](https://github.com/OpenZeppelin/stellar-contracts) ([Architecture](https://github.com/OpenZeppelin/stellar-contracts/blob/main/Architecture.md)) | [docs.openzeppelin.com/stellar-contracts](https://docs.openzeppelin.com/stellar-contracts) | `.rs` | Cargo cache (`~/.cargo/registry/src/`) |
| Sui Move | [contracts-sui](https://github.com/OpenZeppelin/contracts-sui) ([llms.txt](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/llms.txt) · [ARCHITECTURE](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/ARCHITECTURE.md)) | [docs.openzeppelin.com/contracts-sui](https://docs.openzeppelin.com/contracts-sui) | `.move` | Move Registry cache (`~/.move/`, resolve from `Move.toml`) |

### Directory Structure Conventions

Where to find components within each repository:

| Category | Solidity | Cairo | Stylus | Stellar |
|----------|---------|-------|--------|---------|
| Tokens | `contracts/token/{ERC20,ERC721,ERC1155}/` | `packages/token/` | `contracts/src/token/` | `packages/tokens/` |
| Access control | `contracts/access/` | `packages/access/` | `contracts/src/access/` | `packages/access/` |
| Governance | `contracts/governance/` | `packages/governance/` | — | `packages/governance/` |
| Proxies / Upgrades | `contracts/proxy/` | `packages/upgrades/` | `contracts/src/proxy/` | `packages/contract-utils/` |
| Utilities / Security | `contracts/utils/` | `packages/utils/`, `packages/security/` | `contracts/src/utils/` | `packages/contract-utils/` |
| Accounts | `contracts/account/` | `packages/account/` | — | `packages/accounts/` |

Browse these paths first when searching for a component.

**Sui Move** deliberately isn't in this fixed grid: its package set doesn't map cleanly onto these categories and grows over time, so don't work from a hardcoded list. Discover it from the library's own metadata instead — start at [`llms.txt`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/llms.txt), follow it to whatever package catalogs it links, then read each package's `README.md` (module list) and `examples/`. A capability is often one module among several inside a package, so read the package README rather than assuming one package equals one component. (The `setup-sui-contracts` skill covers this discovery flow in full.)

### Known Version-Specific Considerations

Do not assume override points from prior knowledge — always verify by reading the installed source. Functions that were `virtual` in an older version may no longer be in the current one, making them non-overridable. The source NatSpec will indicate the correct override point (e.g., `NOTE: This function is not virtual, {X} should be overridden instead`).

A known example: the Solidity ERC-20 transfer hook changed between v4 and v5. Read the installed `ERC20.sol` to confirm which function is `virtual` before recommending an override.

### Sui Move Integration Notes

Sui has **no `@openzeppelin/contracts-cli` generator**, so there is no generate-compare-apply shortcut — always use the pattern-discovery methodology above, treating each package's `examples/` as the canonical, compilable integration recipe (adapt one rather than writing from scratch).

Don't restate Sui/Move conventions here — read them from the library's own sources of truth (all linked from [`llms.txt`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/llms.txt)):

- **How the library is shaped and composed** — capability-based access (witnesses/OTW), owned vs. shared objects, PTB-friendliness, initialization: [`ARCHITECTURE.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/ARCHITECTURE.md) + the package `examples/`.
- **Move 2024 idioms** — receiver/method syntax (`public use fun`), module layout, naming: [`STYLEGUIDE.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/STYLEGUIDE.md).
- **The exact `r.mvr` dependency snippet and `use` path** (the Move package name differs from the MVR slug): the package's own `README.md`.
- **Exact API — signatures, parameters, events, abort conditions**: the documentation site [docs.openzeppelin.com/contracts-sui](https://docs.openzeppelin.com/contracts-sui) (concepts/guides, plus the generated API reference under `.../<major>.x/api/<package>`); the installed source and its doc-comments are the ground truth when the docs are terse.
- **Toolchain, `Move.toml` (including resolving version conflicts when you combine OZ packages, e.g. an `override` on a shared math dependency), `--build-env` builds, and testing conventions**: the `setup-sui-contracts` skill.

As in every ecosystem, integrate by importing via MVR — never copy library source into the project.

**Before finishing, run the project's full quality gate** — build, test, lint, and formatting — using the commands and formatter the `setup-sui-contracts` skill already specifies.

## CLI Generators

The `@openzeppelin/contracts-cli` package generates reference OpenZeppelin contract implementations from the command line. Use it as the reference source in the generate-compare-apply workflow whenever a command exists for the contract type.

### Discovering Commands and Options

Run `npx @openzeppelin/contracts-cli --help` to list available commands. Each command corresponds to a contract type (e.g., `solidity-erc20`, `cairo-erc721`, `stellar-fungible`). Run `npx @openzeppelin/contracts-cli <command> --help` to see the available options. Do not rely on prior knowledge of what options exist; check `--help` at the start of a conversation since the CLI may have been updated.

### Generate-Compare-Apply Shortcut

When a CLI command exists for the contract type, pipe generated output to temporary files and diff them to keep generated contract code out of the conversation context:

1. **Generate baseline** — run with only required options, all features disabled, pipe to a file:
   ```bash
   npx @openzeppelin/contracts-cli solidity-erc20 --name MyToken --symbol MTK > /tmp/oz-baseline.sol
   ```
2. **Generate with feature** — run again with the feature enabled, pipe to a second file:
   ```bash
   npx @openzeppelin/contracts-cli solidity-erc20 --name MyToken --symbol MTK --pausable > /tmp/oz-variant.sol
   ```
3. **Compare** — diff the two files to identify exactly what changed (imports, inheritance, state, constructor, functions, modifiers):
   ```bash
   diff /tmp/oz-baseline.sol /tmp/oz-variant.sol
   ```
4. **Apply** — edit the user's existing contract to add the discovered changes

For interacting features (e.g., access control + upgradeability), generate a combined variant as well.

### When No CLI Command Exists or a Feature Is Not Covered

The absence of a CLI command does NOT mean the library lacks support. It only means there is no generator for that contract type. Always fall back to the generic pattern discovery methodology in [Pattern Discovery and Integration](#pattern-discovery-and-integration).

Similarly, when a CLI command exists but does not expose an option for a specific feature, do not stop there. Fall back to pattern discovery for that feature: read the installed library source to find the relevant component, extract the integration requirements, and apply them to the user's contract.
