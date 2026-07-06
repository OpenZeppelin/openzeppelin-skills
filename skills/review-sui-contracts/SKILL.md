---
name: review-sui-contracts
description: "Review Sui Move code that integrates OpenZeppelin Contracts for Sui against the library's own patterns and conventions. Use this when a developer wants their integration checked before shipping. Triggers: \"review my Sui Move code\", \"am I using OpenZeppelin correctly\", \"check this integration\", \"is this idiomatic\", \"review before audit\". Checks correct use of OZ primitives, deviations from examples/doc-comments, upheld invariants, test coverage, and code-quality/style. This is an AI code review, not a formal security audit."
license: AGPL-3.0-only
metadata:
  author: OpenZeppelin
---

# Review Sui Contracts

AI-assisted review of a developer's Sui Move code against **OpenZeppelin Contracts for Sui**: does it use the library's on-chain primitives correctly, and where does it deviate from the library's `examples/`, doc-comments, and documented conventions?

> **This is an AI code review, not a formal security audit.** It supports the audit process by flagging misuse and deviations early; it does not replace an independent audit, and a clean review is not an assurance of safety.

For **building** an integration use `develop-secure-contracts`; for **scaffolding** a project use `setup-sui-contracts`. This skill reviews code that already exists.

## Establish the reference before reviewing

You cannot flag "incorrect use" or "deviation" without the correct pattern in hand — so discover it from the library's own metadata, never from memory, using the discovery chain the **`develop-secure-contracts`** skill defines. Read the primitive's **API and behavior at the revision the integrator builds against** — resolve each OZ dependency's pinned rev from `Move.lock` and use that source / those doc-comments, not `main`. Read the library-wide docs — `llms.txt`, the catalogs, `ARCHITECTURE.md`, `STYLEGUIDE.md` — from the repo's current `main` instead: they evolve continuously and often do not exist at an older pinned rev. For each OZ primitive the code uses, load its authoritative pattern from:

- the package's **`examples/` if it has them, otherwise the module-header doc-comment's idiomatic-usage block** — the canonical recipe (some packages ship only the latter);
- the modules' **doc-comments** and generated API reference (reached via the catalog's `Docs` link);
- **`ARCHITECTURE.md`** (capability model, object ownership, initialization) and **`STYLEGUIDE.md`** (conventions).

Review the code against those, not against assumptions.

## What to review

- **Correct use of OZ primitives.** Does the code thread the capability/witness the component defines (rather than hand-rolling `sender` checks or re-implementing what the library already provides), call functions with the right arguments, and handle the documented abort conditions?
- **Who chooses the type argument — and is it the right one.** For each OZ primitive parameterized by a witness or generic/`phantom` type, check *who* supplies it and whether it is the correct authority. A caller-chosen type is an authority the caller controls, so the trusted party must fix it, not the caller. And a developer-fixed type must be the *right* one for the action — a privileged operation gated by a lower-privilege role's `Auth` (or any weaker witness than the guarantee needs) is a privilege-escalation bug.
- **Deviations from the reference pattern.** Compare the flow against the closest `examples/` recipe (or the doc-comment's idiomatic-usage block) and flag divergences that weaken the guarantees it preserves — e.g. an object shared vs. embedded against its intended ownership model, a required setup step skipped, a returned value or receipt ignored, or an invariant the doc-comment states being bypassed.
- **Copied library source.** Flag any OZ module source copied into the project instead of imported via MVR — copies miss security updates.
- **Invariants upheld.** Check that the code preserves the invariants the OZ components document (in their doc-comments and `ARCHITECTURE.md`) — e.g. an object that must stay shared, a value that must remain bounded, a capability that must not be exposed. Flag any path that can break one.
- **Test coverage of the integration.** Check that the tests exercise the abort and branch paths the OZ usage introduces (role-denied, expired, over-limit, wrong-type, and similar), not just the happy path — untested failure paths are where integration bugs hide.
- **Known issue classes (heuristic).** Check the code for the classes of issue that recur in OZ-on-Sui / Move code — capability handling, rounding, object ownership, initialization — and raise suspicion where the code touches them. (The library's published audits, under `audits/`, inform these classes; the reports themselves are PDFs, not a findings feed.) A suspicion-raising self-check, not a reproduction of the audits or a security audit.

## Code quality — enforce the conventions

Treat OpenZeppelin's Sui conventions as a first-class review dimension: report violations as **findings, not optional suggestions**. The rules and the review procedure are the single source of truth — apply them, don't restate them:

- **[`STYLEGUIDE.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/STYLEGUIDE.md)** — the codified rules (naming, section ordering, imports, idiomatic Move 2024, testing, docs, lint suppression).
- **[`.claude/commands/code-quality.md`](https://raw.githubusercontent.com/OpenZeppelin/contracts-sui/main/.claude/commands/code-quality.md)** — the canonical procedure that checks `.move` files against `STYLEGUIDE.md`; run it against the code under review.
- **Lint is not optional.** Build and test with the project's `--build-env` and `--lint` (see `setup-sui-contracts` for the exact invocation); per the styleguide, warnings must be **fixed**, never suppressed with `#[allow(...)]`. `--lint` enforces the compiler-checkable subset independently, so if the `STYLEGUIDE.md` fetch is momentarily unreachable, retry rather than skipping conventions.

Report each violation against the `STYLEGUIDE.md` section it breaks.

## Verify before reporting

The checks above optimize for catching issues, so before writing anything, try to *disprove* each finding: quote the exact line and confirm the problem is really there; confirm it is a genuine deviation and not an intentional, documented choice; and confirm you can state a concrete one-line fix. Drop whatever does not survive all three, and keep the severity bar high — a short review a developer trusts beats a long one they skim.

## Output

Produce one review, findings ordered by severity. Each finding: `file:line` — what's wrong (one clause) → the fix (one clause) — and the OZ `example` or doc-comment it deviates from, so the developer can compare directly. A short preamble is useful — the scope/boundary call and the build/test/lint status — and on a clean integration a single "verified and cleared" line is fine; keep the body to findings, not a recap.

If the project builds, run its checks first (`sui move build` / `test`, per `setup-sui-contracts`) so the review reflects compiling code, and report build/test failures separately from the pattern findings.

## Boundaries

- Reviews the **integrator's** code — how it uses the library — not the OpenZeppelin library internals themselves.
- **Not a security audit** — no exhaustive threat modeling or completeness guarantee; recommend a formal audit for production code.
- Complements `develop-secure-contracts` (integration) and `setup-sui-contracts` (scaffolding); it neither builds nor scaffolds.
