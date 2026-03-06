# Codex Support Layer

This repository keeps the original Claude-oriented skills in `skills/` and adds Codex-specific drafts in `skills-codex/`.

## Use This Layer

- Use Codex skills from `skills-codex/` when working in Codex.
- Keep `skills/` unchanged for Claude compatibility.
- Prefer Codex skill names when invoking directly (for example, `$codex-setup-solidity-contracts`).

## Skill Map

- `codex-develop-secure-contracts`: generic OpenZeppelin integration workflow across Solidity, Cairo, Stylus, and Stellar.
- `codex-setup-solidity-contracts`, `codex-setup-cairo-contracts`, `codex-setup-stylus-contracts`, `codex-setup-stellar-contracts`: project setup.
- `codex-upgrade-solidity-contracts`, `codex-upgrade-cairo-contracts`, `codex-upgrade-stylus-contracts`, `codex-upgrade-stellar-contracts`: upgrade flows and storage safety.

## Coordination

- Combine `codex-develop-secure-contracts` with a setup or upgrade skill when the task spans multiple phases.
- Keep responses library-first and project-first:
- Read existing project files before suggesting code changes.
- Import OpenZeppelin dependencies; do not paste library source into user code.

## Maintenance Rule

- Add Codex updates under `skills-codex/`.
- Do not modify existing files under `skills/` when the goal is Codex compatibility only.
