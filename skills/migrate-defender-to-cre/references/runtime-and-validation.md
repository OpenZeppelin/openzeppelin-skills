# Runtime and validation

## Resolve the user's configuration

Read manifests, lockfiles and scripts to find package boundaries and the package manager. Inspect CLI help and installed SDK/contract types; use current official docs for service behavior. Explain version differences instead of mixing APIs or upgrading dependencies to match an example.

Resolve `project.yaml` and `workflow.yaml` for the selected target, including paths relative to the applicable project/workflow context. Prepare and validate the file deployment will read using the actual schema. No particular filename, target name or set of config keys is required by this skill. A simulation `--config` override does not persist into deployment.

When real values are unavailable, use explicit fixtures for config-independent tests and report deployment configuration as unresolved. Passing tests with fixture values does not mean deployment configuration is ready.

## Check the actual execution environment

TypeScript workflows compile to WebAssembly and run in CRE's runtime, not Node.js. Validate configuration parsing and runtime dependencies there as well as in local tests: a passing Node/Bun check does not prove a built-in or dependency exists in CRE. Preserve integer precision and types when adapting formatters or cloning code.

[`Runtime` and `NodeRuntime`](https://docs.chain.link/cre/reference/sdk/core-ts) distinguish DON execution from per-node work. Follow [non-determinism guidance](https://docs.chain.link/cre/concepts/non-determinism-ts) and installed time, randomness and aggregation APIs. Do not substitute a constant for required randomness or confuse agreement on responses with a single HTTP request.

Missing deploy access does not prevent local implementation. Inspect access with read-only status commands; login, registry selection and `cre account access` may change state or request access. Check help and the [access guide](https://docs.chain.link/cre/account/deploy-access) before acting. Simulation may require RPC/authentication and can send real HTTP requests; `--broadcast` only controls the onchain boundary.

Use runtime [secrets](https://docs.chain.link/cre/guides/workflow/secrets/using-secrets-deployed) for credentials. A create/list/delete test proves management, not deployed retrieval. When consumption needs validation, use an authorized isolated test secret and a non-revealing assertion; never log values or publish production-secret fingerprints. Consult [Troubleshooting](troubleshooting.md) for matching runtime, secret-management or lookup errors.

## Validate progressively

| Stage | Check | Does not establish |
|---|---|---|
| Local build and tests | Types, config, payloads, filters, errors | Deployed delivery or network consensus |
| Relevant trigger simulation | Selected handler, runtime config, isolated effects | Multi-node agreement, active subscriptions or automatic retries |
| Authorized test deployment | Fresh triggers, execution status, destination effects | Untested handlers or production readiness |

For notifications, use [Events and notifications](events-and-notifications.md) to test delivery and duplicates. For contract writes, use [Onchain actions](onchain-actions.md) to test caller/identity guards and target state. For thresholds and cooldowns, include ordering, boundary times, concurrency, duplicates and storage failure; process memory and short-lived caches are not durable state.

Report what each check proves, including failures and gaps. Production cutover needs an overlap/backfill decision, duplicate control, monitoring and recovery. Remove only test resources created for the run within the authorized cleanup scope.
