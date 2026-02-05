---
name: openzeppelin-contracts-patterns
description: Use when writing, modifying, or reviewing smart contracts that use OpenZeppelin Contracts. Triggers include: (1) implementing token standards (ERC20, ERC721, ERC1155, fungible, non-fungible), (2) adding security features (access control, pausability), (3) governance contracts (Governor, voting, timelocks), (4) smart accounts (account abstraction), (5) upgradeability patterns, (6) questions about OpenZeppelin component integration. Supports Solidity/EVM, Cairo/Starknet, Stylus, Stellar, and Uniswap Hooks.
---

# OpenZeppelin Contracts Patterns

## CRITICAL: Always Read the Project First

**BEFORE calling any MCP generator tool**, you MUST:

1. **Search the user's project** for existing smart contracts (e.g., `Glob` for `**/*.sol`, `**/*.cairo`, etc.)
2. **Read the relevant contract files** to understand what already exists
3. **Only then** use the MCP generators as a discovery tool for patterns

Never assume the user wants a new contract. When they say "my account needs multisig" or "add pausability to my token", they are asking you to modify their existing code, not generate something from scratch.

---

Use the MCP smart contract generators as a **discovery tool** to learn OpenZeppelin Contracts patterns, then **apply those patterns to the user's existing contract**.

**Important**: Do NOT simply return generated code to the user. The generators produce reference implementations that show what imports, inheritance, state variables, constructor changes, functions, and modifiers a feature requires. Your job is to extract those patterns and apply them to the user's actual contract files.

**Default to integration, not replacement**: When the user asks to modify an existing contract (e.g., "make my token use AccessManaged"), integrate the pattern into their existing code—don't ask whether to replace or convert. Generated contracts are reference implementations; the user's contract contains their business logic. Only replace an existing contract if the user explicitly requests it (e.g., "start fresh", "replace this with a basic ERC20").

## Workflow Summary

1. **Search and read** the user's existing contracts to understand their current structure
2. **Generate** reference implementations using MCP tools to discover patterns
3. **Compare** baseline vs. feature-enabled outputs to identify what changes
4. **Edit** the user's contract file directly, applying the discovered patterns

## Prerequisites

Verify the required MCP server is available. Server names and tools by ecosystem:

| Ecosystem | Server Name | Tools (non-exhaustive) |
|-----------|-------------|------------------------|
| Solidity/EVM | `OpenZeppelinSolidityContracts` or `OpenZeppelinContracts` | `solidity-erc20`, `solidity-erc721`, `solidity-erc1155`, `solidity-governor`, `solidity-account`, `solidity-custom` |
| Cairo/Starknet | `OpenZeppelinCairoContracts` or `OpenZeppelinContracts` | `cairo-erc20`, `cairo-erc721`, `cairo-account`, `cairo-governor`, `cairo-custom` |
| Stylus | `OpenZeppelinStylusContracts` or `OpenZeppelinContracts` | `stylus-erc20`, `stylus-erc721`, `stylus-erc1155` |
| Stellar | `OpenZeppelinStellarContracts` or `OpenZeppelinContracts` | `stellar-fungible`, `stellar-stablecoin`, `stellar-non-fungible` |
| Uniswap | `OpenZeppelinUniswapHooks` or `OpenZeppelinContracts` | `uniswap-hooks` |

If unavailable, direct user to https://mcp.openzeppelin.com/ for installation.

## Discovery Loop

Do not assume knowledge of what code each feature adds to a contract. Generate and compare to learn the actual patterns.

### Step 1: Select Tool

Match the user's ecosystem and contract type to the appropriate generator tool from the Prerequisites table.

### Step 2: Generate Baseline

Call the generator with only the required parameters (name, symbol, etc.) and all optional features disabled or at defaults. Keep this generated code as the baseline reference.

### Step 3: Generate Variants

For each feature the user needs:
1. Call the generator again with the same base parameters
2. Enable only that one feature
3. Keep each generated variant

If features might interact (e.g., access control + upgradeability), generate a combined variant as well.

### Step 4: Compare Baseline to Variants

For each variant, compare line-by-line against the baseline. Identify exactly what the feature added or changed:

- **Imports**: New dependencies added
- **Inheritance**: New base contracts/traits
- **State variables**: New storage fields
- **Constructor/initializer**: New parameters or initialization logic
- **New functions**: Functions that didn't exist in baseline
- **Modified functions**: Functions that exist in both but have different implementations (look for modifiers, hooks, guards, or overrides)

Record these observations—they are the pattern for that feature.

### Step 5: Apply Patterns to User's Contract

**Read the user's existing contract file first**, then apply the observed differences from Step 4 by editing their file:

1. Add the same imports the feature required
2. Add the same inheritance
3. Add any new state variables
4. Add or modify constructor/initializer logic
5. Add the new functions exactly as observed
6. Modify relevant existing functions to include the same modifiers, hooks, guards, or override logic

**Do not ask the user to make these changes themselves**—use the Edit tool to apply them directly to their contract file.

### Answering Best Practices Questions

When users ask about best practices, run the discovery loop for relevant features, then:
- Show the concrete patterns observed from the generated code
- Explain which generator options produced which changes
- Recommend based on what you observed, not assumptions
