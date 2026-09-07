# 0002 — Native Space identifiers are ephemeral

**Status:** Accepted  
**Date:** 2026-09-07

## Context

Undocumented macOS data exposes managed-space integers, `id64`-style values, UUID-like strings, per-display order, window membership, and related references. Some tools report that certain UUIDs persist across reboots and successfully use them for desktop naming.

Apple does not publish a compatibility contract for these identifiers, and the SpaceDress primary use case naturally has stronger identity: the app or split-app set occupying a full-screen Space.

## Decision

All macOS-provided Space identifiers are treated as **ephemeral observations** for architecture and persistence purposes.

They may be used for:

- joining platform data during a snapshot;
- tracking a Space within a running session;
- diagnostics;
- short-lived caches;
- reconciliation hints.

They may not be the sole durable key for:

- user styles;
- per-app preferences;
- named full-screen contexts;
- persisted split-Space rules.

SpaceDress-owned persistent rules target logical application/document identity instead.

## Consequences

- A reboot or Space recreation does not inherently lose user styling.
- The system can survive Apple changing ID generation.
- Reconciliation code is somewhat more sophisticated.
- A stable native UUID can still be used opportunistically without becoming architectural debt.

## Alternatives considered

### Persist native UUIDs

Attractive because existing tools demonstrate practical stability in some environments. Rejected as the foundational identity because the guarantee is undocumented and unnecessary for the full-screen problem.

### Persist Mission Control index

Rejected because Spaces may reorder and index describes location, not identity.

## Revisit when

A future public Apple API explicitly defines a durable Space identifier with persistence semantics useful to SpaceDress. Even then, app identity may remain the better user-facing selector.
