---
name: codex-upgrade-stellar-contracts
description: Upgrade Stellar Soroban contracts with OpenZeppelin Upgradeable or UpgradeableMigratable modules. Trigger for prompts like "make my Soroban contract upgradeable", "upgrade and migrate Soroban state atomically", "use Upgradeable derive macro", "implement _require_auth for upgrades", or "is changing storage key/type safe in Soroban".
---

# Stellar Upgrades (Codex Draft)

## Soroban Upgrade Model

- Soroban upgrades use native WASM replacement.
- No proxy is required.
- Contracts are mutable only if upgrade functions are exposed.

## Choose Upgrade Mode

- `Upgradeable`: upgrade code only.
- `UpgradeableMigratable`: upgrade code plus storage migration.

Use derive macros and implement required internal authorization hooks.

## Access Control

- Upgrade modules do not enforce auth automatically.
- Implement `_require_auth` correctly to restrict upgrade authority.

## Atomic Upgrade + Migration

- The new implementation is active only after the current invocation completes.
- For atomic upgrade-and-migrate flows, use an auxiliary Upgrader contract that performs both steps safely.

## Storage Safety

- Do not rename existing storage keys.
- Do not remove existing storage keys.
- Do not change value types under existing keys.
- Adding new keys is generally safe.

## Testing Expectations

- Test V1 deployment and state writes.
- Upgrade to V2 and verify prior state.
- Test migrate flows (if used) and auth constraints.
- Confirm V2 still exposes the desired upgrade mechanism to avoid accidental immutability.
