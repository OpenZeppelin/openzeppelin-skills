---
name: codex-setup-stylus-contracts
description: Set up Arbitrum Stylus Rust projects with OpenZeppelin Contracts for Stylus. Use when users need Rust/WASM toolchain setup, Cargo Stylus project scaffolding, dependency pinning, and Stylus import/storage conventions.
---

# Stylus Setup (Codex Draft)

## Toolchain

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-unknown-unknown
cargo install --force cargo-stylus
```

Create project:

```bash
cargo stylus new my_project
```

Use a nightly toolchain and include `rust-src` + `wasm32-unknown-unknown` in `rust-toolchain.toml`.

## Dependency Configuration

Check the latest crate release before pinning.

```toml
[dependencies]
openzeppelin-stylus = "=<VERSION>"

[features]
export-abi = ["openzeppelin-stylus/export-abi"]

[lib]
crate-type = ["lib", "cdylib"]
```

## Import and Contract Patterns

- Use `openzeppelin_stylus` as crate root.
- Use `#[entrypoint]` and `#[storage]` on the main contract struct.
- Expose methods via `#[public]` and trait `impl` blocks.

## Basic Commands

```bash
cargo stylus check
cargo stylus export-abi
cargo stylus deploy --endpoint="<RPC_URL>" --private-key-path="<KEY_FILE>"
```
