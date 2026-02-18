# Solidity Upgrades

## Contents

- [Proxy Patterns Overview](#proxy-patterns-overview)
- [Upgrade Restrictions Between Major Versions (v4 → v5)](#upgrade-restrictions-between-major-versions-v4--v5)
- [Writing Upgradeable Contracts](#writing-upgradeable-contracts)
- [Hardhat Upgrades Workflow](#hardhat-upgrades-workflow)
- [Foundry Upgrades Workflow](#foundry-upgrades-workflow)
- [Upgrade Safety Checklist](#upgrade-safety-checklist)

## Proxy Patterns Overview

| Pattern | Upgrade logic lives in | Best for |
|---------|----------------------|----------|
| **UUPS** (`UUPSUpgradeable`) | Implementation contract (override `_authorizeUpgrade`) | Most projects — lighter proxy, lower deploy gas |
| **Transparent** | Separate `ProxyAdmin` contract | When admin/user call separation is critical — admin cannot accidentally call implementation functions |
| **Beacon** | Shared beacon contract | Multiple proxies sharing one implementation — upgrading the beacon atomically upgrades all proxies |

All three use EIP-1967 storage slots for the implementation address, admin, and beacon.

> **Transparent proxy — v5 constructor change:** In v5, `TransparentUpgradeableProxy` automatically deploys its own `ProxyAdmin` contract and stores the admin address in an immutable variable (set at construction time, never changeable). The second constructor parameter is the **owner address** for that auto-deployed `ProxyAdmin` — do **not** pass an existing `ProxyAdmin` contract address here. Transfer of upgrade capability is handled exclusively through `ProxyAdmin` ownership. This differs from v4, where `ProxyAdmin` was deployed separately and its address was passed to the proxy constructor.

## Upgrade Restrictions Between Major Versions (v4 → v5)

**Upgrading an existing proxy from a v4 implementation to a v5 implementation is not supported.**

v4 uses sequential storage (slots in declaration order); v5 uses namespaced storage (ERC-7201, structs at deterministic slots). A v5 implementation cannot safely read state written by a v4 implementation. Manual data migration is theoretically possible but often infeasible — `mapping` entries cannot be enumerated, so values written under arbitrary keys cannot be relocated.

**Recommended approach:** Deploy new proxies with v5 implementations and migrate users to the new address — do not upgrade proxies that currently point to v4 implementations.

**Updating your codebase to v5 is encouraged.** The restriction above applies only to already-deployed proxies. New deployments built on v5, and upgrades within the same major version, are fully supported.

## Writing Upgradeable Contracts

### Use initializers instead of constructors

Proxy contracts delegatecall into the implementation. Constructors run only when the implementation itself is deployed, not when a proxy is created. Replace constructors with initializer functions:

```solidity
import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract MyToken is Initializable, ERC20Upgradeable, OwnableUpgradeable {
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers(); // lock the implementation
    }

    function initialize(address owner) public initializer {
        __ERC20_init("MyToken", "MTK");
        __Ownable_init(owner);
    }
}
```

Key rules:
- Top-level `initialize` uses the `initializer` modifier
- Parent init functions (`__X_init`) use `onlyInitializing` internally — call them explicitly, the compiler does not auto-linearize initializers like constructors
- Always call `_disableInitializers()` in a constructor to prevent attackers from initializing the implementation directly
- Do not set initial values in field declarations (e.g., `uint256 x = 42`) — these compile into the constructor and won't execute for the proxy. `constant` is safe (inlined at compile time). `immutable` values are stored in bytecode and shared across all proxies — the plugins flag them as unsafe by default; use `/// @custom:oz-upgrades-unsafe-allow state-variable-immutable` to opt in when a shared value is intended

### Use the upgradeable package

Import from `@openzeppelin/contracts-upgradeable` for base contracts (e.g., `ERC20Upgradeable`, `OwnableUpgradeable`). Import interfaces and libraries from `@openzeppelin/contracts`.

### Storage layout rules

When upgrading, the new implementation must be storage-compatible with the old one:

- **Never** reorder, remove, or change the type of existing state variables
- **Never** insert new variables before existing ones
- **Only** append new variables at the end
- **Never** change the inheritance order of base contracts

### Namespaced storage (ERC-7201)

The modern approach — all `@openzeppelin/contracts-upgradeable` contracts (v5+) use this. State variables are grouped into a struct at a deterministic storage slot, isolating each contract's storage and eliminating the need for storage gaps. Recommended for all contracts that may be imported as base contracts.

```solidity
/// @custom:storage-location erc7201:example.main
struct MainStorage {
    uint256 value;
    mapping(address => uint256) balances;
}

// keccak256(abi.encode(uint256(keccak256("example.main")) - 1)) & ~bytes32(uint256(0xff))
bytes32 private constant MAIN_STORAGE_LOCATION = 0x...;

function _getMainStorage() private pure returns (MainStorage storage $) {
    assembly { $.slot := MAIN_STORAGE_LOCATION }
}
```

Using a variable from namespaced storage:
```solidity
function _getBalance(address account) internal view returns (uint256) {
    MainStorage storage $ = _getMainStorage();
    return $.balances[account];
}
```

Benefits over legacy storage gaps: safe to add variables to base contracts, inheritance order changes don't break layout, each contract's storage is fully isolated.

#### Computing ERC-7201 storage locations

When generating namespaced storage code, always compute the actual `STORAGE_LOCATION` constant. **Use the Bash tool to run the command below** with the actual namespace id and embed the computed value directly in the generated code. Never leave placeholder values like `0x...`.

The formula is: `keccak256(abi.encode(uint256(keccak256(id)) - 1)) & ~bytes32(uint256(0xff))` where `id` is the namespace string (e.g., `"example.main"`).

**Node.js with ethers** (available in Hardhat projects and Foundry upgrades projects):

```bash
node -e "const{keccak256,toUtf8Bytes,zeroPadValue,toBeHex}=require('ethers');const id=process.argv[1];const h=BigInt(keccak256(toUtf8Bytes(id)))-1n;console.log(toBeHex(BigInt(keccak256(zeroPadValue(toBeHex(h),32)))&~0xffn,32))" "example.main"
```

Replace `"example.main"` with the actual namespace id, run the command, and use the output as the constant value.

### Unsafe operations

- **No `selfdestruct`** — on pre-Dencun chains, destroys the implementation and bricks all proxies. Post-Dencun (EIP-6780), `selfdestruct` only destroys code if called in the same transaction as creation, but the plugins still flag it as unsafe
- **No `delegatecall`** to untrusted contracts — a malicious target could `selfdestruct` or corrupt storage

Additionally, avoid using `new` to create contracts inside an upgradeable contract — the created contract won't be upgradeable. Inject pre-deployed addresses instead.

## Hardhat Upgrades Workflow

Install the plugin:

```bash
npm install --save-dev @openzeppelin/hardhat-upgrades
npm install --save-dev @nomicfoundation/hardhat-ethers ethers  # peer dependencies
```

Register in `hardhat.config`:

```javascript
require('@openzeppelin/hardhat-upgrades'); // JS
import '@openzeppelin/hardhat-upgrades';   // TS
```

Workflow concept — the plugin provides functions on the `upgrades` object (`deployProxy`, `upgradeProxy`, `deployBeacon`, `upgradeBeacon`, `deployBeaconProxy`). Each function:
1. Validates the implementation for upgrade safety (storage layout, initializer patterns, unsafe opcodes)
2. Deploys the implementation (reuses if already deployed)
3. Deploys or updates the proxy/beacon
4. Calls the initializer (on deploy)

The plugin tracks deployed implementations in `.openzeppelin/` per-network files. Commit non-development network files to version control.

Use `prepareUpgrade` to validate and deploy a new implementation without executing the upgrade — useful when a multisig or governance contract holds upgrade rights.

> Read the installed plugin's README or source for exact API signatures and options, as these evolve across versions.

## Foundry Upgrades Workflow

Install dependencies:

```bash
forge install foundry-rs/forge-std
forge install OpenZeppelin/openzeppelin-foundry-upgrades
forge install OpenZeppelin/openzeppelin-contracts-upgradeable
```

Configure `foundry.toml`:

```toml
[profile.default]
ffi = true
ast = true
build_info = true
extra_output = ["storageLayout"]
```

> Node.js is required — the library shells out to the OpenZeppelin Upgrades CLI for validation.

Import and use in scripts/tests:

```solidity
import {Upgrades} from "openzeppelin-foundry-upgrades/Upgrades.sol";

// Deploy
address proxy = Upgrades.deployUUPSProxy(
    "MyContract.sol",
    abi.encodeCall(MyContract.initialize, (args))
);

// IMPORTANT: Before upgrading, annotate MyContractV2 with: /// @custom:oz-upgrades-from MyContract

// Upgrade and call a function
Upgrades.upgradeProxy(proxy, "MyContractV2.sol", abi.encodeCall(MyContractV2.foo, ("arguments for foo")));

// Upgrade without calling a function
Upgrades.upgradeProxy(proxy, "MyContractV2.sol", "");
```

Key differences from Hardhat:
- Contracts are referenced by name string, not factory object
- No automatic implementation tracking — annotate new versions with `@custom:oz-upgrades-from` or pass `referenceContract` in the `Options` struct
- `UnsafeUpgrades` variant skips all validation (takes addresses instead of names) — never use in production scripts
- Run `forge clean` or use `--force` before running scripts

> Read the installed library's `Upgrades.sol` for the full API and `Options` struct.

## Upgrade Safety Checklist

- [ ] **Storage compatibility**: No reordering, removal, or type changes of existing variables. Only append new variables (or add fields to namespaced structs).
- [ ] **Initializer protection**: Top-level `initialize` uses `initializer` modifier. Implementation constructor calls `_disableInitializers()`.
- [ ] **Parent initializers called**: Every inherited upgradeable contract's `__X_init` is called exactly once in `initialize`.
- [ ] **No unsafe opcodes**: No `selfdestruct` or `delegatecall` to untrusted targets.
- [ ] **Function selector clashes**: Proxy admin functions and implementation functions must not share selectors. UUPS and Transparent patterns handle this by design; custom proxies need manual review.
- [ ] **UUPS `_authorizeUpgrade`**: Overridden with proper access control (e.g., `onlyOwner`). Forgetting this makes the proxy non-upgradeable or upgradeable by anyone.
- [ ] **Test the upgrade path**: Deploy V1, upgrade to V2, verify state is preserved and new logic works. Both Hardhat and Foundry plugins can validate upgrades in test suites.
- [ ] **Reinitializer for V2+**: If V2 needs new initialization logic, use `reinitializer(2)` modifier (not `initializer`, which can only run once).
- [ ] **Unique ERC-7201 namespace ids**: No two contracts in the inheritance chain share the same namespace id. Colliding ids map to the same storage slot, causing silent storage corruption.
