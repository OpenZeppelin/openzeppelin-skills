# Onchain actions and Relayer permissions

Read the target's source, deployed permissions and the installed CRE contracts before choosing a design. A notification-only migration does not need this component.

The Forwarder -> Receiver path is not a general replacement for a Relayer. Native-value transfers, calls carrying value, message signing, contract deployments, and gas or mempool policies require separate assessment; do not assume a Receiver preserves them.

## Explain the new call path

A report carries the operation's data signed by a Decentralized Oracle Network (DON). Chainlink's Forwarder verifies it and calls a user-controlled Receiver. The Receiver processes the payload directly or calls an existing target:

```text
CRE workflow → signed report → Chainlink Forwarder → user Receiver
                                                        └→ existing target (optional)
```

Use the official [consumer-contract guide](https://docs.chain.link/cre/guides/workflow/using-evm-client/onchain-write/building-consumer-contracts) for `IReceiver`, `onReport`, `ReceiverTemplate` and the proxy-pattern example. Import the matching library version instead of copying its implementation. In this context, the proxy pattern is a receiving adapter, not necessarily an upgradeable proxy.

| Design | Caller the contract sees |
|---|---|
| Target receives reports directly | Its `onReport` sees the Forwarder |
| Separate Receiver calls the target | Receiver sees Forwarder; target sees Receiver |

Grant the Receiver the target role required for the specific operation when authorized. A normal adapter cannot preserve or impersonate the old Relayer's `msg.sender`. A permanently fixed old-caller check blocks this call path; assess an alternative authorized signer or a different target design. Do not describe deploying an adapter as sufficient. Read-only functions and publicly callable functions may need no new role.

## Restrict both delivery and source identity

Resolve the correct Forwarder for the network and environment from [the directory](https://docs.chain.link/cre/guides/workflow/using-evm-client/forwarder-directory) and the account's supported-chain information. Do not assume one universal address per chain.

Require the trusted Forwarder **and** the intended workflow identity. Inspect the installed template: relevant versions provide an expected workflow ID, or an expected author together with workflow name. Confirm the actual setters and matching rules before generating configuration. Trusting the Forwarder alone does not restrict accepted reports to this user's workflow.

Decode only intended operations and validate their arguments. Keep authorization, chain binding and application-level replay protection even if transport has duplicate-transmission checks. Distinct reports with identical payloads can still repeat the business operation. Use a stable operation ID and an appropriate duplicate policy where repetition matters.

For metadata-layout errors or retry-state questions, read the matching sections in [Troubleshooting](troubleshooting.md). Transport retry eligibility is version-dependent and does not establish automatic resubmission.

## Validate the operation, not just submission

- Successful delivery: inspect transaction and receiver-execution status using the installed SDK's result types, then read the target state to confirm the intended effect.
- Unauthorized caller: a direct EOA call should fail, but that proves the caller guard only.
- Wrong workflow identity: test rejection through a trusted test Forwarder path with wrong metadata, and success with allowed metadata.
- Repetition and invalid payload: verify the contract's intended duplicate behavior and that malformed or unauthorized operations cannot execute.

A simulator's mock Forwarder may omit production metadata. Use an isolated test Receiver or a suitable test harness to cover those paths; do not disable identity checks on the deployed Receiver to make a simulation pass. A single-node simulation or mock broadcast does not establish production DON authorization.

For a scheduled job (sometimes called a keeper), the handler checks whether work is due before creating a report. This name does not imply registration in Chainlink Automation. If schedule enforcement is a contract invariant, implement it onchain too; the workflow's clock alone cannot enforce it against all callers.
