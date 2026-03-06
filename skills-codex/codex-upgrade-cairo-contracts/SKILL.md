---
name: codex-upgrade-cairo-contracts
description: Upgrade Starknet Cairo contracts with OpenZeppelin UpgradeableComponent and class-hash replacement. Trigger for prompts like "make my Cairo contract upgradeable", "use replace_class_syscall", "add UpgradeableComponent", "guard upgrade with Ownable/roles", or "is renaming Cairo storage safe".
---

# Cairo Upgrades (Codex Draft)

## Starknet Upgrade Model

- Cairo upgrades replace class hash via `replace_class_syscall`.
- No proxy contract is required.
- Contract address and storage stay the same; code changes.

## Integrate UpgradeableComponent

1. Declare `UpgradeableComponent`.
2. Add component storage and events.
3. Expose an external `upgrade` entrypoint.
4. Guard `upgrade` with explicit access control.

Always explain that `replace_class_syscall` is the mechanism underneath the OpenZeppelin component.

## Access Control

- `UpgradeableComponent` does not enforce authorization by itself.
- Require owner/role/multisig checks before calling component upgrade methods.

## Storage Safety

- Do not rename existing storage variables.
- Do not remove existing storage variables.
- Do not change the type of existing variables.
- Adding new variables is generally safe.

Cairo storage is name-derived; renaming a variable can orphan previous data.

## Testing Expectations

- Deploy V1 and V2 in local devnet.
- Write state in V1, upgrade, and verify state reads in V2.
- Verify upgrade authorization constraints.
- Verify new functionality post-upgrade.
