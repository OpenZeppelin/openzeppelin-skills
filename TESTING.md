# Testing Guide

Test suite for the `openzeppelin-contracts` skill and its references. The `openzeppelin-security` skill is not yet implemented (placeholder) and is not covered here.

## Instructions for the Test Runner

You are testing an AI coding agent's behavior when the `openzeppelin-contracts` skill is active.

### Simulation Protocol

Run each test **one at a time, using a fresh subagent per test** (a new conversation with no prior context). This is essential: if the agent running the simulation already has the evaluation criteria in context, the simulation is invalid — it will draw on that knowledge to influence its behavior.

**Before running any tests**, create the output directory:

```
.test-output/
```

For each test, the subagent performs two phases. Keep them separate.

**Phase 1 — Simulate (subagent produces the response):**

Send the subagent the following instructions along with the test prompt:

> You are simulating a coding agent with the `openzeppelin-contracts` skill active. The skill is defined at `skills/openzeppelin-contracts/SKILL.md` (relative to the repo root).
>
> 1. Read `SKILL.md`.
> 2. Determine whether the skill would trigger for this prompt — the trigger condition is in the frontmatter `description` field. If the skill would NOT trigger, produce a normal response with no OpenZeppelin context.
> 3. If the skill triggers: read whichever reference files `SKILL.md` instructs you to read for this type of request (setup, upgrades, patterns, etc.). Do not pre-load all references — load only what the skill says is relevant to the prompt.
> 4. Produce the response you would give to the user. Do NOT self-evaluate — just produce the response.
> 5. Write your response to `.test-output/test-[ID].md` using the Write tool, where `[ID]` is the test ID (e.g., `1.1`, `3.4`). Use this exact format:
>
> ```
> === TEST [ID] RESPONSE ===
> [simulated response here]
> === END TEST [ID] ===
> ```

For tests with a **Setup** step, tell the subagent to treat the described files as existing in the working directory.

**Phase 2 — Evaluate (you score the response):**

Read the subagent's output from `test-output/test-[ID].md`, compare it against the **Expected** checklist for that test, and record:
- **PASS** — all expected behaviors observed
- **PARTIAL** — some expected behaviors missing
- **FAIL** — incorrect behavior or critical expectation violated

For any PARTIAL or FAIL, note which specific expected behaviors were missing or wrong.

After running all tests, produce a summary table with columns: Test ID | Result | Notes.

### Evaluation Criteria

Apply these checks to every test unless noted otherwise:

- **No hallucination**: Code references real OpenZeppelin APIs, not invented ones
- **Library-first**: Prefers importing library components over writing custom code
- **Project-first**: When a project exists, reads existing files before suggesting changes

---

## 1. Skill Triggering

### 1.1 Triggers on OpenZeppelin mention

**Prompt:**
> What OpenZeppelin components would I need for an ERC-20 token with minting and pausing?

**Expected:**
- Skill activates (SKILL.md body is loaded into context)
- References real components (ERC20, Ownable/AccessControl, Pausable or ERC20Pausable)
- Does not generate a full contract unprompted — answers the question

### 1.2 Triggers on setup request

**Prompt:**
> Help me set up a new Foundry project with OpenZeppelin Contracts.

**Expected:**
- Skill activates
- Reads `references/setup-solidity.md`
- Provides `forge install` command with version tag
- Includes remappings.txt configuration

### 1.3 Does not trigger on unrelated prompts

**Prompt:**
> Write a Python script that reads a CSV file and prints the sum of the second column.

**Expected:**
- Skill does NOT activate
- Normal Python response with no OpenZeppelin references

### 1.4 Answers adjacent-topic questions without refusing

**Prompt:**
> I have an ERC-20 contract deployed with OpenZeppelin. How do I call the `transfer` function from a Python script using web3.py?

**Expected:**
- Answers the Python/web3.py question directly — does NOT refuse
- Provides working web3.py code for calling `transfer`
- May offer to help with the OpenZeppelin contract side if relevant, but does not require it

---

## 2. Setup References

### 2.1 Solidity — Hardhat

**Prompt:**
> Create a new Hardhat project with OpenZeppelin Contracts and the upgradeable variant.

**Expected:**
- Reads `references/setup-solidity.md`
- Includes `npx hardhat init` (or `--init` for v3)
- Includes both `npm install @openzeppelin/contracts` and `@openzeppelin/contracts-upgradeable`
- Does NOT hard-code a stale version number for npm packages

### 2.2 Solidity — Foundry

**Prompt:**
> Set up a Foundry project with OpenZeppelin upgradeable contracts.

**Expected:**
- Reads `references/setup-solidity.md`
- `forge install` commands with version tags
- Correct remappings for upgradeable — `@openzeppelin/contracts/` must route through the upgradeable submodule: `@openzeppelin/contracts/=lib/openzeppelin-contracts-upgradeable/lib/openzeppelin-contracts/contracts/`
- Mentions looking up the latest release

### 2.3 Solidity — Framework autodetection

**Setup:** Create an empty temp directory containing only a `foundry.toml` file. Run this test from that directory.

**Prompt:**
> Add OpenZeppelin Contracts to this project.

**Expected:**
- Detects Foundry from `foundry.toml` (does not ask which framework)
- Uses `forge install` (not `npm install`)
- Provides correct `remappings.txt`

### 2.4 Cairo

**Prompt:**
> Set up a new Starknet project with OpenZeppelin Contracts for Cairo.

**Expected:**
- Reads `references/setup-cairo.md`
- Uses `starkup` installer and `scarb new` with `--test-runner=starknet-foundry`
- Shows `Scarb.toml` dependency configuration with `openzeppelin` or individual packages
- Mentions looking up the current version from docs
- Notes that `openzeppelin_interfaces` and `openzeppelin_utils` are versioned independently

### 2.5 Stylus

**Prompt:**
> Set up a new Stylus project with OpenZeppelin Contracts.

**Expected:**
- Reads `references/setup-stylus.md`
- Includes Rust toolchain + `wasm32-unknown-unknown` target
- `cargo stylus new`
- `Cargo.toml` with exact-pinned dependency `openzeppelin-stylus = "=<VERSION>"`, `export-abi` feature flag, and `crate-type = ["lib", "cdylib"]`

### 2.6 Stellar

**Prompt:**
> Set up a new Soroban project with OpenZeppelin Stellar Contracts.

**Expected:**
- Reads `references/setup-stellar.md`
- Includes Rust toolchain + `wasm32v1-none` target + Stellar CLI
- `stellar contract init`
- Workspace dependency pattern: `[workspace.dependencies]` in root `Cargo.toml` with exact-pinned versions (`"=<VERSION>"`), then `{ workspace = true }` in per-contract `Cargo.toml`

---

## 3. Core Workflow — Pattern Discovery

### 3.1 Reads project before suggesting changes

**Setup:** Create a minimal Hardhat project with an ERC-20 contract file (e.g., `contracts/MyToken.sol` that imports and inherits `ERC20`).

**Prompt:**
> Add pausability to my ERC-20 contract.

**Expected:**
- Searches for existing `.sol` files before generating code
- Reads the existing contract file
- Integrates Pausable into the existing contract (does not replace it)
- Imports from the library, does not copy source code

### 3.2 Library-first — does not write custom access control

**Prompt:**
> Write a Solidity contract that only the owner can mint tokens.

**Expected:**
- Uses `Ownable` or `AccessControl` from OpenZeppelin
- Does NOT write `require(msg.sender == owner)` manually
- Imports from the dependency

### 3.3 Pattern discovery from source

**Prompt:**
> I have a Cairo contract using OwnableComponent. I want to also make it pausable. Show me how to integrate PausableComponent by reading the OpenZeppelin Cairo library source.

**Expected:**
- Reads `references/patterns.md` or follows its methodology
- Attempts to find the installed dependency or repository source
- Identifies integration requirements (component macro, substorage, embedded impls)
- Applies minimal changes to the existing contract

### 3.4 Conflict resolution — incompatible access control

**Setup:** Create an ERC-20 Solidity contract that uses `AccessControl` with a `MINTER_ROLE` guarding `mint`.

**Prompt:**
> Make minting owner-only using Ownable.

**Expected:**
- Reads the existing contract file before making changes
- Detects the conflict between `AccessControl` (already present) and `Ownable` (requested)
- Does NOT blindly stack both — proposes a coherent approach (e.g., migrate to Ownable, or keep AccessControl and adjust roles)
- Explains the conflict to the user

---

## 4. MCP Generators

### 4.1 Generate-compare-apply workflow

**Setup:** Create an ERC-20 Solidity contract file without permit support.

**Prompt:**
> I want to add permit (gasless approvals) to my ERC-20. Use the MCP generator to discover what changes are needed, then apply them to my contract.

**Expected:**
- Calls the Solidity MCP generator twice (baseline without permit, then with permit enabled)
- Compares the two outputs to identify the diff
- Applies only the discovered changes to the user's contract
- Does NOT replace the entire contract with generated output

### 4.2 MCP generator for new contract

**Prompt:**
> Generate a Stellar fungible token called "MyToken" with symbol "MTK" that is mintable, burnable, and pausable with ownable access control.

**Expected:**
- Calls the Stellar MCP generator with the specified parameters
- Returns generated contract code
- Uses `ownable` access, `mintable`, `burnable`, `pausable` flags

### 4.3 MCP generator — Cairo ERC-721

**Prompt:**
> Generate a Cairo ERC-721 NFT contract called "MyNFT" with symbol "MNFT" that is mintable with role-based access control.

**Expected:**
- Calls the Cairo ERC-721 MCP generator
- Uses `roles` access control, `mintable` enabled
- Returns valid Cairo contract code

### 4.4 MCP generator — Stylus ERC-20

**Prompt:**
> Generate a Stylus ERC-20 token called "MyToken" that supports permit and is burnable.

**Expected:**
- Calls the Stylus ERC-20 MCP generator
- Enables `permit` and `burnable`
- Returns valid Rust contract code

### 4.5 Fallback when no MCP tool exists

**Prompt:**
> Help me implement a Stellar multisig smart account using OpenZeppelin Stellar Contracts.

**Expected:**
- Recognizes there may not be an MCP generator for this specific contract type
- Falls back to pattern discovery methodology (reading library source or repository)
- Does NOT claim the feature doesn't exist just because there's no generator

### 4.6 Fallback when MCP tool exists but feature is not covered

**Prompt:**
> I want to add a transfer fee to my ERC-20 — every transfer should send 1% to a treasury address. Use the MCP generator to help, then fill in whatever it can't cover.

**Expected:**
- Inspects the MCP generator schema and recognizes there is no `transferFee` parameter (schema inspection is sufficient — calling the generator is not required when the feature is clearly absent from the parameter list)
- Falls back to pattern discovery: searches the installed library source for a hook or extension point (e.g., `_update` override in v5)
- Does NOT stop after noting the generator can't handle it — applies the discovered pattern directly
- Does NOT refuse or produce an incomplete response just because the generator lacks the feature

---

## 5. Upgrades — Solidity

### 5.1 UUPS proxy setup

**Prompt:**
> Make my existing Solidity ERC-20 contract upgradeable using the UUPS pattern. I'm using Foundry.

**Expected:**
- Reads `references/upgrades-solidity.md`
- Adds `Initializable`, uses `initializer` modifier
- Replaces constructor with `initialize` function, calls `_disableInitializers()` in constructor
- Adds `UUPSUpgradeable` with `_authorizeUpgrade` override
- Switches imports to `@openzeppelin/contracts-upgradeable`
- Mentions Foundry-specific setup (`foundry.toml` config, `openzeppelin-foundry-upgrades`)

### 5.2 Namespaced storage (ERC-7201)

**Prompt:**
> I need to add custom storage to my upgradeable Solidity contract using ERC-7201 namespaced storage. The namespace should be "myproject.token.storage" with a uint256 field called "cap" and a mapping from address to bool called "authorized".

**Expected:**
- Reads `references/upgrades-solidity.md`
- Generates the `@custom:storage-location` annotated struct
- Computes the actual `STORAGE_LOCATION` constant (not a placeholder `0x...`)
- Provides the `_getStorage()` accessor function
- Uses the Node.js one-liner or equivalent to compute the hash

### 5.3 Upgrade safety awareness

**Prompt:**
> I want to upgrade my Solidity contract V1 to V2. In V2, I reordered two state variables and removed one. Is this safe?

**Expected:**
- Reads `references/upgrades-solidity.md`
- Warns that reordering and removing state variables breaks storage compatibility
- Explains the rules (never reorder, remove, or change types; only append)
- Suggests using namespaced storage for better safety

### 5.4 Hardhat upgrade workflow

**Prompt:**
> Show me how to deploy and then upgrade a UUPS proxy using the Hardhat upgrades plugin.

**Expected:**
- Reads `references/upgrades-solidity.md`
- Includes plugin installation (`@openzeppelin/hardhat-upgrades`)
- Describes `deployProxy` / `upgradeProxy` workflow
- Mentions `.openzeppelin/` tracking files
- References the `reinitializer` modifier for V2+ initialization

### 5.5 Upgrade path test — state preservation

**Prompt:**
> Write a Foundry test that proves a V1 to V2 UUPS upgrade preserves ERC-20 balances. V1 is a basic ERC-20 with minting. V2 adds a cap.

**Expected:**
- Reads `references/upgrades-solidity.md`
- Produces a test that: deploys proxy with V1, mints/transfers tokens, upgrades to V2, then asserts balances are unchanged
- Uses `Upgrades.deployUUPSProxy` and `Upgrades.upgradeProxy` (or equivalent)
- V2 uses `reinitializer(2)` if it needs new initialization logic
- Annotates V2 with `@custom:oz-upgrades-from`

### 5.6 Transparent proxy — v5 constructor

**Prompt:**
> Deploy a TransparentUpgradeableProxy for my contract. The admin should be my deployer address.

**Expected:**
- Reads `references/upgrades-solidity.md`
- Passes the deployer address (owner) as the second constructor argument to `TransparentUpgradeableProxy` — NOT a `ProxyAdmin` contract address
- Does NOT instruct the user to deploy a `ProxyAdmin` separately and pass it in

### 5.7 Cross-major-version upgrade restriction (v4 → v5)

**Prompt:**
> I have a deployed proxy whose implementation uses OpenZeppelin v4. I want to upgrade it to a new implementation that uses v5. How do I do that?

**Expected:**
- Reads `references/upgrades-solidity.md`
- Warns that upgrading a proxy from a v4 implementation to a v5 implementation is not supported
- Explains the cause: v4 uses sequential storage, v5 uses namespaced storage (ERC-7201), making the layouts incompatible
- Recommends deploying new proxies with v5 implementations and migrating users to the new address instead
- Does NOT provide steps for an in-place upgrade from v4 to v5
- Clarifies that migrating the codebase to v5 is encouraged, as long as the existing proxies are not upgraded in place

---

## 6. Upgrades — Cairo

### 6.1 Cairo upgradeable contract

**Prompt:**
> Make my Cairo contract upgradeable using OpenZeppelin's UpgradeableComponent.

**Expected:**
- Reads `references/upgrades-cairo.md`
- Describes the 4 integration steps (declare component, add to storage/events, expose upgrade function, initialize access control)
- Emphasizes guarding the upgrade function with access control
- Mentions `replace_class_syscall` as the underlying mechanism

### 6.2 Cairo storage compatibility

**Prompt:**
> I'm upgrading my Cairo contract. I want to rename a storage variable from "total" to "total_supply". Is this safe?

**Expected:**
- Reads `references/upgrades-cairo.md`
- Warns this is NOT safe — Cairo storage slots are derived from variable names via `sn_keccak`
- Renaming makes old data inaccessible
- Advises adding a new variable instead

---

## 7. Upgrades — Stellar

### 7.1 Stellar upgrade-only

**Prompt:**
> Make my Soroban contract upgradeable using OpenZeppelin's upgradeable module. I don't need migration support.

**Expected:**
- Reads `references/upgrades-stellar.md`
- Recommends `#[derive(Upgradeable)]` and implementing `UpgradeableInternal`
- Describes the `_require_auth` method requirement
- Does NOT use a proxy pattern — explains Soroban's native upgrade model

### 7.2 Stellar upgrade with migration

**Prompt:**
> I need to upgrade my Soroban contract and migrate some storage entries during the upgrade. How do I do this atomically?

**Expected:**
- Reads `references/upgrades-stellar.md`
- Explains that the new implementation takes effect after the current invocation completes
- Describes the `UpgradeableMigratable` derive macro and `UpgradeableMigratableInternal` trait
- Describes the atomic `Upgrader` contract pattern for wrapping upgrade + migrate
- References the `examples/` directory for full integration examples

### 7.3 Stellar storage compatibility

**Prompt:**
> I'm upgrading my Soroban contract. Can I change the type of a value stored under an existing storage key?

**Expected:**
- Reads `references/upgrades-stellar.md`
- Warns this is NOT safe — incompatible changes corrupt state
- Explains the rules: don't remove/rename keys, don't change types, adding new keys is safe
- Notes that Soroban uses explicit string keys (e.g., `symbol_short!("OWNER")`)

---

## 8. Upgrades — Stylus

### 8.1 Stylus UUPS setup

**Prompt:**
> Make my Stylus contract upgradeable using the UUPS pattern.

**Expected:**
- Reads `references/upgrades-stylus.md`
- Describes the 5 integration steps (storage struct, constructor, initialize/set_version, IUUPSUpgradeable, IErc1822Proxiable)
- Explains the two-step initialization (constructor sets `logic_flag`, `set_version()` via proxy)
- References the `examples/` directory for full integration examples

### 8.2 Stylus context detection

**Prompt:**
> How does UUPS proxy detection work in Stylus? I know Solidity uses `address(this)` as an immutable, but Stylus doesn't support immutables.

**Expected:**
- Reads `references/upgrades-stylus.md`
- Explains the `logic_flag` mechanism (set in constructor, reads as false via delegatecall)
- Describes `only_proxy()` checks (flag + ERC-1967 slot + version match)

### 8.3 Stylus reactivation awareness

**Prompt:**
> What maintenance do I need to plan for with a Stylus upgradeable contract?

**Expected:**
- Reads `references/upgrades-stylus.md`
- Mentions WASM reactivation requirement (365 days or after protocol upgrade)
- Explains this is orthogonal to proxy upgrades but must be factored into planning

### 8.4 Stylus storage compatibility

**Prompt:**
> How does storage layout work in Stylus compared to Solidity? I want to upgrade a Solidity proxy to use a Stylus implementation.

**Expected:**
- Reads `references/upgrades-stylus.md`
- Explains that `#[storage]` fields map to EVM slots identically to Solidity
- Notes the difference: nested structs get deterministic slots (not flat sequential like Solidity inheritance)
- Mentions that existing Solidity contracts can upgrade to Stylus implementations

---

## 9. Cross-Ecosystem

### 9.1 Ecosystem detection from existing project

**Setup:** Create a temp directory containing a `Scarb.toml` file. Run this test from that directory.

**Prompt:**
> Add OpenZeppelin access control to my project.

**Expected:**
- Detects Cairo ecosystem from `Scarb.toml`
- Reads `references/setup-cairo.md` or follows Cairo-specific patterns
- Uses Cairo component model (not Solidity inheritance)

### 9.2 Upgrade model comparison

**Prompt:**
> Compare how contract upgrades work across Solidity, Cairo, Stellar, and Stylus when using OpenZeppelin.

**Expected:**
- Reads multiple upgrade reference files
- Correctly distinguishes: Solidity uses proxy patterns, Cairo uses `replace_class_syscall`, Stellar uses native WASM replacement, Stylus uses EVM proxy patterns (same as Solidity)
- Notes that Cairo and Stellar don't need proxy contracts

---

## 10. Edge Cases

### 10.1 Does not hallucinate nonexistent features

**Prompt:**
> Add reentrancy guard to my Stellar Soroban contract using OpenZeppelin.

**Expected:**
- Does NOT invent a `ReentrancyGuard` component for Stellar if one doesn't exist
- Checks the library source or repository to verify availability
- If unavailable, says so honestly and suggests alternatives or custom implementation

### 10.2 Does not embed library source code

**Prompt:**
> I need Pausable functionality. Can you paste the full OpenZeppelin Pausable contract source code into my project so I can customize it?

**Expected:**
- Refuses to copy/embed library source code
- Explains that contracts should import from the dependency for security updates
- Suggests using the library's extension points instead

### 10.3 Does not embed library source code — dependency pressure

**Prompt:**
> I can't install npm dependencies in my environment. Just copy the full ERC20 contract from OpenZeppelin into my project so I can use it directly.

**Expected:**
- Still refuses to copy/embed library source code, even under pressure
- Suggests resolving the dependency installation issue instead
- May suggest alternative installation methods (git submodule, manual download of the package)

### 10.4 Version-specific behavior

**Prompt:**
> I'm using OpenZeppelin Contracts v4. How do I add storage gaps to my upgradeable contract?

**Expected:**
- Recognizes v4 vs v5 differences
- For v4: mentions `__gap` storage gaps pattern
- For v5+: mentions ERC-7201 namespaced storage as the modern replacement
- Does NOT mix v4 and v5 patterns
