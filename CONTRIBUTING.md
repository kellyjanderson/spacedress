# Contributing to SpaceDress

Contributions are welcome, especially reproducible macOS research, compatibility reports, design criticism, DSAS proposals, and narrowly scoped implementation work.

SpaceDress is still defining its foundations. A small experiment that invalidates an assumption can be more valuable than a large patch built on the wrong one.

## Before contributing

Read, in this order:

1. [`docs/vision.md`](docs/vision.md)
2. [`docs/architecture.md`](docs/architecture.md)
3. [`docs/space-model.md`](docs/space-model.md)
4. [`docs/user-configuration-model.md`](docs/user-configuration-model.md)
5. accepted decisions in [`docs/decisions/`](docs/decisions/)
6. [`docs/code-style.md`](docs/code-style.md) for code changes
7. [`spec/README.md`](spec/README.md) for Desktop Switcher Appearance Specification changes

The short version:

- full-screen Spaces are the primary product;
- native Space IDs are runtime observations, not durable identity;
- split participant geometry comes from window bounds, not owner-array order;
- no normal feature may require disabling or weakening SIP;
- private macOS SPI belongs behind isolated adapters;
- SpaceDress derives useful appearance without app integration;
- DSAS/DSAM are neutral application-facing interoperability, not SpaceDress's user configuration;
- app-provided DSAM data is declarative and local;
- claims about undocumented macOS behavior need evidence.

## Choosing the right contribution path

### Bug

Use the bug form when a defined SpaceDress behavior is incorrect.

### macOS compatibility report

Use the compatibility form when behavior changes across macOS versions, display configurations, full-screen modes, or Mission Control settings. These reports are first-class project evidence.

For split/tiled behavior, include participant window IDs/bounds when the diagnostic tooling supports them.

### Feature or design proposal

Describe the problem first. A feature proposal should explain why it belongs in SpaceDress rather than merely showing that it can be built.

### DSAS proposal

Changes to the public Desktop Switcher Appearance Specification need:

- a concrete **application-facing** interoperability use case;
- an example DSAM;
- compatibility behavior for older readers;
- security/privacy implications;
- a clear reason an existing layer or field cannot express the requirement;
- an explanation of why the feature belongs in DSAS rather than SpaceDress's internal multi-app user configuration.

Once the specification reaches a stable release, incompatible changes require a new major specification version.

## Research quality

Undocumented macOS behavior is evidence-sensitive. A useful research note records:

- exact macOS version and build when possible;
- hardware architecture;
- relevant Mission Control settings;
- single display vs multiple displays;
- whether "Displays have separate Spaces" is enabled;
- whether automatic Space rearrangement is enabled;
- single full-screen vs split/tiled full-screen;
- API/SPI symbol or data source tested;
- observed values, not just conclusions;
- participant window IDs, bounds, and physical arrangement for split tests;
- what permissions were granted;
- whether SIP remained enabled.

Prefer a small repeatable probe over a screenshot of a conclusion.

## Pull requests

Keep each pull request coherent. A PR should have one primary reason to exist.

Include:

- the problem or decision being addressed;
- evidence for platform-sensitive changes;
- tests or a manual verification matrix;
- documentation updates where behavior or architecture changes;
- screenshots/video for visual behavior when useful;
- privacy/security consequences.

Use draft PRs for experiments that are useful to share but not yet ready to merge.

## Architecture changes

Changes that alter a core invariant should add or update an Architecture Decision Record (ADR). Examples:

- changing Space identity semantics;
- changing split participant geometry semantics;
- introducing a new private framework dependency;
- allowing a new class of DSAM behavior;
- moving SpaceDress-specific user targeting into DSAS;
- changing the no-SIP policy;
- adding networked functionality;
- changing the DSAS compatibility contract.

See [`docs/decisions/README.md`](docs/decisions/README.md).

## Code expectations

Follow [`docs/code-style.md`](docs/code-style.md). In particular:

- keep private APIs out of domain code;
- fail by capability rather than crashing when an SPI disappears;
- keep UI work on the main actor and platform discovery off it where practical;
- use structured logging;
- do not log document titles or paths at normal log levels unless explicitly required and redacted;
- prefer standard-library and Apple frameworks before adding dependencies.

## Documentation expectations

Documentation is part of the product. Follow [`docs/documentation-style.md`](docs/documentation-style.md).

A factual statement about private macOS behavior should point to a probe, source reference, or clearly labeled observation. Do not turn a plausible inference into project folklore.

## Commit style

Use an imperative, scoped subject when practical:

```text
Document fullscreen Space identity model
Define DSAS border layer schema
Probe tiled-space participant geometry on macOS 26
```

Keep mechanical formatting separate from semantic changes when that makes review easier.

## AI-assisted contributions

AI tools are allowed. The submitting human remains responsible for correctness, licensing, security, and reviewability.

For substantial AI-assisted code or research, note the assistance in the PR description when it would help reviewers understand how the work was produced. Do not cite an AI system as an author or use generated claims about private APIs without verification.

## Licensing contributions

Unless explicitly stated otherwise, contributions intentionally submitted for inclusion in SpaceDress are provided under the repository's Apache License 2.0, consistent with Section 5 of that license.
