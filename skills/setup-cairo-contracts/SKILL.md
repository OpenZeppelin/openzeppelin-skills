---
name: setup-cairo-contracts
description: "Set up a Cairo smart contract project with OpenZeppelin Contracts for Cairo on Starknet. Use when users need to: (1) create a new Scarb/Starknet project, (2) add OpenZeppelin Contracts for Cairo dependencies to Scarb.toml, (3) configure individual or umbrella OpenZeppelin packages, or (4) understand Cairo import conventions and component patterns for OpenZeppelin."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Cairo Setup

## Project Scaffolding

Install toolchain and create a project:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.starkup.sh | sh
scarb new my_project --test-runner=starknet-foundry
```

This scaffolds a complete Starknet project with `snforge` testing preconfigured.

## OpenZeppelin Dependencies

Look up the current version from https://docs.openzeppelin.com/contracts-cairo before adding. Add to `Scarb.toml`:

Full library (umbrella package):

```toml
[dependencies]
openzeppelin = "<VERSION>"
```

Individual packages (faster builds — only compiles what you use):

```toml
[dependencies]
openzeppelin_token = "<VERSION>"
openzeppelin_access = "<VERSION>"
```

Available individual packages: `openzeppelin_access`, `openzeppelin_account`, `openzeppelin_finance`, `openzeppelin_governance`, `openzeppelin_interfaces`, `openzeppelin_introspection`, `openzeppelin_merkle_tree`, `openzeppelin_presets`, `openzeppelin_security`, `openzeppelin_token`, `openzeppelin_upgrades`, `openzeppelin_utils`.

> `openzeppelin_interfaces` and `openzeppelin_utils` are versioned independently. Check the docs for their specific versions. All other packages share the same version.

## Import Conventions

The import path depends on which dependency is declared:

- **Umbrella package** (`openzeppelin = "..."`): use `openzeppelin::` as the root
- **Individual packages** (`openzeppelin_token = "..."`): use the package name as the root

```cairo
// Individual packages
use openzeppelin_token::erc20::{ERC20Component, ERC20HooksEmptyImpl};
use openzeppelin_access::ownable::OwnableComponent;
use openzeppelin_upgrades::UpgradeableComponent;

// Umbrella package equivalents
use openzeppelin::token::erc20::{ERC20Component, ERC20HooksEmptyImpl};
use openzeppelin::access::ownable::OwnableComponent;
use openzeppelin::upgrades::UpgradeableComponent;
```

Components are integrated via the `component!` macro, embedded impls, and substorage:

```cairo
component!(path: ERC20Component, storage: erc20, event: ERC20Event);

#[abi(embed_v0)]
impl ERC20MixinImpl = ERC20Component::ERC20MixinImpl<ContractState>;
impl ERC20InternalImpl = ERC20Component::InternalImpl<ContractState>;
```
