# OpenZeppelin Skills

Agent skills for smart contract development with OpenZeppelin Contracts libraries.

## Installation

### Option 1: Skills CLI

```bash
npx skills add OpenZeppelin/openzeppelin-skills
```

### Option 2: Claude Code Plugin

```bash
/plugin marketplace add OpenZeppelin/openzeppelin-skills
/plugin install openzeppelin-skills
```

### Option 3: Manual

Copy skill folders to your Claude skills directory:

```bash
cp -r skills/*-contracts ~/.claude/skills/
```

## Available Skills

| Skill | Purpose |
|-------|---------|
| [setup-solidity-contracts](skills/setup-solidity-contracts/) | Set up a Solidity project |
| [setup-cairo-contracts](skills/setup-cairo-contracts/) | Set up a Cairo project |
| [setup-stylus-contracts](skills/setup-stylus-contracts/) | Set up a Stylus project |
| [setup-stellar-contracts](skills/setup-stellar-contracts/) | Set up a Stellar project |
| [develop-contracts](skills/develop-contracts/) | Develop contracts using OpenZeppelin Contracts libraries |
| [upgrade-solidity-contracts](skills/upgrade-solidity-contracts/) | Upgrade Solidity contracts |
| [upgrade-cairo-contracts](skills/upgrade-cairo-contracts/) | Upgrade Cairo contracts |
| [upgrade-stylus-contracts](skills/upgrade-stylus-contracts/) | Upgrade Stylus contracts |
| [upgrade-stellar-contracts](skills/upgrade-stellar-contracts/) | Upgrade Stellar contracts |

## License

AGPL-3.0
