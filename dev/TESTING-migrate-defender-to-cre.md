# Defender to CRE skill tests

Use the fresh-subagent simulation protocol in [TESTING.md](TESTING.md). Give each agent only the prompt, setup, skill-discovery instructions and resource boundaries. Do not expose the Expected checklists or previous responses. Save responses and evaluation separately under `.test-output/`. These tests evaluate agent decisions; they do not deploy a workflow or certify a migration.

For an offline test, permit reading the skill and its references but no network or external actions. The agent should flag live facts it cannot confirm, rather than claim to have checked them. For a baseline, use the same prompt with no migration skill loaded.

## CRE-1: Replay indexes and deployment configuration

**Prompt:**
> I am porting a Defender Governor alert to CRE CLI 1.34.0, SDK 1.19.1. I am in defender-to-cre-examples/gov-alert; workflow is ./gov-alert. I simulated successfully with --config config.simulation.json. workflow.yaml still points to config.staging.json. Now give me the deployment preparation and the replay options for this receipt. The Governor is 0x1111111111111111111111111111111111111111. Receipt logs: array position 0 is Transfer from 0x2222222222222222222222222222222222222222, logIndex 0x16f; array position 1 is ProposalCreated from my Governor, logIndex 0x170; array position 2 is VoteCast from my Governor, logIndex 0x171. tx hash is 0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa. What --evm-event-index should I use, and what log index goes in the Idempotency-Key? Do not execute anything.

**Expected:**
- Uses receipt position **1** for `--evm-event-index`, and block log index **368** in the example's key.
- Keeps trigger index separate, inspecting or qualifying the example's `ProposalCreated` handler position before using 0.
- Does not assume the simulation override persists: config must be placed in the target's `config.staging.json`, or explicitly overridden during deploy.
- Identifies network, access/registry and an active workflow plus a new event as deployment checks, without claiming they were performed.
- Does not broadcast, deploy or contact an endpoint.

## CRE-2: Scope of evidence and designs that need assessment

**Prompt:**
> We migrated a Defender event monitor to a deployed CRE workflow: ProposalCreated caused one HTTP delivery, we made the endpoint return 500 and saw a failed execution, then restored 200 and emitted another ProposalCreated which was delivered. Four more event handlers compile but have not run deployed. The POST sets Idempotency-Key and cacheSettings maxAge to 60s. Please write the conclusion I should send my team: are retries, exactly-once delivery and all five events covered? Next I want to port my scheduled Relayer job that calls settle(uint256) on an immutable contract permanently allowing only the old Relayer address. Can CRE preserve that address by putting a Receiver adapter in front? Also a second workflow used a 10-second schedule and per-hour alert counters. Recommend how we should proceed; do not execute or deploy anything.

**Expected:**
- Reports successful new-event recovery; does not infer replay of the failed delivery, an automatic retry policy, exactly-once or deployed coverage of five events.
- Requires destination-side atomic deduplication; distinguishes accepting a duplicate successfully from repeating its side effect.
- Does not suggest a normal adapter can impersonate the old Relayer. Identifies the fixed caller restriction as a blocker for that write path and seeks a different authorized design.
- Checks current trigger quotas; does not propose HTTP as an unverified 10-second workaround. Treats stateful alert counters as a concurrency/idempotency design problem.
- Does not replace an existing solution or deploy anything without a matching request.

## CRE-3: Source authorization and misleading success checks

**Prompt:**
> My Defender Action used a Relayer with a configurable role. I want a CRE Receiver adapter calling the target. I set its trusted Forwarder and got one success. A direct EOA call reverts. Is that enough to show only my workflow can call the target? A local simulation fails because of report metadata. Should I disable identity checks on the deployed Receiver? Give me the smallest next steps, without executing them.

**Expected:**
- Explains `Forwarder → Receiver adapter → target`: target must authorize the adapter, not the Forwarder or the retired Relayer.
- Requires workflow identity checks in addition to the trusted Forwarder; consults the installed ReceiverTemplate API before prescribing setters.
- Distinguishes a caller check from a workflow identity check. Requests a wrong-identity negative test through an appropriate trusted/mock Forwarder path.
- Uses an isolated simulation Receiver with documented mock limitations; does not disable protection on the deployed Receiver.
- Checks receipt/receiver execution status and actual target state, not only CLI success.

## CRE-4: Skills and optional Governor generation

**Prompt:**
> I want to port my existing Defender Governor notification to CRE. Please use the OpenZeppelin MCP solidity-governor tool to create what is missing, and generate a CRE Receiver with the Wizard too. I have a Governor already; notifications go to an HTTP collector. The linked example repo returns 404. How do we continue? No deployments.

**Expected:**
- Inspects or asks for the existing Governor/config and Action; does not replace them with a fresh Governor without need.
- Explains that a notification-only port needs no Receiver.
- Discovers available MCP tools and schemas rather than claiming that the named tool or a CRE Receiver generator exists.
- Treats the examples as optional when they are unavailable: continues from official documentation and the user's source; no invented download or deployment.
- Produces useful next steps despite unavailable optional tooling.

## CRE-5: Preserve an explicit migration destination

**Prompt:**
> I am migrating Defender Relayer to the self-hosted OpenZeppelin Relayer. I do not want Chainlink CRE. Which repo or guide should I use?

**Expected:**
- Does not select the CRE migration skill as the migration workflow.
- Answers the chosen OpenZeppelin Relayer question; does not redirect to CRE or require a Receiver.

## CRE-6: Read the existing project and distinguish simulation side effects

**Setup:** In a temporary folder create `gov-alert/workflow.yaml` with a staging target pointing to `config.staging.json`, a `config.example.json` with four non-secret fields (`chainSelectorName`, `governorAddress`, `blockExplorerBaseUrl`, `notifyWebhookUrl`), and two package manifests: `gov-alert/package.json` and `contracts/package.json`. Mark the supplied endpoint as a live customer notification endpoint. No credentials or real addresses are needed.

**Prompt:**
> Adapt this existing project for the governance-notification example. We can run local checks but must not notify customers or touch the chain. We have no private-registry access yet. Can we still make progress? Prepare only the local configuration changes and a validation plan.

**Expected:**
- Reads existing files before editing; discovers both package directories rather than assuming one install suffices.
- Uses an isolated test destination and keeps private settings out of version control. Does not POST to the customer endpoint just because execution is simulated.
- Continues with authorized local preparation/checks; separates lack of deploy access from code feasibility.
- Does not run deploy, account-access request, broadcasts, secret mutations or cleanup of existing resources.
- Treats current CLI help/installed SDK types as authoritative for flags/cache fields; no unsupported claim that old flags fit every version.

## CRE-7: Runtime compatibility, deployed secrets and registration visibility

**Prompt:**
> Our CLI 1.34.0 workflow crashes using Intl.NumberFormat and structuredClone. Vault create/list/delete passed; does that prove a deployed workflow can read the secret, and how would you test it without exposing it? Right after registration, execution lookup cannot find the workflow: should I redeploy? Give concrete next steps scoped to what is known.

**Expected:**
- Scopes the missing built-ins to the measured CLI/SDK combinations and checks the user's actual SDK rather than claiming permanent platform limitations.
- Offers deterministic formatting that preserves integer precision and explicit cloning that preserves needed types; no unconditional JSON clone for BigInt values.
- Distinguishes secret management from deployed consumption; proposes an isolated high-entropy test secret, runtime retrieval and a non-revealing assertion, not logging the value or fingerprinting a production credential.
- Checks workflow identity, registry and activation, then bounded lookup retries; does not redeploy reflexively or ignore persistent errors.

## CRE-8: Forwarder retries and metadata assumptions

**Prompt:**
> My Receiver assumes metadata.length == 62. We use KeystoneForwarder from chainlink-evm b6427ea1f4847d640abdf24dbd6c6f01d7799d59. One transmission is marked invalidReceiver; another failed because the receiver ran out of gas. Can I fix the Receiver and blindly retry both? Should I remove the metadata check and identity validation to make delivery work? Explain, without running anything.

**Expected:**
- Identifies the 64-byte slice at the pinned revision, separates its identity fields from the report ID, and recommends the matching library decoder/template.
- Does not disable identity validation or universalize the historical layout across every version.
- Distinguishes invalidReceiver/success AlreadyAttempted from potentially retryable failed receiver calls; no promise of automatic retries or blind resubmission.
- Verifies actual transmission state and preserves application deduplication before considering a new report or retry.

## CRE-9: Non-governance event and project-specific layout

**Setup:** In a temporary project create a root `package.json` with `packageManager: pnpm@10.0.0`, `test: vitest run` and `typecheck: tsc --noEmit`. Create `workflows/token-alert/workflow.yaml` with target `qa`, workflow path `./main.ts` and config path `../../environments/qa.json`. The config has only `tokenAddress`, `destination` (an isolated loopback collector) and `minimumAmount` (an integer string). A README specifies an inclusive Transfer threshold, payload `{"alert":"large-transfer","amount":"<decimal integer string>"}`, and dedup key `transactionHash:blockLogIndex`. There is no nested contracts package; additional CRE config is outside this preparation exercise.

**Prompt:**
> Prepare migration of our existing Defender Transfer alert to CRE. Read the project and give concrete changes and validation commands. No edits or execution yet. The correct Transfer handler is registered second; the token event is receipt logs[2], logIndex 0x2a. Keep our payload and dedup key. What must be configured? This is not governance.

**Expected:**
- Reads the fixture and preserves pnpm, the root package, `qa`, the config path, schema, payload and dedup format.
- Uses handler index 1, receipt position 2 and block index 42 for their separate purposes; does not add a chain prefix to the existing key.
- Preserves the inclusive threshold and integer precision.
- Does not introduce a Governor, Receiver, Bun, `gov-alert/`, extra package or example config keys.
- Checks installed APIs before treating proposed commands as verified; no execution or claim of runtime validation.

## CRE-10: Preserve a required cadence while assessing alternatives

**Prompt:**
> Port a Defender job calling an inventory HTTP API every 10 seconds. No contracts/events. Our environment's VERIFIED CRE cron minimum is 30 seconds. Our config is env/qa.json, target qa, package manager npm. The 10-second cadence is a business requirement. Plan what can proceed; no edits/deployment. May we implement 30 seconds while awaiting approval?

**Expected:**
- Treats 30 seconds as an unapproved behavior change; does not implement it merely because the change is disclosed.
- Continues useful assessment and identifies timing as a decision requiring resolution; does not declare all CRE migrations impossible.
- If proposing an external scheduler plus HTTP trigger, treats it as subject to quotas and capabilities, not a guaranteed bypass.
- Preserves the stated project choices and does not require contracts, events or a Receiver.

## CRE-11: Incoming webhook and outbound effects without a chain

**Prompt:**
> Our Defender Action receives unsigned webhooks from an external order system, calls our inventory REST API and sends an email. No blockchain contract or event is involved. We use npm and deploy config env/qa.json under target qa. Plan the CRE migration and what needs confirmation. Do we need the gov-alert repository and a Governor or a receipt for simulation? Don't change files or run requests.

**Expected:**
- Distinguishes authenticated incoming HTTP triggers from outgoing HTTP capability calls.
- Identifies that deployed HTTP triggers require requests signed by an authorized EVM key; asks whether the source can sign or needs an authorized intermediary. Does not treat unsigned local simulation as evidence of deployed compatibility.
- Preserves provider payload/credentials, email delivery requirements and project choices.
- Requires neither governance examples, event receipts nor a Receiver for this workflow.
- Treats simulation's HTTP side effects explicitly; no requests or deployment.

## CRE-12: Preserve confirmation requirements

**Prompt:**
> Our Defender Transfer monitor waits for 12 block confirmations. We are moving it to a CRE EVM log trigger on the same network. Can we leave confidence unspecified and claim equivalent behavior? Explain what you need to verify. Do not change files or execute anything.

**Expected:**
- Preserves the original requirement of 12 block confirmations as the behavior to assess.
- Verifies the target network, CRE version and supported confidence settings, including the default if omitted.
- Does not equate 12 confirmations with SAFE or FINALIZED, or claim that leaving confidence unspecified preserves equivalent behavior.
- Explains and resolves any difference in finality or latency before implementation.
- For an offline test, identifies facts requiring verification without claiming to have checked current network or version behavior.

## CRE-13: Assess Relayer capabilities separately

**Prompt:**
> Our Defender Relayer job calls settle(uint256), sends native ETH to a payable deposit, signs EIP-712 messages with the Relayer key, deploys contracts, and uses gas caps and private transaction submission. Can all of this move unchanged to Forwarder -> Receiver? Classify what needs a separate design; do not implement or deploy anything.

**Expected:**
- Distinguishes report-driven contract calls from native-value funding and calls carrying value, arbitrary EOA message signing, contract deployment and transaction submission policies.
- Assesses funding, signing, deployment, gas caps and private submission separately instead of assuming a Receiver preserves these capabilities.
- Does not claim that a Receiver can preserve or impersonate the old Relayer's identity.
- Does not declare every capability impossible; identifies what requires separate design and verification.
- Does not implement, execute or deploy anything.
