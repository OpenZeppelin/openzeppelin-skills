---
name: codex-setup-stellar-contracts
description: Bootstrap Stellar Soroban Rust projects with OpenZeppelin Stellar contracts. Trigger for prompts like "set up Soroban project", "install Stellar CLI + Rust target", "add OpenZeppelin Stellar crates", "configure workspace.dependencies", or "show Soroban contract import/trait patterns".
---

# Stellar Setup (Codex Draft)

## Toolchain and Project

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32v1-none
curl -fsSL https://github.com/stellar/stellar-cli/raw/main/install.sh | sh
stellar contract init my_project
```

## Workspace Dependency Pattern

Pin OpenZeppelin crates in root `Cargo.toml`:

```toml
[workspace.dependencies]
stellar-tokens = "=<VERSION>"
stellar-access = "=<VERSION>"
stellar-contract-utils = "=<VERSION>"
stellar-macros = "=<VERSION>"
```

Reference them in each contract crate:

```toml
[dependencies]
soroban-sdk = { workspace = true }
stellar-tokens = { workspace = true }
stellar-access = { workspace = true }
stellar-contract-utils = { workspace = true }
stellar-macros = { workspace = true }
```

## Contract Patterns

- Use `#[contract]` and `#[contractimpl]`.
- Implement trait-based modules (token/access/utility traits).
- Use macros like `#[only_owner]` or `#[when_not_paused]` when appropriate.

## Build and Test

```bash
stellar contract build
cargo test
```
