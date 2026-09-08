# SpaceDress TDD Implementation Plan

This is the executable engineering plan for taking SpaceDress from a specification-first repository to a releasable macOS application.

The product/risk roadmap remains [`ROADMAP.md`](ROADMAP.md). This document is deliberately lower level: **every unchecked item is intended to fit in one focused coding-agent prompt** and leave the branch in a green, reviewable state.

## How to use this plan

### One checkbox = one prompt

A slice should be implemented alone unless it explicitly says otherwise. Do not opportunistically complete later slices.

For every slice:

1. **Red** — add the smallest failing automated test, contract test, or reproducible system probe that demonstrates the missing behavior.
2. **Green** — implement only enough production code to make that test/probe pass.
3. **Refactor** — improve names/boundaries without changing behavior; rerun the relevant suite.
4. Run the broader affected test suite before considering the slice complete.
5. Update documentation when the slice changes a public contract, private-SPI assumption, permission, persisted format, or user-visible behavior.
6. Check the box only after the work is merged or otherwise accepted into the integration branch.

A useful prompt wrapper is:

```text
Implement only SpaceDress implementation-plan slice SD-### using strict TDD.
Read AGENTS.md and the documents referenced by the slice first.
Start by producing the failing test/probe. Then implement the minimum behavior,
refactor with tests green, run all affected tests, and report exactly what changed.
Do not start later slices. Mark SD-### complete only if every acceptance condition passes.
```

## Test architecture

Use four test levels deliberately:

- **Unit tests — Swift Testing.** Pure domain logic, decoding, merging, transforms, policies, state machines, and adapters around injected values.
- **Fixture/contract tests — Swift Testing.** Recorded SkyLight/CGS/Accessibility/window payloads are sanitized into fixtures and used to test translators without requiring a specific live macOS state.
- **System/probe tests — explicit real-Mac suite.** Tests that exercise private macOS SPI, Mission Control, permissions, multi-display behavior, or real full-screen Spaces. These must report macOS build/configuration and may be excluded from routine CI.
- **UI/performance tests — XCTest/XCUIAutomation.** Settings workflows, accessibility-visible UI, launch behavior, and performance baselines.

Do not replace uncertain private-macOS behavior with mocks and then treat the mock result as evidence. Mock project-owned protocols; validate the platform adapter itself with captured fixtures and real-system probes.

## Global definition of done

A slice is not complete if it leaves:

- a failing test;
- a warning that indicates a correctness problem;
- a new private-SPI assumption without a probe or research note;
- persistent state keyed by native Space ID/UUID/index;
- Mission Control overlay behavior that can remain visible when alignment confidence is lost;
- user/document metadata in routine logs without explicit privacy handling;
- executable or remote behavior in DSAS/DSAM data;
- a normal-install requirement to weaken SIP.

---

# Phase A — Build and test foundation

Goal: create a project structure in which every later macOS-specific capability can be developed test-first.

- [ ] **SD-001 — Create the macOS application project and first failing smoke test.** Red: add a test that imports the application/core module and fails because it does not exist. Green: create the minimal Xcode project/targets and make the test compile/pass. Done: a clean checkout can build and run the unit test from the command line.
- [ ] **SD-002 — Establish module boundaries.** Red: tests should demonstrate that domain types can be imported without importing AppKit/private platform code. Green: create project-owned modules/targets for domain/core, platform integration, rendering, and app presentation with one-way dependencies matching `docs/architecture.md`.
- [ ] **SD-003 — Add the default unit test plan.** Red: add a CI/local command that fails because no named unit test plan exists. Green: create an Xcode test plan containing deterministic Swift Testing targets only. Done: one documented command runs the fast suite.
- [ ] **SD-004 — Add system-probe and UI test plans.** Red: verification script/tests expect separately addressable `System` and `UI` plans. Green: add plans so platform probes and XCUI tests never accidentally run as part of the fast unit loop.
- [ ] **SD-005 — Add test tags/traits for platform-sensitive tests.** Red: a sample tagged test proves selection/exclusion behavior. Green: define common test tags such as `fixture`, `system`, `privateSPI`, `missionControl`, and `privacy` and document their intended use.
- [ ] **SD-006 — Add deterministic test-fixture support.** Red: a test expects fixture loading from the test bundle and fails. Green: provide a tiny fixture loader plus conventions for sanitized JSON/plist fixture files.
- [ ] **SD-007 — Add structured diagnostic logging abstraction.** Red: unit tests verify category/level and privacy annotations without writing private values. Green: wrap `Logger`/OSLog behind a project interface used by domain/application layers.
- [ ] **SD-008 — Add dependency composition root.** Red: a test builds the application coordinator from fake project protocols. Green: introduce explicit dependency containers/factories; no service locator or global mutable singleton.
- [ ] **SD-009 — Add basic CI for deterministic tests.** Red: repository validation fails without an automated test workflow. Green: add GitHub Actions/build automation that compiles and runs the fast deterministic suite on an available macOS runner; do not pretend private-SPI system probes are portable CI tests.
- [ ] **SD-010 — Add a test-support application fixture.** Red: tests expect metadata for a controlled sample `.app` fixture. Green: create the smallest test fixture/bundle needed to exercise bundle metadata, duplicate identifiers, icons/resources, and later DSAM discovery without depending on third-party apps.

# Phase B — Stable domain model

Goal: make SpaceDress's project-owned model testable before any private API code exists.

- [ ] **SD-011 — Implement `NativeSpaceReference`.** Red: parameterized tests cover source, kind, opaque value, equality, and timestamp/provenance behavior. Green: add the immutable domain type without assigning persistence semantics.
- [ ] **SD-012 — Implement `ProcessObservation`.** Red: tests cover PID plus optional executable/launch metadata and prove PID is runtime-only. Green: add the value type.
- [ ] **SD-013 — Implement `ApplicationIdentity`.** Red: tests show two apps with the same bundle identifier but different canonical bundle URLs remain distinct when required. Green: add bundle ID, canonical bundle URL, localized name, and optional team identity.
- [ ] **SD-014 — Implement `WindowObservation`.** Red: tests cover window ID, owner PID, raw bounds, title-presence metadata, and provenance without persisting sensitive title text by default. Green: add the type.
- [ ] **SD-015 — Implement normalized participant regions.** Red: parameterized geometry tests cover 50/50, 70/30, reversed participants, vertical arrangements, invalid/zero-size rectangles, and clamping. Green: add a normalized region type independent of AppKit coordinates.
- [ ] **SD-016 — Implement physical-side derivation.** Red: tests prove left/right/top/bottom/overlap/unknown derive from rectangle geometry and never collection order. Green: add the convenience classification.
- [ ] **SD-017 — Implement `SpaceParticipant`.** Red: tests cover one process/app with one or more windows and optional normalized region/role/confidence. Green: add the immutable participant type.
- [ ] **SD-018 — Implement `SpaceSnapshot`.** Red: tests cover desktop, single-fullscreen, split-fullscreen, and unknown snapshots with native references treated as observation data. Green: add snapshot, display association, participants, MC frame/position, confidence, and observation timestamp.
- [ ] **SD-019 — Implement capability-state model.** Red: tests cover available/degraded/unavailable/permission-denied/unsupported where relevant and verify features can declare required capabilities. Green: add typed capability status instead of booleans.
- [ ] **SD-020 — Implement confidence/provenance model.** Red: tests show ambiguous observations remain ambiguous and combining weak evidence cannot silently become certain. Green: add bounded confidence/provenance structures used by platform translators.
- [ ] **SD-021 — Implement snapshot reconciliation within a session.** Red: fixtures exercise PID continuity, native reference continuity, app identity, geometry, window IDs, and Mission Control reorder. Green: implement the documented signal priority without creating durable Space identity.
- [ ] **SD-022 — Prove reconciliation does not persist native identity.** Red: serialization/persistence tests fail if a native Space ID/UUID becomes a user-rule key. Green: enforce the separation in model APIs.
- [ ] **SD-023 — Implement privacy-safe snapshot descriptions.** Red: tests ensure debug/diagnostic descriptions omit document paths/titles unless explicitly redacted/opted in. Green: add sanitized diagnostic rendering.
- [ ] **SD-024 — Add domain fixture corpus.** Red: parameterized tests reference canonical single, split, duplicate-bundle, same-app-split, and ambiguous fixtures. Green: add sanitized fixtures that become regression inputs for later adapters.

# Phase C — Space topology provider

Goal: enumerate current Spaces and classify full-screen topology while isolating private APIs.

- [ ] **SD-025 — Define `SpaceTopologyProvider` contract.** Red: fake-provider tests drive a coordinator that consumes snapshots without knowing SkyLight/CGS details. Green: add the protocol and result/error types.
- [ ] **SD-026 — Implement private-symbol loader boundary.** Red: tests inject symbol-present/symbol-missing lookup tables and verify clean capability downgrade. Green: add a narrow dynamic loader; no private symbol escapes the platform module.
- [ ] **SD-027 — Wrap the connection/session primitive required by the selected CGS/SLS path.** Red: adapter tests verify lifecycle/error translation using injected function pointers. Green: implement minimal connection handling.
- [ ] **SD-028 — Enumerate raw Spaces for one display.** Red: fixture contract test starts from a captured raw payload and expects sanitized native references. Green: implement the first read-only enumeration path.
- [ ] **SD-029 — Enumerate Spaces across displays.** Red: fixtures cover multiple managed displays and duplicated/reordered Space positions. Green: associate each observation with display identity without using ordinal position as persistent identity.
- [ ] **SD-030 — Classify regular vs full-screen Space.** Red: fixtures cover known type values plus unknown future values. Green: classify only when evidence supports it and preserve unknown values for diagnostics.
- [ ] **SD-031 — Resolve owner PID arrays.** Red: fixture tests cover zero, one, two, duplicate, stale, and malformed owner PID results. Green: implement owner lookup with ambiguity preserved.
- [ ] **SD-032 — Build topology snapshots from topology + owner evidence.** Red: integration tests compose fake enumeration and owner providers into single/split/desktop snapshot skeletons. Green: implement translation into domain types.
- [ ] **SD-033 — Add live topology probe command/mode.** Red: a system test expects machine-readable output describing OS build, displays, Space refs/types, owner PIDs, and capabilities. Green: implement a diagnostics-only probe runnable without Mission Control.
- [ ] **SD-034 — Record and sanitize first live topology fixtures.** Red: contract tests fail until captured fixtures parse into expected snapshots. Green: capture representative current-macOS payloads and commit only sanitized data.
- [ ] **SD-035 — Test topology with inactive full-screen Spaces.** Red: system probe asserts the tool distinguishes active and inactive observed Spaces without assuming only current-space APIs are authoritative. Green: adapt provider/fallback strategy as required and document limitations.
- [ ] **SD-036 — Test topology across Space reorder.** Red: live/system test records before/after automatic/manual reorder and verifies logical app observations survive without position identity. Green: fix reconciliation/provider assumptions exposed by the probe.

# Phase D — Window membership and split geometry

Goal: identify the actual participant windows and their physical regions.

- [ ] **SD-037 — Define `SpaceWindowProvider` contract.** Red: fake-provider tests map a Space observation to window observations without importing CGWindow/SLS structures. Green: add protocol and errors/capabilities.
- [ ] **SD-038 — Enumerate window IDs for a Space.** Red: fixture tests cover one-window fullscreen, two-window tiled, nonparticipant utility windows, and unavailable membership. Green: implement selected read-only window-membership path.
- [ ] **SD-039 — Read window bounds by window ID.** Red: tests translate synthetic `CGWindowListCopyWindowInfo`/equivalent dictionaries into typed rectangles and reject malformed values. Green: implement supported public-window-info path where possible.
- [ ] **SD-040 — Join window IDs to owner PIDs.** Red: tests cover windows belonging to unexpected/helper processes and stale IDs. Green: correlate membership, bounds, and process ownership without trusting array index.
- [ ] **SD-041 — Compute participant regions from screen bounds.** Red: tests cover 50/50, 70/30, 30/70, top/bottom, gaps, overlap, Retina scale, and nonzero display origins. Green: normalize relative to the correct display/work area.
- [ ] **SD-042 — Derive split ordering from geometry.** Red: randomized tests shuffle all provider arrays and always produce the same physical participant ordering. Green: sort/classify solely from geometry.
- [ ] **SD-043 — Handle same-app split windows.** Red: tests prove two windows from the same bundle remain two participants when window IDs/regions differ. Green: construct participant identity at window/process level before app-style merging.
- [ ] **SD-044 — Handle unequal split ratios.** Red: parameterized tests verify ratio preservation through normalization. Green: retain actual rectangles rather than reducing geometry to a left/right enum.
- [ ] **SD-045 — Add live split-geometry probe.** Red: system test expects window ID, PID, raw bounds, normalized region, and derived side for each tiled participant. Green: implement probe output.
- [ ] **SD-046 — Verify current macOS split matrix.** Red: run/record 50/50, unequal ratio, swapped apps, same-app split, inactive Space, multiple display, and scaling cases; failing cases become explicit capability limitations. Green: fix supported paths or document fallback.
- [ ] **SD-047 — Add geometry fixture corpus.** Red: contract tests consume sanitized live outputs and assert stable translator behavior. Green: commit fixtures keyed by macOS build/configuration metadata.

# Phase E — Running application and document metadata

Goal: derive useful identity automatically from ordinary applications with no DSAM.

- [ ] **SD-048 — Define `ApplicationResolver` contract.** Red: tests resolve a PID observation into project-owned app identity/icon metadata through a fake resolver. Green: add protocol.
- [ ] **SD-049 — Resolve PID to `NSRunningApplication`.** Red: adapter tests cover valid, exited, inaccessible, and helper processes. Green: implement the supported AppKit lookup and typed failure.
- [ ] **SD-050 — Resolve the actual running bundle URL.** Red: tests using controlled app fixtures prove the resolver returns the launching bundle path rather than searching `/Applications`. Green: canonicalize and retain the actual URL.
- [ ] **SD-051 — Preserve duplicate bundle copies.** Red: fixture/system tests launch/correlate two app copies sharing a bundle identifier and assert different canonical bundle URLs. Green: fix identity semantics if needed.
- [ ] **SD-052 — Resolve localized app name and bundle metadata.** Red: tests cover localized display name fallback order and missing fields. Green: implement deterministic title derivation.
- [ ] **SD-053 — Resolve the system app icon.** Red: tests verify icon lookup is based on the actual bundle and produces a renderable image/fallback. Green: use system icon services rather than hard-coded `.icns` parsing.
- [ ] **SD-054 — Add optional code-sign/team identity.** Red: tests cover signed, unsigned/dev, and lookup-failure cases without making signing data mandatory. Green: add the narrow resolver only if needed for selector strengthening.
- [ ] **SD-055 — Define `DocumentResolver` contract.** Red: fake tests expose optional title/document identity while preserving privacy flags. Green: add the protocol independently of Accessibility implementation.
- [ ] **SD-056 — Resolve window/document title through Accessibility when permitted.** Red: adapter tests cover title present, absent, permission denied, app unsupported, and volatile titles. Green: implement optional extraction without treating title as durable identity.
- [ ] **SD-057 — Add document-title privacy tests.** Red: tests prove document titles do not enter routine logs, persisted diagnostics, or default user manifest. Green: enforce redaction/opt-in boundaries.

# Phase F — Mission Control observation and thumbnail geometry

Goal: know exactly when and where decoration may be drawn.

- [ ] **SD-058 — Define `MissionControlObserver` contract.** Red: state-machine tests consume open/close/frame-update events independent of AX implementation. Green: add protocol/event types.
- [ ] **SD-059 — Implement Mission Control lifecycle detector.** Red: injected AX/notification fixtures cover open, opening, open, closing, closed, duplicate notifications, and unexpected interruption. Green: implement the first reliable strategy.
- [ ] **SD-060 — Add lifecycle system probe.** Red: real-Mac test records a complete three-finger-swipe/keyboard Mission Control lifecycle and fails on unbalanced/stuck states. Green: adjust detector and record evidence.
- [ ] **SD-061 — Read thumbnail frames through Accessibility when available.** Red: AX fixture tests parse known element trees and reject wrong roles/frames. Green: implement exact-frame strategy behind the observer.
- [ ] **SD-062 — Correlate AX thumbnails with Space observations.** Red: fixtures cover reorder, full-screen app labels, split labels, regular desktops, and missing names without using labels as sole identity. Green: implement correlation with confidence.
- [ ] **SD-063 — Implement inferred thumbnail-layout fallback.** Red: pure geometry tests calculate frames from display size/count/native bar metrics for recorded cases. Green: add a versioned inference strategy isolated from exact AX strategy.
- [ ] **SD-064 — Fail closed on low frame confidence.** Red: state tests prove overlays are suppressed immediately when thumbnail correlation/frame confidence drops below policy. Green: implement confidence gating.
- [ ] **SD-065 — Handle Mission Control animation frame updates.** Red: synthetic event streams test interpolation/update frequency, reorder during animation, and cancellation. Green: publish frame updates without stale trailing overlays.
- [ ] **SD-066 — Handle multiple displays.** Red: fixture tests ensure thumbnail coordinates are associated with the correct display including nonzero origins and mixed scales. Green: implement per-display frame spaces.
- [ ] **SD-067 — Transform participant normalized regions into thumbnail coordinates.** Red: tests cover 50/50, 70/30, top/bottom, display origin, thumbnail inset/border, and rounding. Green: add pure transform function.
- [ ] **SD-068 — Add Mission Control geometry fixture corpus.** Red: contract tests consume sanitized AX/layout recordings for supported macOS builds. Green: commit fixtures plus configuration metadata.
- [ ] **SD-069 — Verify real thumbnail mapping matrix.** Red: system tests exercise one app fullscreen, two-app split, same-app split, reorder, inactive Spaces, multiple displays, and scaling. Green: update exact/inferred strategies or mark capability degraded where evidence requires.

# Phase G — Application baseline appearance

Goal: create immediately useful decoration with zero app adoption.

- [ ] **SD-070 — Define resolved appearance model.** Red: tests construct immutable border/tint/icon/text/image layers and prove rendering code need not know where values came from. Green: add project-owned `ResolvedAppearance`/layer types.
- [ ] **SD-071 — Define baseline appearance policy.** Red: tests expect app icon plus concise app title with safe defaults when no DSAM/user rule exists. Green: implement deterministic baseline generation.
- [ ] **SD-072 — Implement app-icon baseline.** Red: tests cover real icon, missing icon, corrupted/unrenderable icon, and generic fallback. Green: produce a resolved icon layer.
- [ ] **SD-073 — Implement app-title baseline.** Red: tests cover localized name, process name fallback, truncation metadata, and title suppression. Green: produce a resolved text layer.
- [ ] **SD-074 — Implement deterministic restrained accent derivation.** Red: image-fixture tests assert stable results, contrast bounds, and graceful fallback for monochrome/transparent icons. Green: add a small deterministic algorithm; do not let visual extraction block app identification.
- [ ] **SD-075 — Implement optional document-title baseline.** Red: policy tests cover global off/on, permission unavailable, privacy mode, empty/volatile title. Green: add document layer only when allowed.
- [ ] **SD-076 — Implement accessibility appearance policy.** Red: tests cover Increase Contrast, Reduce Transparency, Reduce Motion, color-only cues, and forced text/icon visibility. Green: transform resolved appearance according to system/user accessibility policy.
- [ ] **SD-077 — Implement baseline fallback ladder.** Red: property/parameterized tests remove metadata one source at a time and always yield a safe recognizable result or explicit no-decoration decision. Green: codify fallback order.

# Phase H — Render planning and overlay primitives

Goal: make all appearance decisions testable before drawing system windows.

- [ ] **SD-078 — Define pure `RenderPlan`.** Red: tests transform snapshot + resolved appearance + thumbnail frame into immutable drawing commands. Green: add render-plan types with no AppKit window ownership.
- [ ] **SD-079 — Plan border rendering.** Red: geometry tests cover inset/outset policy, corner radius, participant clipping, and whole-space user border. Green: emit border commands.
- [ ] **SD-080 — Plan tint rendering.** Red: tests cover opacity clamping, participant clipping, accessibility suppression, and stacking order. Green: emit tint commands.
- [ ] **SD-081 — Plan icon rendering.** Red: tests cover semantic sizes, all placements, split regions, clipping, backing scrim/material request, and tiny thumbnails. Green: emit icon commands.
- [ ] **SD-082 — Plan text rendering.** Red: tests cover app/document/user title selection, max lines, truncation, placements, split regions, contrast background, and privacy suppression. Green: emit text commands.
- [ ] **SD-083 — Plan bitmap rendering.** Red: tests cover fit/fill, participant clipping, opacity, local resource failure, and unsupported blend-mode fallback. Green: emit image commands.
- [ ] **SD-084 — Implement render-plan z-order.** Red: tests verify deterministic layer ordering and renderer safety overrides. Green: define composition order independent of AppKit view order.
- [ ] **SD-085 — Create transparent click-through overlay window primitive.** Red: AppKit integration test verifies borderless/nonactivating behavior, mouse pass-through, no key status, and hide/show control. Green: implement the minimal overlay surface.
- [ ] **SD-086 — Render a static `RenderPlan` into an overlay.** Red: snapshot/image test or integration test expects known layout output for a fixed plan. Green: implement drawing/views for the current layer vocabulary.
- [ ] **SD-087 — Update overlay without recreating window.** Red: integration test changes frame/layers repeatedly and asserts stable window identity plus updated content. Green: support cheap plan updates.
- [ ] **SD-088 — Remove overlay immediately on invalidation.** Red: state-machine integration test drops confidence/MC state and asserts no overlay remains. Green: wire invalidation to destruction/hide.
- [ ] **SD-089 — Add rendering snapshot fixtures.** Red: deterministic render tests compare geometry/semantic output for single and split examples. Green: establish regression fixtures without depending on live Mission Control screenshots.

# Phase I — First end-to-end single-fullscreen vertical slice

Goal: solve the original problem before building configuration complexity.

- [ ] **SD-090 — Build snapshot coordinator.** Red: integration tests feed fake topology/window/app/MC providers and expect one coherent current model. Green: implement cancellable orchestration and stale-result rejection.
- [ ] **SD-091 — Build appearance coordinator.** Red: integration tests turn an identified app participant into baseline `ResolvedAppearance`. Green: connect resolver and baseline policy.
- [ ] **SD-092 — Build overlay coordinator.** Red: fake-renderer tests verify open/update/close lifecycle for one fullscreen thumbnail and no drawing outside MC. Green: connect snapshots, frames, render plans, and overlay renderer.
- [ ] **SD-093 — Run real single-fullscreen acceptance test.** Red: system/UI procedure fails until a real full-screen app receives the correct icon/title decoration in Mission Control. Green: fix only defects in the vertical path.
- [ ] **SD-094 — Test multiple simultaneous full-screen Spaces.** Red: system test opens several dark-mode full-screen apps and asserts each observed Space maps to the correct app decoration. Green: fix correlation/reconciliation issues.
- [ ] **SD-095 — Test rapid Mission Control entry/exit.** Red: stress test repeatedly enters/exits Mission Control and asserts zero orphan overlays and no crash. Green: fix lifecycle/cancellation races.

# Phase J — Split/tiled vertical slice

Goal: make split full-screen a first-class production path, not an icon-pair hack.

- [ ] **SD-096 — Compose two participant baseline appearances.** Red: integration tests produce independent appearances clipped to two normalized regions. Green: add split composition.
- [ ] **SD-097 — Render physical left/right correctly.** Red: shuffle owner/window arrays in fixtures while expected left/right visual result stays unchanged. Green: rely only on participant geometry.
- [ ] **SD-098 — Preserve unequal split ratios in overlays.** Red: 70/30 and 30/70 fixture/render tests assert correct participant clipping. Green: remove any hidden 50/50 assumptions.
- [ ] **SD-099 — Support same-app split windows.** Red: tests require two participant regions/icons/titles even when bundle identity is shared. Green: retain window participant distinction.
- [ ] **SD-100 — Add neutral fallback when split geometry is unknown.** Red: tests assert paired identity without pretending which side belongs to which app. Green: implement confidence-aware fallback composition.
- [ ] **SD-101 — Run real split acceptance matrix.** Red: system tests cover 50/50, unequal split, swapped sides, same-app, inactive split Space, multiple displays. Green: fix supported behavior and record any capability degradation.

# Phase K — DSAS / DSAM reader

Goal: support the neutral application-facing standard without making app adoption necessary.

- [ ] **SD-102 — Generate/implement DSAM Codable model from the draft schema vocabulary.** Red: valid/invalid schema examples fail before decoder types exist. Green: decode the current Draft 0.1 model with explicit version handling.
- [ ] **SD-103 — Validate unknown manifest versions.** Red: tests expect unsupported versions to be ignored with diagnostics while baseline appearance survives. Green: implement version gate.
- [ ] **SD-104 — Implement macOS explicit DSAM discovery.** Red: test app fixture declares `DesktopSwitcherAppearanceManifest` and resolver expects the canonical resource path. Green: implement lookup from the actual running bundle.
- [ ] **SD-105 — Implement conventional DSAM fallback discovery.** Red: fixture with no Info.plist key but conventional resource path must resolve; missing DSAM must remain normal/non-error. Green: add fallback.
- [ ] **SD-106 — Enforce resource containment.** Red: tests cover `..`, symlinks escaping Resources, absolute paths, URLs, and valid nested local resources. Green: canonicalize and reject anything outside the approved root.
- [ ] **SD-107 — Resolve DSAM border/tint layers.** Red: tests merge declared fields over baseline and clamp unsafe values. Green: implement resolver translation.
- [ ] **SD-108 — Resolve DSAM icon layer.** Red: tests cover `bundleIcon`, valid resource icon, missing resource, malformed image, and baseline fallback. Green: implement safely.
- [ ] **SD-109 — Resolve DSAM text layer.** Red: tests cover appTitle/documentTitle/literal, privacy suppression, missing AX data, and split placement. Green: implement.
- [ ] **SD-110 — Resolve DSAM bitmap layer.** Red: tests cover local image, fit/fill, opacity, blend-mode fallback, and participant clipping. Green: implement.
- [ ] **SD-111 — Implement application baseline → DSAM precedence.** Red: merge tests prove omitted DSAM concerns preserve baseline while declared concerns refine only the app's participant. Green: implement deterministic application-authority resolution.
- [ ] **SD-112 — Add DSAM diagnostics.** Red: tests produce structured nonprivate diagnostics for unsupported version, invalid schema, rejected resource, and suppressed layer. Green: add diagnostics surfaced later in Settings.
- [ ] **SD-113 — Add DSAM validation command/tooling.** Red: CLI/tool tests validate bundled examples plus expected failures. Green: provide a developer-facing validator that does not require launching SpaceDress.
- [ ] **SD-114 — Run DSAS conformance fixtures through SpaceDress.** Red: every published example becomes an executable test fixture. Green: ensure prose/schema/examples/reader agree.

# Phase L — SpaceDress private multi-app user manifest

Goal: give users durable customization while keeping it outside DSAS.

- [ ] **SD-115 — Freeze internal user-manifest v1 only after model tests exist.** Red: tests describe required global preferences, presets, application rules, optional document rules, and resources. Green: choose the smallest Codable persistence shape and document that it is SpaceDress-private.
- [ ] **SD-116 — Implement atomic user-manifest load/save.** Red: tests cover missing file, valid file, interrupted write, malformed file, permissions failure, and backup/recovery. Green: use atomic replacement and explicit diagnostics.
- [ ] **SD-117 — Add manifest migration framework.** Red: fixture tests load a synthetic older version and migrate to current while preserving semantics. Green: add versioned migrations before real user data exists.
- [ ] **SD-118 — Implement application selector matching by bundle ID.** Red: tests cover exact match, absent identifier, and nonmatch. Green: add basic rule selector.
- [ ] **SD-119 — Add optional canonical bundle-path selector.** Red: tests distinguish two copies sharing a bundle identifier. Green: strengthen matching only when the rule specifies a path.
- [ ] **SD-120 — Add optional team/signing selector.** Red: tests cover matching, missing signing metadata, and explicit mismatch. Green: implement without making signing mandatory.
- [ ] **SD-121 — Implement preset model.** Red: tests create reusable appearance presets independent of app selectors. Green: add preset storage and reference validation.
- [ ] **SD-122 — Implement per-app override merge.** Red: precedence tests prove `baseline → DSAM → user rule` and verify user suppression/removal as well as replacement. Green: implement field/layer merge semantics.
- [ ] **SD-123 — Implement global privacy preferences.** Red: tests show document titles/resources can be globally disabled after all lower authorities resolve. Green: add final user-policy layer.
- [ ] **SD-124 — Implement global accessibility preferences.** Red: tests combine user choices with current system accessibility state without allowing user config to defeat required safety behavior. Green: add policy merge.
- [ ] **SD-125 — Implement local custom user resources.** Red: tests cover valid imported image, missing file, path traversal/escape, replacement, and deletion. Green: copy/manage resources inside a SpaceDress-owned directory rather than storing arbitrary external paths where practical.
- [ ] **SD-126 — Implement per-document rule identity only when stable.** Red: tests reject volatile window title as durable identity and accept explicitly supported stable document URL/identifier paths under privacy controls. Green: add optional document selector boundary.
- [ ] **SD-127 — Implement import/export of SpaceDress user configuration.** Red: tests round-trip manifest plus owned resources and reject malformed/unsafe archives. Green: add explicit export format without claiming DSAS compatibility.
- [ ] **SD-128 — Add user-config corruption recovery tests.** Red: fuzz/fixture tests cover truncated JSON, unknown fields, missing presets/resources, duplicate rule IDs, and migration failures. Green: recover to safe defaults while preserving recoverable user data.

# Phase M — Settings and configuration UI

Goal: let a nontechnical user inspect detected apps and customize them without editing JSON.

- [ ] **SD-129 — Add menu-bar/status entry and Settings window shell.** Red: XCUI test launches the app, opens Settings from the status item/app command, and verifies a stable accessibility identifier. Green: implement the minimal shell.
- [ ] **SD-130 — Add permission/status overview.** Red: UI tests inject capability states and verify clear available/degraded/permission-denied presentation. Green: add status view without exposing private-SPI jargon as the primary user message.
- [ ] **SD-131 — Add observed applications list.** Red: view-model tests and XCUI fixture mode show running/fullscreen apps with actual icon/name and duplicate-copy distinction. Green: implement list from project models.
- [ ] **SD-132 — Add per-app enable/disable control.** Red: UI → manifest integration test toggles one app and verifies persisted rule plus resolver result. Green: implement control.
- [ ] **SD-133 — Add title-source controls.** Red: tests select app title, document title, user title, or hidden and verify resolved appearance/privacy behavior. Green: implement editor section.
- [ ] **SD-134 — Add border editor.** Red: view-model tests cover color/weight/reset and generate the expected user override. Green: implement UI plus live preview model.
- [ ] **SD-135 — Add tint/accent editor.** Red: tests cover derived/default/custom/none and opacity safety clamping. Green: implement.
- [ ] **SD-136 — Add icon editor.** Red: tests cover app icon, custom imported resource, reset, missing resource recovery, and preview. Green: implement.
- [ ] **SD-137 — Add bitmap overlay editor.** Red: tests cover import, fit/fill, opacity, remove, and invalid asset. Green: implement with owned-resource handling.
- [ ] **SD-138 — Add style preset management.** Red: UI/view-model tests create, rename, duplicate, apply, update, and delete presets while preserving rules. Green: implement.
- [ ] **SD-139 — Add split preview.** Red: deterministic preview tests show two participants, swapped sides, unequal ratio, and same-app split. Green: implement preview using the same `RenderPlan` path as Mission Control.
- [ ] **SD-140 — Add diagnostics view.** Red: tests inject DSAM errors/capability degradation and verify useful sanitized explanations/copyable report. Green: implement.
- [ ] **SD-141 — Add global privacy/accessibility settings.** Red: UI integration tests persist settings and alter resolver output. Green: implement global controls.
- [ ] **SD-142 — Add import/export UI.** Red: XCUI tests exercise successful round trip plus invalid import failure without losing current config. Green: wire existing config service.
- [ ] **SD-143 — Add reset/recovery workflows.** Red: UI tests reset one app, one preset, or all user config with confirmation and verify baseline appearance remains. Green: implement.

# Phase N — Permissions, lifecycle, and normal macOS behavior

Goal: make the app behave like a trustworthy background macOS utility.

- [ ] **SD-144 — Implement Accessibility permission state provider.** Red: adapter tests cover notDetermined/denied/granted/change-at-runtime states. Green: provide typed state and recovery action without repeatedly nagging.
- [ ] **SD-145 — Request Accessibility only when needed.** Red: launch tests verify no unnecessary repeated prompt and that features degrade correctly before consent. Green: implement onboarding/request path.
- [ ] **SD-146 — Determine whether Screen Recording is actually required.** Red: system probe attempts all production functionality without it; only if a concrete feature fails for this reason should a permission dependency be introduced. Green: document result and keep permission absent unless justified.
- [ ] **SD-147 — Add launch-at-login support.** Red: service/view-model tests cover enable, disable, unavailable/error, and current-state refresh. Green: use current supported macOS mechanism.
- [ ] **SD-148 — Handle app relaunch with restored full-screen apps.** Red: system test restarts SpaceDress while several fullscreen/split Spaces exist and expects correct re-observation without native-ID persistence. Green: fix startup reconciliation.
- [ ] **SD-149 — Handle sleep/wake.** Red: lifecycle integration/system test suspends/resumes providers and verifies stale overlays/data are discarded and rebuilt. Green: implement reset/reprobe.
- [ ] **SD-150 — Handle display hot-plug/reconfiguration.** Red: fake/system tests change display set/origins/scales and assert all geometry is invalidated/recomputed. Green: wire display notifications to snapshot refresh.
- [ ] **SD-151 — Handle Space/app/window destruction races.** Red: event-sequence tests terminate an app/remove a Space while Mission Control is opening and assert no crash/wrong overlay. Green: make coordinator cancellation/idempotence robust.
- [ ] **SD-152 — Handle permission revocation at runtime.** Red: tests transition AX capability from available to denied while overlays/settings are active and expect safe degradation. Green: implement live capability refresh.
- [ ] **SD-153 — Add user-visible degraded-mode messaging.** Red: view-model tests map each capability failure to a concise actionable status without blocking unrelated features. Green: implement.

# Phase O — Performance and correctness hardening

Goal: make frequent Mission Control use feel native and boring.

- [ ] **SD-154 — Establish Mission Control response performance baseline.** Red: XCTest performance test measures observation-to-render-plan and overlay-update latency against an initial documented budget. Green: optimize only enough to meet the budget.
- [ ] **SD-155 — Establish idle CPU/memory baseline.** Red: performance/system test measures the background app while Mission Control is closed and flags continuous polling/leaks. Green: move to event-driven/cached behavior as needed.
- [ ] **SD-156 — Stress rapid open/close for leak/orphan detection.** Red: repeated Mission Control cycles assert bounded overlay/window/object counts. Green: fix ownership/lifecycle leaks.
- [ ] **SD-157 — Stress Space creation/destruction/reorder.** Red: system test repeatedly adds/removes/reorders fullscreen Spaces and verifies reconciliation never labels a known Space with the wrong app. Green: fix stale state races.
- [ ] **SD-158 — Stress split divider movement.** Red: system test changes split ratio repeatedly and expects participant decoration to track without crossing participants. Green: optimize geometry refresh.
- [ ] **SD-159 — Stress multi-display mixed scaling.** Red: system test covers display origins/scales/rearrangement and asserts pixel/frame mapping remains correct or overlays fail closed. Green: fix coordinate-space bugs.
- [ ] **SD-160 — Add malformed platform-data fuzz/property tests.** Red: generated dictionaries/arrays/NaNs/extreme rectangles/private-payload variations must never crash translators. Green: harden parsing and capability downgrade.
- [ ] **SD-161 — Add malformed DSAM/user-manifest fuzz tests.** Red: generated invalid/deep/oversized data must be rejected within bounded resources. Green: add size/depth/resource limits.
- [ ] **SD-162 — Add privacy audit tests for diagnostics/export.** Red: fixtures containing sensitive titles/paths must not appear in default logs, crash-style reports, or diagnostics copy unless explicitly included/redacted. Green: close leaks.

# Phase P — Compatibility and supported-platform contract

Goal: convert private-API uncertainty into an explicit tested support matrix.

- [ ] **SD-163 — Define compatibility-report format.** Red: tests validate a machine-readable report containing app version, macOS version/build, architecture, displays, relevant Mission Control settings, permissions, SIP state, and capability results. Green: implement report generator.
- [ ] **SD-164 — Automate current-macOS probe suite.** Red: one command runs topology, owner, window geometry, MC lifecycle/frame, overlay, and split acceptance probes and produces a report. Green: compose existing system tests without duplicating logic.
- [ ] **SD-165 — Run supported-version matrix.** Red: each intended macOS release must produce a stored compatibility result; failures either get fixed or explicitly remove/degrade support. Green: decide the initial minimum supported macOS version from evidence rather than convenience.
- [ ] **SD-166 — Add per-version adapter/capability selection.** Red: fixture tests for multiple OS builds choose the correct exact/fallback strategy and unknown future versions fail closed. Green: centralize version-sensitive selection.
- [ ] **SD-167 — Add new-macOS safety gate.** Red: synthetic future major version test verifies undocumented capabilities can start degraded until probes establish compatibility while baseline app operation/settings still launch. Green: implement policy.

# Phase Q — Release engineering

Goal: produce a signed, notarized, reproducible public build with honest documentation.

- [ ] **SD-168 — Finalize bundle identifier, entitlements, and signing configuration.** Red: build validation test/script checks intended entitlements and rejects unexpected additions. Green: commit production configuration without secrets.
- [ ] **SD-169 — Add Release test plan.** Red: release command fails unless deterministic unit/fixture tests, selected integration tests, UI smoke tests, schema examples, and packaging checks all pass. Green: compose the plan.
- [ ] **SD-170 — Add archive build automation.** Red: script test/dry run expects a release archive from a clean checkout. Green: create repeatable `xcodebuild archive` workflow.
- [ ] **SD-171 — Add Developer ID signing and notarization workflow.** Red: CI/local release workflow validates missing credentials clearly and verifies signature/notarization when credentials exist. Green: automate without committing credentials.
- [ ] **SD-172 — Add distributable package/disk image.** Red: packaging test mounts/inspects output and verifies app identity, signature, license/notice where applicable, and no development artifacts. Green: create reproducible distribution artifact.
- [ ] **SD-173 — Add release checksums and provenance metadata.** Red: release script expects SHA-256 and build/version/commit metadata. Green: generate alongside artifact.
- [ ] **SD-174 — Add first-run onboarding.** Red: XCUI tests cover first launch, explanation of purpose, required permission request, skipped/denied permission, and route to Settings. Green: implement minimal onboarding.
- [ ] **SD-175 — Add end-user README/install/troubleshooting documentation.** Red: documentation checklist/test verifies install, permissions, supported macOS, known limitations, uninstall/config location, privacy, and issue-report instructions exist. Green: write/update docs.
- [ ] **SD-176 — Add release changelog generation/check.** Red: release workflow fails if user-visible changes/support changes are undocumented. Green: enforce changelog/release-note input.
- [ ] **SD-177 — Produce release candidate and run clean-machine acceptance.** Red: acceptance checklist on a clean user account verifies install, Gatekeeper, permission flow, single fullscreen, split fullscreen, settings persistence, restart, and uninstall. Green: fix release blockers only.
- [ ] **SD-178 — Publish v1.0 only after acceptance matrix is green.** Red: release checklist blocks publication without signed/notarized artifact, checksums, compatibility report, known issues, and DSAS version support statement. Green: create the GitHub Release and tag.

# Phase R — Secondary regular Desktop Space support

Goal: give regular desktops useful treatment without weakening the full-screen identity model. This phase may ship after the first full-screen-focused release.

- [ ] **SD-179 — Model regular Desktop participant sets.** Red: tests cover zero/one/many windows/apps with no invented owner. Green: extend snapshot composition without persistent desktop index identity.
- [ ] **SD-180 — Derive representative app-set summary.** Red: tests choose deterministic representative icons/titles from a multi-app desktop while avoiding misleading ownership. Green: add secondary baseline policy.
- [ ] **SD-181 — Add user-defined Desktop title rule model.** Red: tests persist a logical user desktop record without keying it to native Space ID/index. Green: add fuzzy/reconciliation-ready internal identity only if evidence is sufficient.
- [ ] **SD-182 — Add Desktop Mission Control decoration.** Red: render/integration tests apply title/app-summary decoration without changing fullscreen behavior. Green: enable behind separate capability/setting.
- [ ] **SD-183 — Add Desktop reconciliation experiments.** Red: system tests cover reorder/reboot/recreated desktops and explicitly quantify when logical mapping is ambiguous. Green: fail closed or require user remapping rather than silently attaching a title to the wrong desktop.

# Phase S — DSAS stewardship and interoperability polish

Goal: make the public standard independently consumable after SpaceDress itself is proven.

- [ ] **SD-184 — Verify DSAS naming and repository separation.** Red: repository search/check fails if app-facing standard docs/schema use SpaceDress-specific wire-format names or internal user-manifest fields. Green: clean remaining coupling.
- [ ] **SD-185 — Publish renderer conformance tests.** Red: extract DSAS valid/invalid/semantic fixtures that can run without SpaceDress internals. Green: provide a small conformance corpus for other implementations.
- [ ] **SD-186 — Publish app-author integration guide.** Red: documentation checklist requires macOS bundle discovery, schema reference, examples, resource constraints, split semantics, and graceful no-reader behavior. Green: add neutral DSAS documentation.
- [ ] **SD-187 — Prove alternate-reader independence.** Red: implement or sketch a tiny standalone test reader/tool that consumes DSAM without importing SpaceDress user configuration/domain machinery. Green: resolve specification coupling discovered by the exercise.
- [ ] **SD-188 — Decide DSAS 1.0 only after implementation evidence.** Red: stabilization checklist requires implemented layer semantics, conformance fixtures, at least one real DSAM test app, compatibility policy, and documented change process. Green: either publish 1.0 or explicitly retain Draft status.

---

# Cross-cutting acceptance scenarios

The following scenarios are not separate implementation slices; they are regression expectations that should become automated at the earliest phase capable of expressing them.

- **Many dark full-screen apps:** Mission Control remains immediately distinguishable using derived icon/title decoration with no DSAMs installed.
- **Duplicate app copies:** two running copies with the same bundle identifier but different bundle paths can receive distinct user styling.
- **Same app, multiple full-screen windows:** each Space is correctly associated with its current window context while sharing app-level defaults.
- **Two-app split:** both identities render in the correct physical participant regions.
- **Same-app split:** two participant regions remain visible and do not collapse into one owner.
- **Unequal split:** 70/30 remains 70/30 in thumbnail clipping/placement.
- **Array-order chaos:** randomizing native owner/window enumeration order never swaps left/right decoration.
- **Mission Control reorder:** native thumbnail/order changes never reassign persistent user rules.
- **SpaceDress restart/reboot:** current Spaces are reconstructed from live logical identity; stale native IDs cannot orphan or misapply styles.
- **Permission denial/revocation:** document-title/AX-dependent features degrade while safe baseline features continue where technically possible.
- **Private-SPI disappearance:** affected capability turns unavailable/degraded; the app does not crash and never draws confidently wrong overlays.
- **Alignment uncertainty:** decoration disappears rather than drifting onto the wrong native thumbnail.
- **No app adoption:** the application is useful on day one with zero DSAMs in the ecosystem.
- **DSAM present:** app preference refines only that app's representation and never overrides explicit user policy.
- **User authority:** SpaceDress private configuration always wins over application baseline/DSAM within safety/accessibility limits.
- **Privacy:** document/window titles and local paths remain local and absent from routine logs/reports by default.
- **SIP:** every supported normal-use acceptance scenario runs with SIP enabled.

# Completion criteria for the application

SpaceDress is implementation-complete for its first production release when:

1. SD-001 through SD-178 are checked;
2. the single-fullscreen and split/tiled acceptance matrices pass on every declared supported macOS release;
3. the app remains useful with no DSAMs installed;
4. user configuration survives restart and app/system re-observation without native Space identity;
5. wrong/alignment-uncertain decoration fails closed;
6. required permissions are minimized and clearly explained;
7. the release artifact is signed, notarized, tested on a clean account, and accompanied by checksums/compatibility notes;
8. DSAS support is accurately labeled Draft or stable according to the evidence at release time.

SD-179 onward represents secondary Desktop support and standards-stewardship work that can continue after the fullscreen-focused application reaches production quality.
