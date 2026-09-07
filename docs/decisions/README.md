# Architecture Decision Records

ADRs preserve the reason behind project constraints so future code does not accidentally erase a deliberate decision.

## Status values

- **Proposed** — open for discussion.
- **Accepted** — current project direction.
- **Superseded** — replaced by a later ADR.
- **Rejected** — considered and deliberately not adopted.

## Format

Each ADR should contain:

```text
# NNNN — Decision title

Status:
Date:

## Context
## Decision
## Consequences
## Alternatives considered
## Revisit when
```

Do not rewrite an accepted ADR to make history look cleaner. If the decision changes, add a new ADR and mark the old one superseded.

## Current decisions

- [`0001-fullscreen-first.md`](0001-fullscreen-first.md) — full-screen Spaces define the primary product.
- [`0002-native-space-identifiers-are-ephemeral.md`](0002-native-space-identifiers-are-ephemeral.md) — OS identifiers are runtime observations, not durable user identity.
- [`0003-overlay-before-injection.md`](0003-overlay-before-injection.md) — augment Mission Control with overlays rather than patching it.
- [`0004-declarative-style-manifest.md`](0004-declarative-style-manifest.md) — app styling is declarative data with no executable hooks.
