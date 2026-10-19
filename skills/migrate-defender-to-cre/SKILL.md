---
name: migrate-defender-to-cre
description: "Migrate OpenZeppelin Defender Actions, Autotasks, Monitors and scheduled or Relayer-backed workflows to Chainlink CRE. Use when assessing CRE migration feasibility, adapting an existing Defender workflow, or validating its event delivery and onchain permissions. Does not cover migration to the self-hosted OpenZeppelin Relayer when CRE is not requested."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Migrate Defender Workflows to CRE

Adapt the user's workflow, preserving its intended behavior. CRE is one migration path; do not redirect a user who chose another destination. This skill covers EVM and offchain workflows using TypeScript, not every Defender capability or CRE network.

## 1. Read the project and establish the boundary

Inspect the Action, Monitor configuration, contract permissions when relevant, manifests, lockfiles and existing CRE configuration. Ask only for missing information that changes the design:

- What starts execution, and which filters, timing requirements or persistent conditions apply?
- What effect is required, who may perform it, and how are duplicates or missed executions handled?
- Which network or external service is involved?

Check installed CLI/SDK APIs, current [supported networks](https://docs.chain.link/cre/supported-networks) where applicable, and [service quotas](https://docs.chain.link/cre/service-quotas). Derive paths, package manager, scripts, config fields and target names from this project. Example layouts are not defaults.

## 2. Map behavior before changing code

| Defender behavior | CRE starting point | Preserve or explicitly resolve |
|---|---|---|
| Scheduled Action / Autotask | [Cron trigger](https://docs.chain.link/cre/guides/workflow/using-triggers/cron-trigger-ts) and handler | Cadence, timezone and condition for doing work |
| Event Monitor and response | [EVM log trigger](https://docs.chain.link/cre/guides/workflow/using-triggers/evm-log-trigger-ts) and handler predicate | Emitting address, event signature, filter semantics and confirmation requirements |
| Webhook-triggered Action | [HTTP trigger](https://docs.chain.link/cre/guides/workflow/using-triggers/http-trigger/configuration-ts) | Request authentication and authorized callers |
| Notification or API action | HTTP capability | Payload, credentials, duplicate handling and errors |
| Relayer-backed contract call | Signed report → Forwarder → Receiver | New caller identity and target authorization |

Deployed HTTP triggers require requests signed by an authorized EVM key. Unsigned webhook sources need signing support or an authorized intermediary; an unsigned local simulation does not establish deployed compatibility.

Function-call/status monitoring, high-frequency schedules, counters and cooldowns need explicit assessment. Explain unsupported behavior without treating it as fundamentally impossible. Do not implement a changed cadence or weaker condition while awaiting approval. An HTTP trigger is not a proven workaround for service limits. Preserve expression semantics, including math.js operators and numeric precision; persistent counters need durable state and concurrency handling.

## 3. Consult references for the behavior being migrated

- **Event triggers or outbound notifications/API calls:** [Events and notifications](references/events-and-notifications.md). Read the relevant sections; receipt replay is only for event-triggered workflows.
- **Onchain writes or Relayer replacement:** [Onchain actions](references/onchain-actions.md), before generating a Receiver or changing permissions.
- **Implementation or validation:** [Runtime and validation](references/runtime-and-validation.md), for config resolution, runtime compatibility and evidence boundaries.
- **A matching error or version-specific question:** [Troubleshooting](references/troubleshooting.md). Historical findings are diagnostic clues, not universal requirements.

The [OpenZeppelin migration examples](https://github.com/OpenZeppelin/defender-to-cre-examples) are optional worked examples. Use a versioned example when it matches the task; do not require its Governor, payload or folder structure.

## 4. Implement and validate within scope

Make a minimal diff. Import existing library components and inspect their installed APIs. Discover optional generator tools only when needed; do not assume a CRE Receiver generator exists. A notification-only workflow needs no Receiver, and monitoring an existing contract needs no replacement contract.

Local preparation does not authorize deployment, broadcasts, account-access requests, secret mutations or live notifications. Simulation can make real HTTP requests. Use an isolated destination and respect authorization already given. Keep credentials and private endpoints out of tracked files and output.

Report the behavior preserved, checks actually run, untested paths and unresolved decisions. Distinguish local tests, simulation, deployed execution and production readiness.
