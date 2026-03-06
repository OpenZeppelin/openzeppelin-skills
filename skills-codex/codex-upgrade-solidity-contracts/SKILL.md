---
name: codex-upgrade-solidity-contracts
description: Upgrade Solidity contracts with OpenZeppelin proxy patterns. Use when users need UUPS/Transparent/Beacon workflows, initializer migration, Hardhat or Foundry upgrades tooling, storage layout safety checks, and ERC-7201 namespaced storage guidance.
---

# Solidity Upgrades (Codex Draft)

## Pick the Right Proxy Pattern

- UUPS: default for most projects.
- Transparent: strict admin/user call separation.
- Beacon: many proxies sharing one implementation.

## Write Upgradeable Implementations Correctly

- Replace constructors with initializer functions.
- Lock the implementation constructor with `_disableInitializers()`.
- Use upgradeable base contracts from `@openzeppelin/contracts-upgradeable` where required.
- Guard UUPS `_authorizeUpgrade` with access control.

## Storage Compatibility Rules

- Never reorder, remove, or type-change existing state variables.
- Never insert variables before existing state.
- Only append new state, or use namespaced storage patterns.
- Never change inheritance order in a deployed upgrade chain.

## Major-Version Restriction

- Do not recommend in-place upgrades from OpenZeppelin Contracts v4 implementations to v5 implementations for existing proxies.
- Prefer new v5 proxy deployments and migration to new addresses.

## Namespaced Storage (ERC-7201)

- Use `@custom:storage-location` annotated structs.
- Compute and embed the real storage slot constant; do not leave placeholders.

Example formula:

`keccak256(abi.encode(uint256(keccak256(id)) - 1)) & ~bytes32(uint256(0xff))`

## Hardhat Workflow

- Install and register `@openzeppelin/hardhat-upgrades`.
- Use `deployProxy` and `upgradeProxy`.
- Track `.openzeppelin/` network files appropriately.

## Foundry Workflow

- Use `openzeppelin-foundry-upgrades`.
- Enable `ffi`, `ast`, `build_info`, and `storageLayout` output.
- Use `Upgrades.deployUUPSProxy` and `Upgrades.upgradeProxy`.
- Annotate V2 contracts with `@custom:oz-upgrades-from`.

## Validation Issue Hierarchy

1. Fix root cause first.
2. Use narrow source annotations for genuinely safe exceptions.
3. Use narrow flags only when annotations cannot apply.
4. Use broad bypasses only as a last resort with manual review.
