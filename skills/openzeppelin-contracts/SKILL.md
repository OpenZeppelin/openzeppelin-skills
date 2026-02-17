---
name: openzeppelin-contracts
description: "Smart contract development with OpenZeppelin Contracts libraries. Use when users need to: (1) implement token standards (ERC20, ERC721, ERC1155, fungible/non-fungible), (2) add access control (Ownable, Roles, AccessManager), (3) add security features (Pausable, ReentrancyGuard), (4) implement governance (Governor, voting, timelocks), (5) make contracts upgradeable (UUPS, Transparent, Beacon proxies), (6) build accounts (multisig, abstraction), (7) set up a project with OpenZeppelin dependencies, or (8) apply any OpenZeppelin library pattern. Supports Solidity, Cairo, Stylus, and Stellar ecosystems."
---

# OpenZeppelin Contracts

## Setup

Read per-ecosystem setup when users need to install dependencies, create a project, or configure tooling.

- [references/setup-solidity.md](references/setup-solidity.md) — Solidity (Hardhat, Foundry)
- [references/setup-cairo.md](references/setup-cairo.md) — Cairo (Scarb, Starknet)
- [references/setup-stylus.md](references/setup-stylus.md) — Stylus (Rust, Arbitrum)
- [references/setup-stellar.md](references/setup-stellar.md) — Stellar (Soroban)

## Core Workflow

### CRITICAL: Always Read the Project First

Before generating code or suggesting changes:

1. **Search the user's project** for existing contracts (`Glob` for `**/*.sol`, `**/*.cairo`, `**/*.rs`, etc.)
2. **Read the relevant contract files** to understand what already exists
3. **Default to integration, not replacement** — when users say "add pausability" or "make it upgradeable", they mean modify their existing code, not generate something new. Only replace if explicitly requested ("start fresh", "replace this").

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

Read [references/patterns.md](references/patterns.md) for the full step-by-step procedure.

### MCP Generators as an Optional Shortcut

If MCP generator tools are available at runtime, use them to accelerate pattern discovery:
generate a baseline, generate with a feature enabled, compare the diff, and apply the changes to the user's code. This replaces the manual source-reading step but follows the same principle — discover patterns, then integrate them.

See [MCP Generators (Optional)](#mcp-generators-optional) for details on checking availability and using the generate-compare-apply shortcut.

If no MCP tool exists for what's needed, use the generic pattern discovery methodology from [references/patterns.md](references/patterns.md). The absence of an MCP tool does not mean the library lacks support — it only means there is no generator.

## Upgrades

Read when users need proxy patterns (UUPS, Transparent, Beacon), initializers, or upgrade procedures.

- [references/upgrades-solidity.md](references/upgrades-solidity.md) — Solidity upgrades (Hardhat, Foundry)
- [references/upgrades-cairo.md](references/upgrades-cairo.md) — Cairo upgrades
- [references/upgrades-stylus.md](references/upgrades-stylus.md) — Stylus upgrades (Rust, Arbitrum)
- [references/upgrades-stellar.md](references/upgrades-stellar.md) — Stellar upgrades (Soroban)

## MCP Generators (Optional)

MCP generators are template/scaffolding tools that produce OpenZeppelin contract boilerplate. They are **not required** — they accelerate pattern discovery when available.

### Checking Availability

Discover MCP tools dynamically at runtime. Look for tools with names matching patterns like `solidity-erc20`, `cairo-erc721`, `stellar-fungible`, etc. Server names follow patterns like `OpenZeppelinSolidityContracts`, `OpenZeppelinCairoContracts`, or `OpenZeppelinContracts`.

MCP tool schemas are self-describing. To learn what a generator supports, inspect its parameter list — each boolean parameter (e.g., `pausable`, `mintable`, `upgradeable`) corresponds to a feature toggle. Do not rely on prior knowledge of what parameters exist; read the schema each time, since tools are updated independently of this skill.

### Generate-Compare-Apply Shortcut

When an MCP generator exists for the contract type:

1. **Generate baseline** — call with only required parameters, all features disabled
2. **Generate with feature** — call again with one feature enabled
3. **Compare** — diff baseline vs. variant to identify exactly what changed (imports, inheritance, state, constructor, functions, modifiers)
4. **Apply** — edit the user's existing contract to add the discovered changes

For interacting features (e.g., access control + upgradeability), generate a combined variant as well.

### When No MCP Tool Exists

The absence of an MCP tool does NOT mean the library lacks support. It only means there is no generator for that contract type. Always fall back to the generic pattern discovery methodology in [references/patterns.md](references/patterns.md).
