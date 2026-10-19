# Events and notifications

Use the event sections for EVM log triggers and receipt replay; use delivery guidance for outbound HTTP actions regardless of trigger. An incoming webhook is an HTTP trigger, not an EVM event subscription.

## Preserve the event semantics

Check the Defender Monitor's confirmation requirement against the CRE log trigger's confidence setting for the target network and version. Do not assume a numeric confirmation count maps directly to a confidence level; explain and resolve any difference in finality or latency before migrating.

Identify the emitting contract address and event signature from the user's configuration and ABI. Generated trigger helpers construct filters; they do not emit events. Inspect the generated binding and installed SDK before using its methods. Keep the existing contract unless the user needs a test fixture.

Translate conditions into handler predicates and check matching and nonmatching inputs. Preserve units and integer precision. A per-log trigger may split Defender's transaction-level batch into separate executions; account for logic that previously combined several logs.

## Replay the intended log

Select a successful receipt and match its log by emitting address and event signature. Do not interchange these indexes:

| Value | Meaning |
|---|---|
| CLI `--trigger-index` | Zero-based position of the intended registered handler |
| CLI `--evm-event-index` | Zero-based position in the transaction receipt's `logs` array |
| CRE event `index` / receipt `logIndex` | Log position within the block |

Derive each from the actual workflow and receipt. Check installed [CLI flags](https://docs.chain.link/cre/reference/cli/workflow) before constructing a replay command with the user's workflow path, target and config. A simulation override does not change deployment configuration. Simulation can send real HTTP requests even without an onchain broadcast.

## Preserve the delivery contract

Use the actual provider's payload, credentials and success/error behavior. A collector accepting arbitrary JSON does not prove a Slack, email or other provider integration works. Email generally needs a mail service.

Keep an existing stable deduplication key and its formatting. When designing a new event identity, account for chain, transaction and log identity as needed by the destination's scope. Do not copy an example's key or payload without checking the consumer.

An `Idempotency-Key` header only helps if the destination enforces it. Coordinate deduplication atomically with the business effect; acknowledge completed duplicates without repeating the effect, while allowing recovery of incomplete attempts. Cache reuse and agreement on HTTP responses do not establish durable deduplication or exactly-once delivery. Follow the [HTTP POST pattern](https://docs.chain.link/cre/guides/workflow/using-http-client/post-request-ts) for the installed SDK and inspect its cache fields.

Test successful delivery, non-success responses and repeated requests against an isolated endpoint. Avoid a cached success masking a failure test: use a fresh failing destination or a verified cache-expiration strategy. Match expected errors to actual code. A successful new event after a failure demonstrates new-event recovery, not redelivery of the failed event.

## Verify active delivery separately

After authorized deployment, confirm activation and cause a new matching trigger. An old receipt used for local replay is not evidence of an active subscription. Correlate execution status with the destination's payload, event identity and effect. Test same-event duplication and recovery separately when required. Report only the handlers and delivery paths actually exercised.

For a concrete Governor illustration, consult [the versioned notification example](https://github.com/OpenZeppelin/defender-to-cre-examples/tree/v1.0.2/gov-alert). Its packages, config keys, events and endpoints belong to that example.
