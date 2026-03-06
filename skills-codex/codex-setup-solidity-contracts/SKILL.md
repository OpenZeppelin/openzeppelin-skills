---
name: codex-setup-solidity-contracts
description: Set up Solidity projects with OpenZeppelin Contracts. Use when users need Hardhat or Foundry initialization, dependency installation, remappings configuration, or import convention guidance for standard and upgradeable OpenZeppelin Solidity packages.
---

# Solidity Setup (Codex Draft)

## Detect Existing Framework

- Detect Hardhat from `hardhat.config.*`.
- Detect Foundry from `foundry.toml`.
- If neither exists and the user is creating a new project, ask which framework they want.

## Hardhat

Initialize a new project only when needed:

```bash
npx hardhat init        # v2
npx hardhat --init      # v3
```

Install dependencies:

```bash
npm install @openzeppelin/contracts
npm install @openzeppelin/contracts-upgradeable   # only when upgradeable contracts are needed
```

## Foundry

Initialize only when needed:

```bash
forge init my-project
cd my-project
```

Install pinned releases (look up latest release tag first):

```bash
forge install OpenZeppelin/openzeppelin-contracts@v<VERSION>
forge install OpenZeppelin/openzeppelin-contracts-upgradeable@v<VERSION>   # if needed
```

Configure `remappings.txt`:

Standard only:

```text
@openzeppelin/contracts/=lib/openzeppelin-contracts/contracts/
```

Upgradeable:

```text
@openzeppelin/contracts/=lib/openzeppelin-contracts-upgradeable/lib/openzeppelin-contracts/contracts/
@openzeppelin/contracts-upgradeable/=lib/openzeppelin-contracts-upgradeable/contracts/
```

## Import Conventions

- Standard contracts: `@openzeppelin/contracts/...`
- Upgradeable contracts: `@openzeppelin/contracts-upgradeable/...`
- Use upgradeable variants only behind proxies.
