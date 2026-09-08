# 0006 — Layered user style selectors

**Status:** Accepted  
**Date:** 2026-09-08

## Context

Bundle identity is a strong default key for application-level customization, but it is not sufficient for the full SpaceDress use case.

A user may need to distinguish:

- two different applications that form a recurring split-screen pair;
- several full-screen windows belonging to the same application;
- several documents opened by one application;
- several browser windows that share both a bundle identifier and a process;
- copied/development builds sharing a bundle identifier;
- a current window instance that has no durable document identity.

Process IDs, window IDs, native Space IDs, and Mission Control positions are runtime observations and cannot be treated as durable user identity.

## Decision

SpaceDress uses a layered selector model.

### Application selector

The normal durable application key is the macOS bundle identifier.

```text
ApplicationSelector
  bundleIdentifier
  bundlePath?        # optional exact-copy distinction
  teamIdentifier?    # optional authenticity/disambiguation constraint
```

`bundleIdentifier` is the default user-facing identity. `bundlePath` and signing/team identity refine a selector when two installed copies must be distinguished.

If a running application has no bundle identifier, SpaceDress may fall back to a weaker project-owned selector derived from the actual executable/bundle location, but that fallback must be labeled as less portable.

### Split-pair selector

A recurring two-application split may have its own user rule.

```text
ApplicationPairSelector
  members[2]
```

The members form a **canonical unordered multiset** of application selectors. Physical left/right placement is not part of pair identity.

Therefore:

```text
Chrome + VS Code == VS Code + Chrome
```

and this is also valid:

```text
Chrome + Chrome
```

A pair rule may define whole-Space/container appearance and contextual participant overrides. Physical-side overrides, when offered, are applied using observed participant geometry after the pair has matched.

### Runtime participant identity

For instance styling, the thing being styled is the visible Space participant/window, not merely a process.

```text
ParticipantObservation
  application
  pid?
  windowID?
  documentIdentity?
  observedRegion?
```

`pid` and `windowID` are excellent runtime correlation keys but are not durable identity.

### Durable instance/document binding

When the platform exposes a genuinely stable document or application-specific identity, a user may explicitly bind a style to it.

Examples may include a stable document URL or future app-specific stable identifier.

A window title by itself is presentation/matching data, not durable identity.

### Current-instance binding

When no durable identity exists, SpaceDress may attach a style to the current participant/window lifetime.

This assignment may use current window/process observations and a session cache, but the product must not promise that it survives application restart or system reboot.

### Instance style pools

An application may define multiple reusable instance styles and an assignment policy.

Conceptually:

```text
InstanceStyleSet
  appSelector
  styles[]
  assignmentPolicy   # manual / sequential
```

For sequential assignment, the first concurrently observed unmatched instance receives the first available style, the second receives the second style, and so on. Assignment is a lease held while that participant remains live/reconciled; it is not derived from Mission Control order.

A style pool should avoid duplicate visual assignments among concurrent instances until the pool is exhausted. Overflow behavior must remain visibly distinguishable or explicitly documented.

## User-rule precedence

Participant appearance resolves approximately as:

```text
platform-derived baseline
  -> optional DSAM
  -> application user rule
  -> split-pair member/context rule
  -> explicit instance/document/instance-slot rule
  -> required global privacy/accessibility/safety policy
```

Whole-pair/container decoration is composed separately so it does not erase participant identity.

An explicit instance choice is more specific than a generic application or pair-member rule.

## Interaction

SpaceDress should support direct customization from Mission Control when it can do so without breaking native interaction.

The intended action is conceptually **Customize This Instance** on the participant/thumbnail, with quick selection among reusable styles and access to app and pair configuration.

Because the normal rendering overlay is click-through, the exact event mechanism must be proven experimentally. Interaction must not require converting the whole thumbnail overlay into an input-blocking replacement for Mission Control.

## Consequences

- Bundle ID remains the simple/default key for most customization.
- Split pairs can have context-specific styling without depending on left/right order.
- Multiple windows/documents from one app can be visually distinct.
- SpaceDress can automatically distribute reusable styles among concurrent same-app instances.
- Runtime instance styling remains useful even when durable per-window identity is impossible.
- Persistence claims stay honest: durable bindings require durable evidence.
- The user configuration format becomes richer than DSAS by design; this machinery remains SpaceDress-private.

## Alternatives considered

### Bundle identifier only

Rejected because it collapses every window/document from one app to the same appearance.

### PID as instance identity

Rejected for persistence because PIDs are process-lifetime values and may not distinguish multiple windows in one process.

### Window ID as durable identity

Rejected because current window IDs are runtime correlation data without a macOS persistence contract.

### Pair key ordered by left/right

Rejected because swapping the physical sides would incorrectly create a different logical pair.

### Mission Control order as instance numbering

Rejected because Spaces may reorder and the order is unrelated to the user's intended instance identity.

## Revisit when

A supported macOS API or app-facing standard provides a stable per-window identity with stronger guarantees than the current observation/document model.
