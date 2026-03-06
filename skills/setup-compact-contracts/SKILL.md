---
name: setup-compact-contracts
description: "Set up a Compact smart contract project for the Midnight Network with OpenZeppelin Contracts for Compact. Use when users need to: (1) install Compact Developer Tools and Node.js for Midnight development, (2) create a new Compact project, (3) add OpenZeppelin Compact Contracts as a dependency, or (4) understand Compact import conventions for OpenZeppelin modules."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Compact Setup (Midnight Network)

Compact is the smart contract language for the [Midnight Network](https://midnight.network/), a privacy-focused blockchain using zero-knowledge proofs. Contracts are written in `.compact` files and compiled to ZK circuits.

## Prerequisites

- **Node.js**: v22+ (check `.nvmrc` if present)
- **Yarn**: v4+
- **Compact Developer Tools**: The `compact` CLI compiler

### Install Compact Developer Tools

Follow Midnight's [Compact Developer Tools installation guide](https://docs.midnight.network/develop/tutorial/building/#midnight-compact-compiler) and confirm `compact` is in your `PATH`:

```bash
compact compile --version
```

### Install Node.js

```bash
nvm install && nvm use
```

## Project Setup

### Add OpenZeppelin submodules

Initialize git (if not already) and add both required submodules:

```bash
git init && \
git submodule add https://github.com/OpenZeppelin/compact-tools.git && \
git submodule add https://github.com/OpenZeppelin/compact-contracts.git
```

### Build the submodules

Build compact-tools (CLI tools for compiling Compact contracts):

```bash
cd compact-tools && yarn install && yarn build && cd -
```

Build compact-contracts (OpenZeppelin Compact contracts library):

```bash
cd compact-contracts && \
nvm install && \
yarn && \
SKIP_ZK=true yarn compact && \
cd -
```

> `SKIP_ZK=true` skips ZK prover/verifier key generation, which is slow and usually unnecessary during development.

### Write a contract

Create a `.compact` file in the project root. Import OpenZeppelin modules through the submodule's `node_modules`:

```typescript
// MyContract.compact

pragma language_version >= 0.18.0;

import { ZswapCoinPublicKey, ContractAddress, Either } from CompactStandardLibrary;
import { initialize, assertOnlyOwner } from "./compact-contracts/node_modules/@openzeppelin/compact-contracts/src/access/Ownable"
  prefix Ownable_;
import { assertNotPaused, _pause, _unpause } from "./compact-contracts/node_modules/@openzeppelin/compact-contracts/src/security/Pausable"
  prefix Pausable_;
import { initialize, transfer, _mint } from "./compact-contracts/node_modules/@openzeppelin/compact-contracts/src/token/FungibleToken"
  prefix FungibleToken_;

constructor(
  _name: Opaque<"string">,
  _symbol: Opaque<"string">,
  _decimals: Uint<8>,
  _recipient: Either<ZswapCoinPublicKey, ContractAddress>,
  _amount: Uint<128>,
  _initOwner: Either<ZswapCoinPublicKey, ContractAddress>,
) {
  Ownable_initialize(_initOwner);
  FungibleToken_initialize(_name, _symbol, _decimals);
  FungibleToken__mint(_recipient, _amount);
}

export circuit transfer(
  to: Either<ZswapCoinPublicKey, ContractAddress>,
  value: Uint<128>,
): Boolean {
  Pausable_assertNotPaused();
  return FungibleToken_transfer(to, value);
}

export circuit pause(): [] {
  Ownable_assertOnlyOwner();
  Pausable__pause();
}

export circuit unpause(): [] {
  Ownable_assertOnlyOwner();
  Pausable__unpause();
}
```

> Import modules through `node_modules` rather than directly to avoid state conflicts between shared dependencies.

### Write a reusable module

Compact supports writing reusable modules using the `module` keyword. Modules can export circuits, structs, ledger state, and witnesses:

```typescript
// math/SafeMath.compact

pragma language_version >= 0.21.0;

module SafeMath {
  export struct DivResult {
    quotient: Uint<64>,
    remainder: Uint<64>
  }

  // Witnesses perform off-chain computation, verified on-chain.
  // Convention: witness names start with `wit_`.
  witness wit_div(a: Uint<64>, b: Uint<64>): DivResult;

  /**
   * @title MAX_UINT64 circuit
   * @description Returns the maximum value for a 64-bit unsigned integer.
   *
   * @circuitInfo k=5, rows=25
   *
   * @returns {Uint<64>} The value 2^64 - 1.
   */
  export pure circuit MAX_UINT64(): Uint<64> {
    return 0xFFFFFFFFFFFFFFFF;
  }

  /**
   * @title addChecked circuit
   * @description Adds two Uint<64> numbers with overflow checking.
   *
   * @circuitInfo k=9, rows=298
   *
   * @param {Uint<64>} a - The first operand.
   * @param {Uint<64>} b - The second operand.
   *
   * @throws {Error} "Math: addition overflow" if the sum exceeds MAX_UINT64.
   *
   * @returns {Uint<64>} The sum of a and b.
   */
  export circuit addChecked(a: Uint<64>, b: Uint<64>): Uint<64> {
    const sum: Uint<128> = a + b;
    assert(sum <= MAX_UINT64(), "Math: addition overflow");
    return sum as Uint<64>;
  }

  /**
   * @title sub circuit
   * @description Subtracts b from a with underflow checking.
   *
   * @circuitInfo k=9, rows=212
   *
   * @param {Uint<64>} a - The minuend.
   * @param {Uint<64>} b - The subtrahend.
   *
   * Requirements:
   * - `a` must be greater than or equal to `b`.
   *
   * @throws {Error} "Math: subtraction underflow" if b > a.
   *
   * @returns {Uint<64>} The difference a - b.
   */
  export circuit sub(a: Uint<64>, b: Uint<64>): Uint<64> {
    assert(a >= b, "Math: subtraction underflow");
    return a - b;
  }

  // Internal circuits (not exported) are prefixed with underscore
  circuit _div(a: Uint<64>, b: Uint<64>): DivResult {
    assert(b != 0, "Math: division by zero");
    const result = wit_div(a, b);
    assert(result.remainder < b, "Math: remainder error");
    assert((result.quotient * b + result.remainder) as Uint<64> == a, "Math: division invalid");
    return result;
  }

  /**
   * @title div circuit
   * @description Divides a by b, returning the quotient.
   *
   * @circuitInfo k=9, rows=411
   *
   * @param {Uint<64>} a - The dividend.
   * @param {Uint<64>} b - The divisor.
   *
   * @throws {Error} "Math: division by zero" if b is zero.
   *
   * @returns {Uint<64>} The quotient of a divided by b.
   */
  export circuit div(a: Uint<64>, b: Uint<64>): Uint<64> {
    return _div(a, b).quotient;
  }
}
```

Then import from another contract or module:

```typescript
import { addChecked, sub, div } from "./math/SafeMath" prefix SafeMath_;

// SafeMath_addChecked(a, b), SafeMath_sub(a, b), SafeMath_div(a, b)
```

### Compile the contract

```bash
compact compile MyContract.compact artifacts/MyContract
```

## Import Conventions

Compact supports named imports (similar to TypeScript/ES modules) and an optional `prefix` keyword:

```typescript
// Named imports (default — import specific identifiers from a module)
import { initialize, assertOnlyOwner } from "./path/to/Ownable";

// Named imports with prefix (namespaces to avoid collisions across modules)
import { initialize, assertOnlyOwner } from "./path/to/Ownable" prefix Ownable_;
// Ownable_initialize(...), Ownable_assertOnlyOwner(), etc.

// Wildcard import (all exports)
import "./path/to/Ownable" prefix Ownable_;

// Wildcard import without prefix (all exports used directly)
import "./path/to/Ownable";
```

When using a prefix, internal (underscore-prefixed) identifiers like `_mint` become double-underscored: `FungibleToken__mint(...)`.

Ask the user whether they prefer named imports, prefixed imports, or a combination. Named imports are explicit and familiar to TypeScript developers; prefixes are useful when importing many identifiers from multiple modules.

## Available OpenZeppelin Compact Modules

| Category | Modules | Import Path |
|----------|---------|-------------|
| Access Control | `Ownable`, `AccessControl`, `ZOwnablePK` | `src/access/` |
| Security | `Pausable`, `Initializable` | `src/security/` |
| Tokens | `FungibleToken`, `NonFungibleToken`, `MultiToken` | `src/token/` |
| Utilities | `Utils` | `src/utils/` |

## Compact Language Basics

- **Pragma**: `pragma language_version >= X.Y.Z;` — specifies the minimum Compact language version. Check existing `.compact` files in the project or the OpenZeppelin library for the current version to use.
- **Circuits**: Functions in Compact are called "circuits" (compiled to ZK circuits)
- **Pure circuits**: `export pure circuit` — no side effects (no ledger access)
- **Export**: `export circuit` makes a circuit callable from outside the contract
- **Ledger**: `export ledger` declares on-chain state variables
- **Witnesses**: Off-chain computation functions, named with `wit_` prefix by convention (e.g., `witness wit_sqrt(...)`)
- **Types**: `Uint<N>`, `Boolean`, `Bytes<N>`, `Either<A, B>`, `Opaque<"string">`, `ZswapCoinPublicKey`, `ContractAddress`
- **Structs**: `export struct` for structured types
- **Constructor**: Top-level `constructor(...)` block initializes the contract
- **Documentation**: Use JSDoc-style comments for circuits following this template:

```typescript
/**
 * @title circuitName circuit
 * @description Brief description of what the circuit does.
 *
 * @circuitInfo k=N, rows=N
 *
 * @param {Type} name - Parameter description.
 *
 * Requirements:                          // optional, only if there are preconditions
 * - Precondition description.
 *
 * @throws {Error} "message" if condition.
 *
 * @returns {Type} Description.
 */
```

## Key Midnight SDK Packages (for TypeScript integration)

| Package | Purpose |
|---------|---------|
| `@midnight-ntwrk/compact-runtime` | Runtime for compiled contracts |
| `@midnight-ntwrk/midnight-js-contracts` | Contract deployment and interaction |
| `@midnight-ntwrk/midnight-js-types` | Type definitions |
| `@midnight-ntwrk/zswap` | Zero-knowledge token swap library |
| `@midnight-ntwrk/ledger-*` | Ledger integration (version suffix changes across releases, check `package.json` for the current one) |

## Testing

Tests use **Vitest** with TypeScript. Contract behavior is tested through simulator classes that wrap compiled contract artifacts:

```bash
# Run tests (skips ZK key generation)
SKIP_ZK=true vitest run

# Or via Turbo
turbo test
```

## Documentation

- [OpenZeppelin Contracts for Compact docs](https://docs.openzeppelin.com/contracts-compact)
- [Midnight Network docs](https://docs.midnight.network/)
- [Compact language reference](https://docs.midnight.network/develop/reference/compact/)
