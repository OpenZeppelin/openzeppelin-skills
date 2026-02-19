# Design

## Purpose

The `openzeppelin-contracts` skill guides correct integration of OpenZeppelin Contracts libraries. The goal is **library-guided integration**: discover what the library already provides, identify the minimal correct integration pattern, and apply it to the user's existing codebase.

## Core Principles

### Use the library

Correctness comes from using secure library components. The skill enforces a decision tree: use an existing component as-is, extend it through its official extension points, or write custom logic only when nothing in the library covers the need. Library source is always imported from the dependency — never copied into user code.

### Read the project first

Before generating any code, the skill reads the user's existing contracts and defaults to integrating into them.

### Discover patterns from source

The skill's primary methodology is **pattern discovery from the installed dependency source**. Rather than relying on prior knowledge, it locates the installed library, browses its directory structure, reads the relevant component source and docs, and extracts the minimal set of changes required. This keeps responses accurate across library versions and ecosystems.

The output of discovery is a **minimal diff**: the exact imports, inheritance/composition, storage, initializer, and override changes needed.

## MCP Generators

When MCP contract generators are available, they serve as an optional shortcut for discovery. The generate-compare-apply approach (baseline → feature variant → diff → apply) replaces the manual source-reading step but follows the same integration discipline. Generator schemas are inspected at runtime; no prior knowledge of available tools or parameters is assumed.

## Scope

The skill covers the full lifecycle of working with OpenZeppelin Contracts libraries: setting up a project, integrating standard components, and managing upgradeability. Each reference within the skill is scoped to a specific library and carries enough reference knowledge to handle that library's specific composition model, versioning constraints, and upgrade patterns.