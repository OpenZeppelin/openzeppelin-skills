# Codex Testing Guide (Draft)

Use this checklist to validate the `skills-codex/` layer without modifying the original `skills/` files.

## Scope

- Confirm correct skill triggering in Codex.
- Confirm library-first, project-first behavior.
- Confirm ecosystem-specific setup and upgrade guidance is accurate.

## Core Checks

1. Triggering
- Prompt with explicit skill intents (setup, upgrades, component integration).
- Verify the matching `codex-*` skill activates.

2. Project-first behavior
- Provide an existing contract file.
- Ask for a feature addition.
- Verify the agent reads and edits existing code rather than replacing it.

3. Library-first behavior
- Ask for access control, pausability, upgradeability, or token standards.
- Verify the agent imports OpenZeppelin components instead of hand-writing equivalents.

4. Setup coverage
- Run one setup prompt per ecosystem (Solidity/Cairo/Stylus/Stellar).
- Verify toolchain, dependency, and import guidance is ecosystem-correct.

5. Upgrade safety coverage
- Run one unsafe storage-change prompt per ecosystem.
- Verify the agent warns about compatibility hazards and proposes safe alternatives.

6. MCP fallback behavior
- Ask for a feature likely absent from generator schema.
- Verify the agent falls back to source pattern discovery rather than refusing.

## Result Labels

- `PASS`: expected behavior is complete.
- `PARTIAL`: mostly correct, with missing key details.
- `FAIL`: incorrect workflow, unsafe advice, or hallucinated APIs.

## Regression Targets

- Do not copy/paste library source code into user contracts.
- Do not recommend in-place v4 to v5 proxy upgrades for Solidity OpenZeppelin Contracts.
- Do not skip project inspection when user already has a repository.
