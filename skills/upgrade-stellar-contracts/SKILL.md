---
name: upgrade-stellar-contracts
description: "Upgrade Stellar/Soroban smart contracts using OpenZeppelin's upgradeable module. Use when users need to: (1) make Soroban contracts upgradeable via native WASM replacement, (2) use Upgradeable or UpgradeableMigratable derive macros, (3) implement atomic upgrade-and-migrate patterns with an Upgrader contract, (4) ensure storage key compatibility across upgrades, or (5) test upgrade paths for Soroban contracts."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Stellar Upgrades

## Contents

- [Soroban Upgrade Model](#soroban-upgrade-model)
- [Using the OpenZeppelin Upgradeable Module](#using-the-openzeppelin-upgradeable-module)
- [Access Control](#access-control)
- [Upgrade Safety](#upgrade-safety)

## Soroban Upgrade Model

Soroban contracts are **mutable by default**. Mutability refers to the ability of a smart contract to modify its own WASM bytecode, altering its function interface, execution logic, or metadata. Soroban provides a **built-in, protocol-level mechanism** for contract upgrades — no proxy pattern is needed.

A contract can upgrade itself if it is explicitly designed to do so. Conversely, a contract becomes immutable simply by not provisioning any upgrade function. This is fundamentally different from EVM proxy patterns:

| | Soroban | EVM (proxy pattern) | Starknet |
|---|---|---|---|
| **Mechanism** | Native WASM bytecode replacement | Proxy `delegatecall`s to implementation contract | `replace_class_syscall` swaps class hash in-place |
| **Proxy contract needed** | No — the contract upgrades itself | Yes — a proxy sits in front of the implementation | No — the contract upgrades itself |
| **Storage location** | Belongs to the contract directly | Lives in the proxy, accessed via delegatecall | Belongs to the contract directly |
| **Opt-in to immutability** | Don't expose an upgrade function | Don't deploy a proxy | Don't call the syscall |

One advantage of protocol-level upgradeability is a significantly reduced risk surface compared to platforms that require proxy contracts and delegatecall forwarding.

The new implementation only becomes effective **after the current invocation completes**. This means if migration logic is defined in the new implementation, it cannot execute within the same call as the upgrade. An auxiliary `Upgrader` contract can wrap both calls to achieve atomicity (see below).

## Using the OpenZeppelin Upgradeable Module

OpenZeppelin Stellar Soroban Contracts provides an `upgradeable` module in the `contract-utils` package with two main components:

| Component | Use when |
|-----------|----------|
| **`Upgradeable`** | Only the WASM binary needs to be updated — no storage migration required |
| **`UpgradeableMigratable`** | The WASM binary and specific storage entries need to be modified during the upgrade |

The recommended way to use these is through derive macros: `#[derive(Upgradeable)]` and `#[derive(UpgradeableMigratable)]`. These macros handle the implementation of necessary functions and set the crate version from `Cargo.toml` as the binary version in WASM metadata, aligning with SEP-49 guidelines.

### Upgrade only

Derive `Upgradeable` on the contract struct, then implement `UpgradeableInternal` with a single required method:

- `_require_auth(e: &Env, operator: &Address)` — verify the operator is authorized to perform the upgrade (e.g., check against a stored owner address)

The `operator` parameter is the invoker of the upgrade function and can be used for role-based access control.

### Upgrade and migrate

Derive `UpgradeableMigratable` on the contract struct, then implement `UpgradeableMigratableInternal` with:

- An associated `MigrationData` type defining the data passed to the migration function
- `_require_auth(e, operator)` — same authorization check as above
- `_migrate(e: &Env, data: &Self::MigrationData)` — perform storage modifications using the provided migration data

The derive macro ensures that migration can only be invoked **after** a successful upgrade, preventing state inconsistencies and storage corruption.

### Atomic upgrade and migration

Because the new implementation only takes effect after the current invocation completes, migration logic in the new contract cannot run in the same call as the upgrade. An auxiliary `Upgrader` contract wraps both calls atomically:

```rust
use soroban_sdk::{contract, contractimpl, symbol_short, Address, BytesN, Env, Val};
use stellar_contract_utils::upgradeable::UpgradeableClient;

#[contract]
pub struct Upgrader;

#[contractimpl]
impl Upgrader {
    pub fn upgrade_and_migrate(
        env: Env,
        contract_address: Address,
        operator: Address,
        wasm_hash: BytesN<32>,
        migration_data: soroban_sdk::Vec<Val>,
    ) {
        operator.require_auth();
        let contract_client = UpgradeableClient::new(&env, &contract_address);
        contract_client.upgrade(&wasm_hash, &operator);
        env.invoke_contract::<()>(
            &contract_address,
            &symbol_short!("migrate"),
            migration_data,
        );
    }
}
```

Guard `upgrade_and_migrate` with proper access control (e.g., `Ownable` from the `access` package with `#[only_owner]`).

If a rollback is required, the contract can be upgraded to a newer version where rollback-specific logic is defined and performed as a migration.

> **Examples:** See the `examples/` directory of the [stellar-contracts repository](https://github.com/OpenZeppelin/stellar-contracts) for full working integration examples of both `Upgradeable` and `UpgradeableMigratable`, including the `Upgrader` pattern.

## Access Control

The `upgradeable` module deliberately does **not** embed access control itself. You must define authorization in the `_require_auth` method of `UpgradeableInternal` or `UpgradeableMigratableInternal`. Forgetting this allows anyone to replace your contract's code.

Common access control options:
- **Ownable** — single owner, simplest pattern (available in the `access` package)
- **AccessControl / RBAC** — role-based, finer granularity (available in the `access` package)
- **Multisig or governance** — for production contracts managing significant value

## Upgrade Safety

### Caveats

The framework structures the upgrade flow but does **not** perform deeper checks:

- The new contract's **constructor will not be invoked** — any initialization must happen via migration or a separate call
- There is **no automatic check** that the new contract includes an upgrade mechanism — an upgrade to a contract without one permanently loses upgradeability
- **Storage consistency is not verified** — the new contract may inadvertently introduce storage mismatches

### Storage compatibility

When replacing the WASM binary, existing storage is reinterpreted by the new code. Incompatible changes corrupt state:

- **Do not remove or rename** existing storage keys
- **Do not change the type** of values stored under existing keys
- **Adding** new storage keys is safe
- Soroban storage uses explicit string keys (e.g., `symbol_short!("OWNER")`), so key naming is critical — unlike EVM sequential slots, there is no ordering dependency

### Version tracking

The derive macros automatically extract the crate version from `Cargo.toml` and embed it as the binary version in the WASM metadata, following SEP-49. This enables on-chain version tracking and can be used to coordinate upgrade paths.

### Testing upgrade paths

Before upgrading a production contract:

- [ ] **Deploy V1** on a local Soroban testnet (e.g., `stellar-cli` with local network)
- [ ] **Write state with V1**, upgrade to V2, and verify that all existing state reads correctly
- [ ] **Verify new functionality** works as expected after the upgrade
- [ ] **Confirm access control** — only authorized callers can invoke `upgrade`
- [ ] **Check that V2 includes an upgrade mechanism** — otherwise upgradeability is permanently lost
- [ ] **Verify storage key compatibility** — ensure no removals, renames, or type changes to existing keys
- [ ] **Test atomic upgrade-and-migrate** using the `Upgrader` pattern if migration is needed
- [ ] **Manual review** — there is no automated storage compatibility validation for Soroban; use the derive macros for safe upgrade scaffolding and rely on testnet testing
