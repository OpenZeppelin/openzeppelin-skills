# Design Principles

## Purpose

The OpenZeppelin skills guide correct integration of OpenZeppelin Contracts libraries. The goal is **library-guided integration**: discover what the library already provides, identify the minimal correct integration pattern, and apply it to the user's existing codebase.

## Skill Organization

The skills are organized by development lifecycle phase:

- **Setup skills** (`setup-<language>-contracts`) — project scaffolding, dependency installation, and import conventions for each ecosystem
- **Develop skill** (`develop-secure-contracts`) — core development workflow, pattern discovery methodology, MCP generators, and library-first integration principles (generic across all ecosystems)
- **Upgrade skills** (`upgrade-<language>-contracts`) — proxy patterns, initializers, storage compatibility, and upgrade procedures for each ecosystem

## Core Principles

### Use the library

Correctness comes from using secure library components. The skills enforce a decision tree: use an existing component as-is, extend it through its official extension points, or write custom logic only when nothing in the library covers the need. Library source is always imported from the dependency — never copied into user code.

### Read the project first

Before generating any code, the skills read the user's existing contracts and default to integrating into them.

### Discover patterns from source

The primary methodology is **pattern discovery from the installed dependency source**. Rather than relying on prior knowledge, it locates the installed library, browses its directory structure, reads the relevant component source and docs, and extracts the minimal set of changes required. This keeps responses accurate across library versions and ecosystems.

The output of discovery is a **minimal diff**: the exact imports, inheritance/composition, storage, initializer, and override changes needed.

## MCP Generators

When MCP contract generators are available, they serve as an optional shortcut for discovery. The generate-compare-apply approach (baseline → feature variant → diff → apply) replaces the manual source-reading step but follows the same integration discipline. Generator schemas are inspected at runtime; no prior knowledge of available tools or parameters is assumed.

## Scope

The skills cover the full lifecycle of working with OpenZeppelin Contracts libraries: setting up a project, integrating standard components, and managing upgradeability. Each skill is scoped to a specific phase and ecosystem, carrying enough reference knowledge to handle that library's specific composition model, versioning constraints, and upgrade patterns.