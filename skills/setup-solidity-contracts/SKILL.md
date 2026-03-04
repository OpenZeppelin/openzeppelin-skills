---
name: setup-solidity-contracts
description: "Set up a Solidity smart contract project with OpenZeppelin Contracts. Use when users need to: (1) create a new Hardhat or Foundry project, (2) install OpenZeppelin Contracts dependencies for Solidity, (3) configure remappings for Foundry, or (4) understand Solidity import conventions for OpenZeppelin."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Solidity Setup

For existing projects, detect the framework by looking for `hardhat.config.*` (Hardhat) or `foundry.toml` (Foundry). For new projects, ask the user which framework they prefer.

## Hardhat Setup

- Initialize project (only if starting a new project)

```bash
npx hardhat init        # Hardhat v2
npx hardhat --init      # Hardhat v3
```

- Install OpenZeppelin Contracts:

```bash
npm install @openzeppelin/contracts
```

- If using upgradeable contracts, also install the upgradeable variant:

```bash
npm install @openzeppelin/contracts-upgradeable
```

## Foundry Setup

- Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

- Initialize project (only if starting a new project)

```bash
forge init my-project
cd my-project
```

- Add OpenZeppelin Contracts:

```bash
forge install OpenZeppelin/openzeppelin-contracts@v<VERSION>
```

- If using upgradeable contracts, also add the upgradeable variant:

```bash
forge install OpenZeppelin/openzeppelin-contracts-upgradeable@v<VERSION>
```

> Look up the current version from https://github.com/OpenZeppelin/openzeppelin-contracts/releases. Pin to a release tag — without one, `forge install` pulls the default branch, which may be unstable.

- `remappings.txt` (if not using upgradeable contracts)

```text
@openzeppelin/contracts/=lib/openzeppelin-contracts/contracts/
```

- `remappings.txt` (if using upgradeable contracts)

```text
@openzeppelin/contracts/=lib/openzeppelin-contracts-upgradeable/lib/openzeppelin-contracts/contracts/
@openzeppelin/contracts-upgradeable/=lib/openzeppelin-contracts-upgradeable/contracts/
```

> **Note**
> The above remappings mean that both `@openzeppelin/contracts/` (including proxy contracts) and `@openzeppelin/contracts-upgradeable/` come from the `openzeppelin-contracts-upgradeable` submodule and its subdirectories, which includes its own transitive copy of `openzeppelin-contracts` of the same release version number. This format is needed for Etherscan verification to work. Particularly, any copies of `openzeppelin-contracts` that are installed separately are NOT used.

## Import Conventions

- Standard: `@openzeppelin/contracts/token/ERC20/ERC20.sol`
- Upgradeable: `@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol`
- Use upgradeable variants only when deploying behind proxies; otherwise use standard contracts.
