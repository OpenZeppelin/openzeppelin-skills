---
name: setup-stellar-contracts
description: "Set up a Stellar/Soroban smart contract project with OpenZeppelin Contracts for Stellar. Use when users need to: (1) install Stellar CLI and Rust toolchain for Soroban, (2) create a new Soroban project, (3) add OpenZeppelin Stellar dependencies to Cargo.toml, or (4) understand Soroban import conventions and contract patterns for OpenZeppelin."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Stellar Setup

## Soroban/Stellar Development Setup

Install the Rust toolchain (v1.84.0+) and the Soroban WASM target:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32v1-none
```

Install the Stellar CLI:

```bash
curl -fsSL https://github.com/stellar/stellar-cli/raw/main/install.sh | sh
```

Create a new Soroban project:

```bash
stellar contract init my_project
```

This creates a Cargo workspace with contracts in `contracts/*/`.

## OpenZeppelin Dependencies

Look up the current version from the [stellar-contracts repo](https://github.com/OpenZeppelin/stellar-contracts) before adding. Pin exact versions with `=` as the library is under active development.

Add OpenZeppelin crates to the **root** `Cargo.toml` under `[workspace.dependencies]`:

```toml
[workspace.dependencies]
stellar-tokens = "=<VERSION>"
stellar-access = "=<VERSION>"
stellar-contract-utils = "=<VERSION>"
stellar-macros = "=<VERSION>"
```

Then reference them in the **per-contract** `contracts/*/Cargo.toml`:

```toml
[dependencies]
soroban-sdk = { workspace = true }
stellar-tokens = { workspace = true }
stellar-access = { workspace = true }
stellar-contract-utils = { workspace = true }
stellar-macros = { workspace = true }
```

Available crates: `stellar-access`, `stellar-accounts`, `stellar-contract-utils`, `stellar-fee-abstraction`, `stellar-governance`, `stellar-macros`, `stellar-tokens`.

> Only add the crates the contract actually uses. `stellar-macros` provides proc-macro attributes (for example, `#[when_not_paused]`, `#[only_owner]`, `#[derive(Upgradeable)]`) and is needed in most contracts.

## Code Patterns

Imports use underscores as the crate root (Rust convention):

```rust
use stellar_tokens::fungible::{Base, FungibleToken};
use stellar_tokens::fungible::burnable::FungibleBurnable;
use stellar_access::ownable::Ownable;
use stellar_contract_utils::pausable::Pausable;
use stellar_macros::when_not_paused;
```

Contracts use `#[contract]` on the struct and `#[contractimpl]` on the impl block (from `soroban_sdk`):

```rust
use soroban_sdk::{contract, contractimpl, Env};

#[contract]
pub struct MyToken;

#[contractimpl]
impl MyToken {
    // Implement trait methods here
}
```

Trait implementations are separate `impl` blocks per trait (e.g., `FungibleToken`, `Pausable`). Guard macros like `#[when_not_paused]` and `#[only_owner]` decorate individual functions.

## Platform Notes

- **Read operations are free in Stellar.** Optimize for minimizing writes; reads and computation are cheap. Prefer clean, readable code over micro-optimizations.
- **Instance storage TTL extension is the developer's responsibility.** The OpenZeppelin library handles TTL extension for other storage entries, but contracts must extend their own `instance` storage entries to prevent expiration.

## Build & Test

Build the contract to WASM:

```bash
stellar contract build
```

This is a shortcut for `cargo build --target wasm32v1-none --release`. Output appears in `target/wasm32v1-none/release/`.

Run tests:

```bash
cargo test
```

> `soroban-sdk` testutils are automatically enabled for in-crate unit tests.
