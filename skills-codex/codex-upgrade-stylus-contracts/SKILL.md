---
name: codex-upgrade-stylus-contracts
description: Upgrade Arbitrum Stylus contracts using OpenZeppelin UUPS or Beacon proxy patterns. Trigger for prompts like "make this Stylus contract upgradeable", "set up UUPS in Stylus", "how does logic_flag work", "upgrade Stylus proxy V1 to V2", "protect upgrade_to_and_call", or "check Stylus storage compatibility".
---

# Stylus Upgrades (Codex Draft)

## Stylus Upgrade Model

- Stylus uses EVM-compatible storage and proxy mechanisms.
- Solidity proxies and Stylus implementations can interoperate.
- UUPS, Beacon, and ERC-1967 concepts apply.

## UUPS Integration Checklist

1. Add `UUPSUpgradeable` and access-control fields in `#[storage]`.
2. Run `self.uups.constructor()` in implementation constructor.
3. Expose initialization path that calls `set_version()` through the proxy.
4. Implement `IUUPSUpgradeable` with guarded `upgrade_to_and_call`.
5. Implement `IErc1822Proxiable` and return the correct UUID.

## Context Detection (Stylus-specific)

- Stylus has no `immutable`; UUPS uses `logic_flag`.
- Implementation constructor sets `logic_flag = true`.
- Through delegatecall/proxy storage, `logic_flag` reads false.
- `only_proxy()` uses this behavior plus ERC-1967 checks.

## Safety Rules

- Apply the same storage compatibility rules as Solidity upgrades.
- Append new fields only.
- Do not reorder/remove/type-change existing fields.
- Always protect upgrade functions with explicit authorization.

## Reactivation Requirement

- Plan for periodic Stylus WASM reactivation (about every 365 days or after Stylus protocol upgrades).
- Treat this as operational maintenance separate from proxy upgrades.

## Testing Expectations

- Test V1 deploy, state writes, upgrade to V2, and state preservation.
- Verify authorization failures for unauthorized upgrade attempts.
- Verify version updates and post-upgrade functionality.
