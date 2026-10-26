# Conditional troubleshooting

Read only the section matching the observed symptom. These findings come from particular versions; confirm installed and deployed sources before applying a workaround.

## Generated bindings fail on overloaded ABI entries

If the binding generator fails on a full ABI, inspect the failing entries. For an event-only workflow, an events-only ABI may avoid unsupported overloads without changing the contract. Prefer regenerating bindings to editing generated code that regeneration will overwrite.

## Missing JavaScript built-ins

Compatibility probes with CLI 1.34.0 and SDK 1.19.1 and 1.21.1 found `Intl.NumberFormat` unavailable and `structuredClone` undefined. Recheck the user's version. Use deterministic formatting that preserves integer precision; do not convert large `bigint` values to `number` for separators. Clone the required fields explicitly or use a compatible tested implementation. JSON round-tripping is not a general substitute: it cannot serialize `bigint` and changes other types.

## Receiver metadata layout or transmission retry errors

In [KeystoneForwarder at revision b6427ea](https://github.com/smartcontractkit/chainlink-evm/blob/b6427ea1f4847d640abdf24dbd6c6f01d7799d59/contracts/cre/src/v1/KeystoneForwarder.sol), the metadata slice is `rawReport[45:109]`: 64 bytes, including 62 bytes of workflow identity fields and a two-byte report ID. An exact `metadata.length == 62` check rejects that path. Use the matching library decoder/template and validate against the actual Forwarder version; do not remove workflow identity checks to fix a layout mismatch. This length is established by source inspection, not by measuring every deployed delivery.

At that same revision, a transmission recorded as successful or `invalidReceiver` rejects another attempt with `AlreadyAttempted`. A failed receiver call, including an out-of-gas failure, can be eligible for another attempt; that does not prove automatic resubmission. Inspect transmission state and repair the cause before deciding whether to retry. An invalid-receiver result is not fixed by blindly resubmitting the same transmission after changing the Receiver. Any new report must preserve the application's duplicate protections.

## Empty decode shortly after contract deployment

The contract may not yet exist at the finalized block used for the read. Check network, address, ABI and selected block first. Waiting for finality can resolve that specific condition, not every empty decode.

## Secret-management authorization or input errors

For CLI 1.34.0 with a private registry and browser authorization, the observed path required a read-only Ethereum mainnet RPC to verify gateway signatures, not a mainnet transaction. Each operation required authorization, which could expire. Creation mapped identifiers to environment-variable names; deletion took a list of identifiers. Check installed help, registry, auth method and [current secret-management documentation](https://docs.chain.link/cre/reference/cli/secrets) instead of assuming these requirements are universal.

If management succeeds but workflow consumption is unproven, use an authorized isolated high-entropy test secret. Compare a digest inside the workflow to its expected digest and expose only pass/fail, keeping the value out of artifacts and logs. Do not use this method to publish fingerprints of low-entropy or production credentials.

## Execution lookup cannot find a newly registered workflow

Check workflow identifier, registry and activation state, then retry within a bounded period. Registration and execution lookup may not become visible simultaneously. Investigate persistent errors instead of redeploying automatically or claiming success because registration returned an ID.
