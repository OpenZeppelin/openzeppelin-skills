---
name: codex-develop-secure-contracts
description: Develop or modify contracts with OpenZeppelin components across Solidity, Cairo, Stylus, and Stellar. Trigger for prompts like "add pausability", "make mint owner-only", "add permit", "integrate AccessControl/Ownable", "secure this contract", "use OpenZeppelin", "add ERC20/ERC721/ERC1155 features", "add governor/timelock", or any request to integrate OpenZeppelin into existing smart contract code.
---

# Develop Secure Contracts with OpenZeppelin (Codex Draft)

## Core Workflow

1. Classify the request.
- Explain concepts for conceptual questions.
- Implement code changes for implementation questions.

2. Read the project first.
- Search for relevant contract files (`.sol`, `.cairo`, `.rs`) before proposing changes.
- Read existing code and integrate features into current contracts by default.

3. Apply a library-first decision tree.
- Use an existing OpenZeppelin component directly if it matches.
- Extend official extension points if close but not exact.
- Write custom logic only if the library does not provide the feature.

4. Never copy OpenZeppelin source into user code.
- Import from dependencies so projects receive upstream updates.

## Pattern Discovery

1. Locate installed dependencies.
- Solidity: `node_modules/@openzeppelin/contracts/` or `lib/openzeppelin-contracts/`
- Cairo: Scarb dependencies resolved from `Scarb.toml`
- Stylus/Stellar: Cargo dependencies resolved from `Cargo.toml`

2. Read component source and examples.
- Identify required imports, inheritance/composition, storage, initializer calls, overrides, and guards.

3. Extract and apply the minimal diff.
- Add only the required changes.
- Avoid replacing full files unless explicitly requested.

## MCP Generators (Optional)

If generator tools are available:

1. Generate baseline (feature disabled).
2. Generate variant (feature enabled).
3. Diff baseline vs variant.
4. Apply only the discovered deltas to user code.

If a generator or parameter is missing, fall back to source-based pattern discovery and continue implementation.

## Quality Bar

- Resolve conflicts coherently (for example, duplicate or incompatible access systems).
- Preserve storage compatibility when working with upgradeable contracts.
- If a required file cannot be read, report the exact path and error.
