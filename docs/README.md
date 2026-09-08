# SpaceDress documentation

This directory is the project's design record. The application is expected to change faster than the principles and compatibility contracts described here.

## Start here

1. [`vision.md`](vision.md) — what SpaceDress is trying to accomplish.
2. [`architecture.md`](architecture.md) — component boundaries and dependency direction.
3. [`space-model.md`](space-model.md) — how runtime Spaces, apps, and persistent intent are represented.
4. [`user-configuration-model.md`](user-configuration-model.md) — SpaceDress's internal multi-app customization model and its boundary from the public standard.
5. [`style-guide.md`](style-guide.md) — visual and interaction rules.
6. [`code-style.md`](code-style.md) — engineering conventions.
7. [`documentation-style.md`](documentation-style.md) — how project knowledge is written and maintained.

## Platform research

- [`research/macos-integration.md`](research/macos-integration.md) — current integration strategy, API tiers, split geometry, and uncertainty rules.
- [`research/prior-art.md`](research/prior-art.md) — relevant macOS projects and what they demonstrate.

Research documents describe evidence and current hypotheses. They are not automatically architectural commitments.

## Decisions

Accepted architecture decisions live in [`decisions/`](decisions/). ADRs exist so that a future contributor can tell the difference between an intentional constraint and an accident of the first implementation.

## Public standard

The **Desktop Switcher Appearance Specification (DSAS)** is maintained under [`../spec/`](../spec/). A conforming application document is a **Desktop Switcher Appearance Manifest (DSAM)**.

DSAS is application-facing and implementation-neutral. SpaceDress's own multi-app user configuration is deliberately outside the specification.

## Maintainers

Repository configuration and release procedures are under [`maintainers/`](maintainers/).
