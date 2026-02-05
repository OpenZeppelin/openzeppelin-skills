# OpenZeppelin Skills

Skills for secure smart contract development with OpenZeppelin Contracts.

## Structure

```
skills/
├── openzeppelin-contracts/
│   ├── SKILL.md
│   └── references/
│       ├── patterns.md
│       ├── setup-cairo.md
│       ├── setup-solidity.md
│       ├── setup-stellar.md
│       ├── setup-stylus.md
│       ├── upgrades-cairo.md
│       └── upgrades-solidity.md
└── openzeppelin-security/
    ├── SKILL.md
    └── references/
        ├── cairo.md
        ├── solidity.md
        ├── stellar.md
        └── stylus.md
```

## Available Skills

| Skill | Purpose | Status |
|-------|---------|--------|
| [openzeppelin-contracts](skills/openzeppelin-contracts/) | Setup, patterns, and upgrades for OpenZeppelin Contracts | ✅ Patterns complete, ❌ Placeholders for setup/upgrades |
| [openzeppelin-security](skills/openzeppelin-security/) | Security best practices | ❌ Placeholders |

## Installation

Copy the skill folder(s) to your `.claude/skills/` directory:

```bash
cp -r skills/openzeppelin-contracts ~/.claude/skills/
cp -r skills/openzeppelin-security ~/.claude/skills/
```

## Ecosystem-Specific Content

Skills may include general guidance, and optionally ecosystem-specific content in the `references/` directory of each skill.