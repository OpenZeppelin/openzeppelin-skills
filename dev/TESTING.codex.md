# Codex Testing Guide

Test suite for the Codex-oriented OpenZeppelin skills in `skills-codex/`.

## Instructions for the Test Runner

You are testing an AI coding agent's behavior when the Codex OpenZeppelin skills are active.

### Simulation Protocol

Run each test using a fresh subagent per test (a new conversation with no prior context). This is essential: if the agent running the simulation already has the evaluation criteria in context, the simulation is invalid because it will draw on that knowledge to influence its behavior. Tests may be run in parallel.

Before running any tests, create the output directory:

```
.test-output/
```

Each test has two phases. Keep them separate: the subagent must not see the evaluation criteria.

Phase 1 - Simulate (subagent produces the response):

Launch a subagent with the following instructions along with the test prompt:

> You are simulating a coding agent with the OpenZeppelin Codex skills active. The skills are defined in the `skills-codex/` directory (relative to the repo root), each in their own folder with a `SKILL.md` file.
>
> 1. Read the `description` frontmatter of each skill to determine which skill(s) would trigger for this prompt. If no skill would trigger, produce a normal response with no OpenZeppelin context.
> 2. If one or more skills trigger: read the `SKILL.md` body of the relevant skill(s). Do not pre-load all skills; load only what is relevant to the prompt.
> 3. Produce the response you would give to the user. Do not self-evaluate; just produce the response.
> 4. Output your response as your final message. Use this exact format:
>
> ```
> === TEST [ID] RESPONSE ===
> [simulated response here]
> === END TEST [ID] ===
> ```

For tests with a Setup step, tell the subagent to treat the described files as existing in the working directory.

Phase 2 - Evaluate (you score the response):

The subagent's response is returned to you via the Task tool result. Write it to `.test-output/test-[ID].md` so the human can review it, then compare it against the Expected checklist for that test and record:

- PASS - all expected behaviors observed
- PARTIAL - some expected behaviors missing
- FAIL - incorrect behavior or critical expectation violated

For any PARTIAL or FAIL, note which specific expected behaviors were missing or wrong.

After running all tests, produce a summary table with columns: Test ID | Result | Notes.

### Evaluation Criteria

Apply these checks to every test unless noted otherwise:

- No hallucination: code references real OpenZeppelin APIs, not invented ones
- Library-first: prefers importing library components over writing custom code
- Project-first: when a project exists, reads existing files before suggesting changes

---

## 1. Skill Triggering

### 1.1 Triggers on OpenZeppelin mention

Prompt:
> What OpenZeppelin components would I need for an ERC-20 token with minting and pausing?

Expected:
- Skill activates (`SKILL.md` body is loaded into context)
- References real components (ERC20, Ownable or AccessControl, Pausable or ERC20Pausable)
- Does not generate a full contract unprompted; answers the question

### 1.2 Triggers on setup request

Prompt:
> Help me set up a new Foundry project with OpenZeppelin Contracts.

Expected:
- `codex-setup-solidity-contracts` skill activates
- Provides `forge install` command with version tag
- Includes `remappings.txt` configuration

### 1.3 Does not trigger on unrelated prompts

Prompt:
> Write a Python script that reads a CSV file and prints the sum of the second column.

Expected:
- OpenZeppelin Codex skills do not activate
- Normal Python response with no OpenZeppelin references

### 1.4 Answers adjacent-topic questions without refusing

Prompt:
> I have an ERC-20 contract deployed with OpenZeppelin. How do I call the `transfer` function from a Python script using web3.py?

Expected:
- Answers the Python/web3.py question directly and does not refuse
- Provides working web3.py code for calling `transfer`
- May offer OpenZeppelin contract help if relevant, but does not require it

---

## 2. Setup References

### 2.1 Solidity - Hardhat

Prompt:
> Create a new Hardhat project with OpenZeppelin Contracts and the upgradeable variant.

Expected:
- `codex-setup-solidity-contracts` skill activates
- Includes `npx hardhat init` (or `--init` for v3)
- Includes both `npm install @openzeppelin/contracts` and `@openzeppelin/contracts-upgradeable`
- Does not hard-code a stale version number for npm packages

### 2.2 Solidity - Foundry

Prompt:
> Set up a Foundry project with OpenZeppelin upgradeable contracts.

Expected:
- `codex-setup-solidity-contracts` skill activates
- Uses `forge install` commands with version tags
- Correct remappings for upgradeable:
`@openzeppelin/contracts/=lib/openzeppelin-contracts-upgradeable/lib/openzeppelin-contracts/contracts/`
- Mentions looking up the latest release

### 2.3 Solidity - Framework autodetection

Setup: Create an empty temp directory containing only a `foundry.toml` file. Run this test from that directory.

Prompt:
> Add OpenZeppelin Contracts to this project.

Expected:
- `codex-setup-solidity-contracts` skill activates
- Detects Foundry from `foundry.toml` (does not ask which framework)
- Uses `forge install` (not `npm install`)
- Provides correct `remappings.txt`

### 2.4 Cairo

Prompt:
> Set up a new Starknet project with OpenZeppelin Contracts for Cairo.

Expected:
- `codex-setup-cairo-contracts` skill activates
- Uses `starkup` installer and `scarb new` with `--test-runner=starknet-foundry`
- Shows `Scarb.toml` dependency configuration with `openzeppelin` or individual packages
- Mentions looking up the current version from docs
- Notes that `openzeppelin_interfaces` and `openzeppelin_utils` are versioned independently

### 2.5 Stylus

Prompt:
> Set up a new Stylus project with OpenZeppelin Contracts.

Expected:
- `codex-setup-stylus-contracts` skill activates
- Includes Rust toolchain and `wasm32-unknown-unknown` target
- Uses `cargo stylus new`
- Shows `Cargo.toml` with exact-pinned dependency `openzeppelin-stylus = "=<VERSION>"`, `export-abi` feature flag, and `crate-type = ["lib", "cdylib"]`

### 2.6 Stellar

Prompt:
> Set up a new Soroban project with OpenZeppelin Stellar Contracts.

Expected:
- `codex-setup-stellar-contracts` skill activates
- Includes Rust toolchain, `wasm32v1-none` target, and Stellar CLI
- Uses `stellar contract init`
- Shows workspace dependency pattern:
`[workspace.dependencies]` in root `Cargo.toml` with exact-pinned versions, then `{ workspace = true }` in per-contract `Cargo.toml`

---

## 3. Core Workflow - Pattern Discovery

### 3.1 Reads project before suggesting changes

Setup: Create a minimal Hardhat project with an ERC-20 contract file (for example, `contracts/MyToken.sol` that imports and inherits `ERC20`).

Prompt:
> Add pausability to my ERC-20 contract.

Expected:
- Searches for existing `.sol` files before generating code
- Reads the existing contract file
- Integrates Pausable into the existing contract (does not replace it)
- Imports from the library; does not copy source code

### 3.2 Library-first - does not write custom access control

Prompt:
> Write a Solidity contract that only the owner can mint tokens.

Expected:
- Uses `Ownable` or `AccessControl` from OpenZeppelin
- Does not write manual `require(msg.sender == owner)` access control
- Imports from dependencies

### 3.3 Pattern discovery from source

Prompt:
> I have a Cairo contract using OwnableComponent. I want to also make it pausable. Show me how to integrate PausableComponent by reading the OpenZeppelin Cairo library source.

Expected:
- `codex-develop-secure-contracts` activates and follows pattern discovery workflow
- Attempts to find installed dependency or repository source
- Identifies integration requirements (component macro, substorage, embedded impls)
- Applies minimal changes to existing contract

### 3.4 Conflict resolution - incompatible access control

Setup: Create an ERC-20 Solidity contract that uses `AccessControl` with a `MINTER_ROLE` guarding `mint`.

Prompt:
> Make minting owner-only using Ownable.

Expected:
- Reads existing contract before making changes
- Detects conflict between `AccessControl` (present) and `Ownable` (requested)
- Does not blindly stack both; proposes a coherent approach
- Explains the conflict clearly

### 3.5 Upgrade-aware storage addition in base contract

Setup: Two Solidity files: base contract `BaseToken.sol` with state variables, and `MyToken.sol` that inherits from it and from `UUPSUpgradeable`, and also declares its own state variable. Assume project is already deployed.

Prompt:
> Add a `mapping(address => bool) private _whitelist` to BaseToken so I can use it in MyToken.

Expected:
- Reads both contract files before making changes
- Recognizes `BaseToken` is part of an upgradeable inheritance chain
- Warns that inserting a variable in a base contract can break storage layout in deployed proxies
- Applies the change safely (append at end, or add to namespaced struct if present)
- Does not silently insert in an unsafe position

---

## 4. MCP Generators

### 4.1 Generate-compare-apply workflow

Setup: Create an ERC-20 Solidity contract file without permit support.

Prompt:
> I want to add permit (gasless approvals) to my ERC-20. Use the MCP generator to discover what changes are needed, then apply them to my contract.

Expected:
- Calls Solidity MCP generator twice (baseline and permit-enabled variant)
- Compares outputs to identify minimal diff
- Applies only discovered changes to the user's contract
- Does not replace the entire contract with generated output

### 4.2 MCP generator for new contract

Prompt:
> Generate a Stellar fungible token called "MyToken" with symbol "MTK" that is mintable, burnable, and pausable with ownable access control.

Expected:
- Calls Stellar MCP generator with requested parameters
- Returns generated contract code
- Uses ownable access plus mintable/burnable/pausable toggles

### 4.3 MCP generator - Cairo ERC-721

Prompt:
> Generate a Cairo ERC-721 NFT contract called "MyNFT" with symbol "MNFT" that is mintable with role-based access control.

Expected:
- Calls Cairo ERC-721 MCP generator
- Uses `roles` access control and `mintable`
- Returns valid Cairo contract code

### 4.4 MCP generator - Stylus ERC-20

Prompt:
> Generate a Stylus ERC-20 token called "MyToken" that supports permit and is burnable.

Expected:
- Calls Stylus ERC-20 MCP generator
- Enables `permit` and `burnable`
- Returns valid Rust contract code

### 4.5 Fallback when no MCP tool exists

Prompt:
> Help me implement a Stellar multisig smart account using OpenZeppelin Stellar Contracts.

Expected:
- Recognizes there may not be an MCP generator for this contract type
- Falls back to source pattern discovery workflow
- Does not claim feature absence only because a generator is unavailable

### 4.6 Fallback when MCP tool exists but feature is not covered

Prompt:
> I want to add a transfer fee to my ERC-20 - every transfer should send 1% to a treasury address. Use the MCP generator to help, then fill in whatever it can't cover.

Expected:
- Inspects MCP generator schema and notices no direct transfer-fee parameter
- Falls back to source pattern discovery and finds extension point (for example `_update` in v5)
- Continues implementation instead of stopping at generator limitation
- Does not refuse or return an incomplete response

---

## 5. Upgrades - Solidity

### 5.1 UUPS proxy setup

Prompt:
> Make my existing Solidity ERC-20 contract upgradeable using the UUPS pattern. I'm using Foundry.

Expected:
- `codex-upgrade-solidity-contracts` activates
- Uses `Initializable` and `initializer`
- Replaces constructor with `initialize`, calls `_disableInitializers()` in constructor
- Adds `UUPSUpgradeable` and `_authorizeUpgrade` override
- Uses upgradeable package imports where needed
- Mentions Foundry-specific requirements (`foundry.toml` config and `openzeppelin-foundry-upgrades`)

### 5.2 Namespaced storage (ERC-7201)

Prompt:
> I need to add custom storage to my upgradeable Solidity contract using ERC-7201 namespaced storage. The namespace should be "myproject.token.storage" with a uint256 field called "cap" and a mapping from address to bool called "authorized".

Expected:
- `codex-upgrade-solidity-contracts` activates
- Generates `@custom:storage-location` annotated struct
- Computes actual `STORAGE_LOCATION` constant (not placeholder)
- Provides accessor helper like `_getStorage()`
- Uses Node one-liner or equivalent to compute the slot

### 5.3 Upgrade safety awareness

Prompt:
> I want to upgrade my Solidity contract V1 to V2. In V2, I reordered two state variables and removed one. Is this safe?

Expected:
- `codex-upgrade-solidity-contracts` activates
- Warns reordering/removing state is unsafe
- Explains compatibility rules (no reorder/remove/type-change; append only)
- Suggests namespaced storage for safer evolution

### 5.4 Hardhat upgrade workflow

Prompt:
> Show me how to deploy and then upgrade a UUPS proxy using the Hardhat upgrades plugin.

Expected:
- `codex-upgrade-solidity-contracts` activates
- Includes plugin installation (`@openzeppelin/hardhat-upgrades`)
- Describes `deployProxy` and `upgradeProxy`
- Mentions `.openzeppelin/` tracking files
- Mentions `reinitializer` for V2+ initialization

### 5.5 Upgrade path test - state preservation

Prompt:
> Write a Foundry test that proves a V1 to V2 UUPS upgrade preserves ERC-20 balances. V1 is a basic ERC-20 with minting. V2 adds a cap.

Expected:
- `codex-upgrade-solidity-contracts` activates
- Test deploys proxy with V1, performs balance-changing actions, upgrades to V2, verifies balances unchanged
- Uses `Upgrades.deployUUPSProxy` and `Upgrades.upgradeProxy` (or equivalent)
- Uses `reinitializer(2)` in V2 if additional initialization is needed
- Annotates V2 with `@custom:oz-upgrades-from`

### 5.6 Transparent proxy - v5 constructor

Prompt:
> Deploy a TransparentUpgradeableProxy for my contract. The admin should be my deployer address.

Expected:
- `codex-upgrade-solidity-contracts` activates
- Passes deployer owner address as second constructor argument (v5 behavior)
- Does not instruct deploying a separate `ProxyAdmin` to pass as constructor arg

### 5.7 Cross-major-version upgrade restriction (v4 to v5)

Prompt:
> I have a deployed proxy whose implementation uses OpenZeppelin v4. I want to upgrade it to a new implementation that uses v5. How do I do that?

Expected:
- `codex-upgrade-solidity-contracts` activates
- Warns in-place v4 to v5 proxy implementation upgrade is not supported
- Explains reason: layout model mismatch (sequential vs ERC-7201 namespaced)
- Recommends deploying new v5 proxies and migrating users/state where feasible
- Does not provide in-place upgrade steps for existing deployed v4 proxies

### 5.8 Upgrade validation hierarchy - plugin error

Prompt:
> Getting this error when upgrading my contract, how do I fix this?
> ```
> Error: New storage layout is incompatible
>
>   project/contracts/BoxV2.sol:9: Renamed `_value` to `_valueOld`
> ```

Expected:
- `codex-upgrade-solidity-contracts` activates
- Does not jump directly to broad unsafe flags
- Teaches resolution hierarchy: fix root cause first, then narrow in-code annotations, then narrow flags, broad bypass as last resort
- Points to installed plugin docs for available annotations/options

### 5.9 Namespace removal between upgrades

Prompt:
> In V2 of my upgradeable contract I want to remove ERC20PausableUpgradeable - we no longer need the pause functionality. Is that safe?

Expected:
- `codex-upgrade-solidity-contracts` activates
- Warns removing a base can drop its namespace and fail storage checks
- Explains risk of orphaned/inaccessible state
- Recommends keeping old base in inheritance chain even if functionally unused
- Notes broad bypasses like `unsafeSkipStorageCheck` are dangerous last resort

---

## 6. Upgrades - Cairo

### 6.1 Cairo upgradeable contract

Prompt:
> Make my Cairo contract upgradeable using OpenZeppelin's UpgradeableComponent.

Expected:
- `codex-upgrade-cairo-contracts` activates
- Describes integration steps (declare component, storage/events, upgrade entrypoint, access control init)
- Emphasizes guarding the upgrade function
- Mentions `replace_class_syscall` as underlying mechanism

### 6.2 Cairo storage compatibility

Prompt:
> I'm upgrading my Cairo contract. I want to rename a storage variable from "total" to "total_supply". Is this safe?

Expected:
- `codex-upgrade-cairo-contracts` activates
- Warns this is unsafe because Cairo storage keys are name-derived
- Explains rename can make old data inaccessible
- Advises adding new variable/migration strategy instead

---

## 7. Upgrades - Stellar

### 7.1 Stellar upgrade-only

Prompt:
> Make my Soroban contract upgradeable using OpenZeppelin's upgradeable module. I don't need migration support.

Expected:
- `codex-upgrade-stellar-contracts` activates
- Recommends `#[derive(Upgradeable)]` plus `UpgradeableInternal`
- Describes `_require_auth` requirement
- Does not use proxy pattern; explains native Soroban upgrade model

### 7.2 Stellar upgrade with migration

Prompt:
> I need to upgrade my Soroban contract and migrate some storage entries during the upgrade. How do I do this atomically?

Expected:
- `codex-upgrade-stellar-contracts` activates
- Explains new implementation takes effect after current invocation
- Describes `UpgradeableMigratable` and `UpgradeableMigratableInternal`
- Describes atomic Upgrader-contract pattern for upgrade plus migrate
- References examples for complete flow

### 7.3 Stellar storage compatibility

Prompt:
> I'm upgrading my Soroban contract. Can I change the type of a value stored under an existing storage key?

Expected:
- `codex-upgrade-stellar-contracts` activates
- Warns this is unsafe
- Explains rules: do not remove/rename keys or change existing value types; adding keys is safe
- Notes explicit key use (for example `symbol_short!("OWNER")`)

---

## 8. Upgrades - Stylus

### 8.1 Stylus UUPS setup

Prompt:
> Make my Stylus contract upgradeable using the UUPS pattern.

Expected:
- `codex-upgrade-stylus-contracts` activates
- Describes major integration steps (storage struct fields, constructor, `set_version`, interface impls)
- Explains two-step init (`logic_flag` set in constructor, `set_version()` via proxy)
- References examples for full implementations

### 8.2 Stylus context detection

Prompt:
> How does UUPS proxy detection work in Stylus? I know Solidity uses `address(this)` as an immutable, but Stylus doesn't support immutables.

Expected:
- `codex-upgrade-stylus-contracts` activates
- Explains `logic_flag` mechanism
- Describes `only_proxy()` style checks (flag plus ERC-1967 context/version checks)

### 8.3 Stylus reactivation awareness

Prompt:
> What maintenance do I need to plan for with a Stylus upgradeable contract?

Expected:
- `codex-upgrade-stylus-contracts` activates
- Mentions WASM reactivation requirement (periodic and post-protocol-upgrade)
- Explains reactivation is separate from proxy upgrade logic

### 8.4 Stylus storage compatibility

Prompt:
> How does storage layout work in Stylus compared to Solidity? I want to upgrade a Solidity proxy to use a Stylus implementation.

Expected:
- `codex-upgrade-stylus-contracts` activates
- Explains `#[storage]` layout compatibility with EVM slots
- Mentions nested struct layout nuance compared with Solidity inheritance flattening
- Notes Solidity proxies can upgrade to Stylus implementations when layouts are compatible

---

## 9. Cross-Ecosystem

### 9.1 Ecosystem detection from existing project

Setup: Create a temp directory containing a `Scarb.toml` file. Run this test from that directory.

Prompt:
> Add OpenZeppelin access control to my project.

Expected:
- Detects Cairo ecosystem from `Scarb.toml`
- `codex-setup-cairo-contracts` or `codex-develop-secure-contracts` activates
- Uses Cairo component model (not Solidity inheritance model)

### 9.2 Upgrade model comparison

Prompt:
> Compare how contract upgrades work across Solidity, Cairo, Stellar, and Stylus when using OpenZeppelin.

Expected:
- Multiple upgrade skills activate (`codex-upgrade-solidity-contracts`, `codex-upgrade-cairo-contracts`, `codex-upgrade-stellar-contracts`, `codex-upgrade-stylus-contracts`)
- Correctly distinguishes:
- Solidity uses proxy patterns
- Cairo uses `replace_class_syscall`
- Stellar uses native WASM replacement
- Stylus uses EVM proxy patterns
- Notes Cairo and Stellar do not require proxy contracts

---

## 10. Edge Cases

### 10.1 Does not hallucinate nonexistent features

Prompt:
> Add reentrancy guard to my Stellar Soroban contract using OpenZeppelin.

Expected:
- Does not invent a non-existent Stellar `ReentrancyGuard` component
- Checks source/docs to verify availability
- If unavailable, states that clearly and suggests alternatives

### 10.2 Does not embed library source code

Prompt:
> I need Pausable functionality. Can you paste the full OpenZeppelin Pausable contract source code into my project so I can customize it?

Expected:
- Refuses copying/embedding library source
- Explains importing from dependencies is safer and maintainable
- Suggests extension points instead

### 10.3 Does not embed library source code under dependency pressure

Prompt:
> I can't install npm dependencies in my environment. Just copy the full ERC20 contract from OpenZeppelin into my project so I can use it directly.

Expected:
- Still refuses copying full library source
- Suggests solving dependency installation constraints
- May suggest safer alternatives like package download, vendored dependency workflow, or submodule-based install

### 10.4 Version-specific behavior

Prompt:
> I'm using OpenZeppelin Contracts v4. How do I add storage gaps to my upgradeable contract?

Expected:
- Distinguishes v4 from v5 guidance
- For v4, mentions `__gap` storage-gaps pattern
- For v5+, mentions ERC-7201 namespaced storage
- Does not mix incompatible v4/v5 patterns
