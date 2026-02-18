# Pattern Discovery and Integration

Procedural guide for discovering and applying OpenZeppelin contract integration patterns
by reading dependency source code. Works for any ecosystem and any library version.

**Prerequisite:** Always follow the library-first decision tree from the main skill instructions
(prefer library components over custom code, never copy/embed source).

## Contents

- [Step 1: Identify Dependencies and Search the Library](#step-1-identify-dependencies-and-search-the-library)
- [Step 2: Read the Dependency Source and Documentation](#step-2-read-the-dependency-source-and-documentation)
- [Step 3: Extract the Minimal Integration Pattern](#step-3-extract-the-minimal-integration-pattern)
- [Step 4: Apply Patterns to the User's Contract](#step-4-apply-patterns-to-the-users-contract)
- [Repository and Documentation Lookup Table](#repository-and-documentation-lookup-table)
- [When to Search the Web](#when-to-search-the-web)

## Step 1: Identify Dependencies and Search the Library

1. Search the project for contract files: `Glob` for `**/*.sol`, `**/*.cairo`, `**/*.rs`,
   or the relevant extension from the lookup table below.
2. Read import/use statements in existing contracts to identify which OpenZeppelin components
   are already in use.
3. Locate the installed dependency in the project's dependency tree:
   - Solidity: `node_modules/@openzeppelin/contracts/` (Hardhat/npm) or
     `lib/openzeppelin-contracts/` (Foundry/forge)
   - Cairo: resolve from `Scarb.toml` dependencies — source cached by Scarb
   - Stylus: resolve from `Cargo.toml` — source in `target/` or the cargo registry cache
     (`~/.cargo/registry/src/`)
   - Stellar: resolve from `Cargo.toml` — same cargo cache locations as Stylus
4. Browse the dependency's directory listing to discover available components. Use `Glob`
   patterns against the installed source (e.g., `node_modules/@openzeppelin/contracts/**/*.sol`).
   Do not assume knowledge of the library's contents — always verify by listing directories.
5. If the dependency is not installed locally, clone or browse the canonical repository
   (see lookup table below).

## Step 2: Read the Dependency Source and Documentation

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

## Step 3: Extract the Minimal Integration Pattern

From Step 2, construct the minimal set of changes needed:

- **Imports / use statements** to add
- **Inheritance / trait implementations** to add (always via import from the dependency)
- **Storage** to declare
- **Constructor / initializer** changes (new parameters, initialization calls)
- **New functions** to add (required overrides, hooks, public API)
- **Existing functions to modify** (add modifiers, call hooks, emit events)

Do not include anything beyond what the dependency requires. This is the minimal diff
between "contract without the feature" and "contract with the feature."

## Step 4: Apply Patterns to the User's Contract

1. Read the user's existing contract file.
2. Apply the changes from Step 3 using the `Edit` tool. Do not replace the entire file —
   integrate into existing code.
3. Check for conflicts: duplicate access control systems, conflicting function overrides,
   incompatible inheritance. Resolve before finishing.
4. Do not ask the user to make changes themselves — apply directly.

## Repository and Documentation Lookup Table

| Ecosystem | Repository | Documentation | File Extension | Dependency Location |
|-----------|-----------|---------------|----------------|-------------------|
| Solidity | [openzeppelin-contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) | [docs.openzeppelin.com/contracts](https://docs.openzeppelin.com/contracts) | `.sol` | `node_modules/@openzeppelin/contracts/` or `lib/openzeppelin-contracts/` |
| Cairo | [cairo-contracts](https://github.com/OpenZeppelin/cairo-contracts) | [docs.openzeppelin.com/contracts-cairo](https://docs.openzeppelin.com/contracts-cairo) | `.cairo` | Scarb cache (resolve from `Scarb.toml`) |
| Stylus | [rust-contracts-stylus](https://github.com/OpenZeppelin/rust-contracts-stylus) | [docs.openzeppelin.com/contracts-stylus](https://docs.openzeppelin.com/contracts-stylus) | `.rs` | Cargo cache (`~/.cargo/registry/src/`) |
| Stellar | [stellar-contracts](https://github.com/OpenZeppelin/stellar-contracts) | [docs.openzeppelin.com/stellar-contracts](https://docs.openzeppelin.com/stellar-contracts) | `.rs` | Cargo cache (`~/.cargo/registry/src/`) |

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

## Known Version-Specific Gotchas

Do not assume override points from prior knowledge — always verify by reading the installed source. Functions that were `virtual` in an older version may no longer be in the current one, making them non-overridable. The source NatSpec will indicate the correct override point (e.g., `NOTE: This function is not virtual, {X} should be overridden instead`).

A known example: the Solidity ERC-20 transfer hook changed between v4 and v5. Read the installed `ERC20.sol` to confirm which function is `virtual` before recommending an override.

## When to Search the Web

Use web search (`WebSearch`, `WebFetch`) only when:

- The dependency source is not installed locally and the repository cannot be accessed
  via `Bash` (e.g., `gh api` or `git clone`)
- Documentation in the source is insufficient for an undocumented or newly added feature
- The user's request involves third-party integrations beyond OpenZeppelin

Effective search patterns:
- `"OpenZeppelin" "{component name}" "{ecosystem}" example`
- `"OpenZeppelin" "{component name}" integration guide`
- `site:docs.openzeppelin.com {component name}`
