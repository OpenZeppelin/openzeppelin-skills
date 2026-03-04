# OpenZeppelin Skills

Agent skills for secure smart contract development with OpenZeppelin Contracts libraries.

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

**Note:** This method automatically installs optional MCP servers for smart contract generation. See [MCP Servers](#mcp-servers) below.

### Option 3: Manual

Copy skill folders to your Claude skills directory:

```bash
cp -r skills/*-contracts ~/.claude/skills/
```

## Available Skills

| Skill | Purpose |
|-------|---------|
| [develop-secure-contracts](skills/develop-secure-contracts/SKILL.md) | Develop secure smart contracts using OpenZeppelin Contracts libraries |
| [setup-solidity-contracts](skills/setup-solidity-contracts/SKILL.md) | Set up a Solidity project |
| [setup-cairo-contracts](skills/setup-cairo-contracts/SKILL.md) | Set up a Cairo project |
| [setup-stylus-contracts](skills/setup-stylus-contracts/SKILL.md) | Set up a Stylus project |
| [setup-stellar-contracts](skills/setup-stellar-contracts/SKILL.md) | Set up a Stellar project |
| [upgrade-solidity-contracts](skills/upgrade-solidity-contracts/SKILL.md) | Upgrade Solidity contracts |
| [upgrade-cairo-contracts](skills/upgrade-cairo-contracts/SKILL.md) | Upgrade Cairo contracts |
| [upgrade-stylus-contracts](skills/upgrade-stylus-contracts/SKILL.md) | Upgrade Stylus contracts |
| [upgrade-stellar-contracts](skills/upgrade-stellar-contracts/SKILL.md) | Upgrade Stellar contracts |

## MCP Servers

Optional MCP servers provide smart contract generation tools. These are automatically installed with the Claude Code Plugin, or can be configured manually by following the steps at https://mcp.openzeppelin.com/

## License

This project is licensed under the GNU Affero General Public License v3.0 - see the [LICENSE](LICENSE) file for details.
