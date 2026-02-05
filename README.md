# OpenZeppelin Skills

Skills for secure smart contract development with OpenZeppelin Contracts.

## Structure

```
skills/
└── openzeppelin/
    ├── SKILL.md (umbrella - routes to sub-skills)
    ├── openzeppelin-setup/
    ├── openzeppelin-patterns/
    ├── openzeppelin-upgrades/
    └── openzeppelin-security/
```

## Available Skills

| Skill | Purpose | Status |
|-------|---------|--------|
| [openzeppelin](skills/openzeppelin/) | General guidance and skill discovery | ✅ Initial draft |
| [openzeppelin-setup](skills/openzeppelin/openzeppelin-setup/) | Project installation and configuration | ❌ Placeholders |
| [openzeppelin-patterns](skills/openzeppelin/openzeppelin-patterns/) | Pattern discovery via OpenZeppelin MCP generators | ✅ Initial draft |
| [openzeppelin-upgrades](skills/openzeppelin/openzeppelin-upgrades/) | Upgradeable contract patterns | ❌ Placeholders |
| [openzeppelin-security](skills/openzeppelin/openzeppelin-security/) | Secure development best practices | ❌ Placeholders |

## Installation

### Full skill (recommended)

Copy the entire `openzeppelin/` folder to your `.claude/skills/` directory:

```bash
cp -r skills/openzeppelin ~/.claude/skills/
```

The umbrella skill routes to sub-skills as needed.

### Individual sub-skills

To install a sub-skill as a standalone top-level skill:

```bash
cp -r skills/openzeppelin/<SKILL_NAME> ~/.claude/skills/
```

This allows direct triggering without going through the umbrella.

## Ecosystem-Specific Content

The `setup`, `patterns`, and `upgrades` skills include guidance for:

- **Solidity/EVM** — Hardhat, Foundry
- **Cairo/Starknet** — Scarb
- **Stylus** — Cargo + Stylus SDK
- **Stellar** — Soroban

## Security Best Practices

The `security` skill covers security principles that apply to all smart contract development, with ecosystem-specific guidance in reference files.

## Future Distribution

A packaging script may be added to create distributable `.skill` files (zip archives) for easier installation of either the full skill or individual sub-skills.
