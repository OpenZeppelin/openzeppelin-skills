---
name: codex-setup-cairo-contracts
description: Set up Starknet Cairo projects with OpenZeppelin Contracts for Cairo. Use when users need Scarb project scaffolding, OpenZeppelin dependency configuration, package selection, and Cairo component import/integration conventions.
---

# Cairo Setup (Codex Draft)

## Scaffold Project

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.starkup.sh | sh
scarb new my_project --test-runner=starknet-foundry
```

## Add OpenZeppelin Dependencies

Check current versions from the OpenZeppelin Cairo docs before writing `Scarb.toml`.

Umbrella package:

```toml
[dependencies]
openzeppelin = "<VERSION>"
```

Individual packages:

```toml
[dependencies]
openzeppelin_token = "<VERSION>"
openzeppelin_access = "<VERSION>"
```

Note that `openzeppelin_interfaces` and `openzeppelin_utils` are versioned independently.

## Import Conventions

Use root path based on selected dependency style:

- Umbrella: `openzeppelin::...`
- Individual: `openzeppelin_token::...`, `openzeppelin_access::...`, etc.

Typical integration model:

- Use `component!` macro declarations.
- Add component substorage to `Storage`.
- Add embedded impl blocks for ABI/internal methods.
