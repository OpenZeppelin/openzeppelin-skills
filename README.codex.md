# OpenZeppelin Skills for Codex (Draft)

This is a Codex-focused companion layer for the existing OpenZeppelin skills repository.

- Existing Claude-oriented skills remain in `skills/` (unchanged).
- Codex drafts live in `skills-codex/`.

## Install in Codex

Copy the Codex skill folders into your Codex skills directory:

```bash
cp -r skills-codex/* "$CODEX_HOME/skills/"
```

On Windows PowerShell:

```powershell
Copy-Item -Recurse skills-codex\* $env:CODEX_HOME\skills\
```

## Included Codex Skills

- `codex-develop-secure-contracts`
- `codex-setup-solidity-contracts`
- `codex-setup-cairo-contracts`
- `codex-setup-stylus-contracts`
- `codex-setup-stellar-contracts`
- `codex-upgrade-solidity-contracts`
- `codex-upgrade-cairo-contracts`
- `codex-upgrade-stylus-contracts`
- `codex-upgrade-stellar-contracts`

## Design Goals

- Keep compatibility without changing existing files.
- Preserve OpenZeppelin library-first and project-first workflows.
- Keep Codex skill instructions concise and tool-agnostic.

## Notes

- This is a draft Codex layer.
- Treat `dev/TESTING.codex.md` as the initial validation checklist.
