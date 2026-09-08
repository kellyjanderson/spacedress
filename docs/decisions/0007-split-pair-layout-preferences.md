# 0007 — Split-pair layout preferences

**Status:** Accepted  
**Date:** 2026-09-08

## Context

Recurring full-screen split pairs are not only visual contexts. Users may repeatedly recreate the same working pair and then manually restore the same divider position. For example, a browser may need substantially more width than a terminal whenever those two applications form a native macOS split full-screen Space.

SpaceDress already models a split pair as a canonical unordered pair of application selectors and observes the actual participant rectangles. That makes a preferred allocation such as 65/35 a natural extension of the pair rule.

The difficult part is mutation. macOS does not publish a dedicated API for setting the divider percentage of another application's native full-screen Split View. Accessibility can report whether an element attribute is settable and can set supported attributes, but whether native tiled full-screen participant windows accept size/position changes must be established empirically on each supported macOS release.

## Decision

A SpaceDress pair rule may carry an optional **preferred split allocation** in addition to appearance.

For distinct identifiable members, the preferred representation is allocation by logical member rather than by current left/right position:

```text
PairRule
  selector: canonical unordered pair
  preferredLayout?
    allocation
      Chrome: 0.65
      iTerm2: 0.35
    applyPolicy: onceWhenPairBecomesStable
```

This means the preference survives a physical side swap. Chrome receives 65% whether it currently occupies the left or right participant region.

When pair members cannot be durably distinguished, such as a same-app pair with no stable document/instance selectors, SpaceDress may instead persist a physical allocation:

```text
preferredLayout
  leftShare: 0.65
```

The actual observed rectangle remains the runtime source of truth.

### Application policy

The default policy is **apply once when a matching pair becomes stable**.

SpaceDress must not continuously force the divider back to the configured percentage while the user is interacting with the split. A manual divider adjustment after restoration is respected until the pair is recreated or the user explicitly requests restoration again.

A future explicit `lock layout` feature would be a separate decision because continuous enforcement changes the relationship between SpaceDress and normal macOS interaction.

### Capability-gated mutation

Implementation must use the least invasive mechanism that works reliably:

1. supported Accessibility mutation when the relevant window attributes are reported settable and the resulting native split remains valid;
2. another supported/system-mediated mechanism if one is discovered;
3. user-visible divider dragging/event synthesis only as an explicitly evaluated fallback, because it is intrusive and timing-sensitive;
4. private SkyLight/CGS mutation only under a separate ADR and only if the project security/compatibility policy is intentionally changed.

The feature is capability-gated. If SpaceDress cannot safely restore a native split ratio on a given macOS release/configuration, it must report the capability unavailable rather than pretending success.

### Verification

After attempting restoration, SpaceDress re-observes participant geometry and verifies the achieved ratio within a documented tolerance. macOS or an application may impose minimum widths and clamp the requested ratio. A clamped result is reported as such; SpaceDress does not enter a retry loop that fights the system.

## Consequences

- A recurring `Chrome + iTerm2` split can remember both its visual identity and its working geometry.
- Pair identity remains independent of physical side.
- Same-app pairs still have a usable physical-side fallback.
- User manual adjustments remain authoritative after the initial restore.
- Layout mutation does not enter DSAS; it is SpaceDress user/workspace machinery.
- Support may vary by macOS version or application capabilities and must be exposed honestly.

## Alternatives considered

### Persist only `left = 65%`

Too weak for distinct app pairs because swapping sides would silently give the larger allocation to the wrong app.

### Continuously enforce the configured percentage

Rejected as the default because SpaceDress would fight normal divider dragging and turn a convenience into a window-management policy engine.

### Depend on yabai-style tiling

Rejected as the product architecture. Third-party tilers demonstrate that programmable split ratios are useful, but SpaceDress is targeting native macOS full-screen Split View and keeps SIP enabled.

## Revisit when

- a supported macOS API directly exposes native Split View allocation;
- current Accessibility behavior proves insufficient;
- users request persistent side assignment or automatic pair construction in addition to ratio restoration;
- a lock/enforcement mode becomes independently compelling.
