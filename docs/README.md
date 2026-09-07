# SpaceDress documentation

This directory is the project's design record. The application is expected to change faster than the principles and compatibility contracts described here.

## Start here

1. [`vision.md`](vision.md) — what SpaceDress is trying to accomplish.
2. [`architecture.md`](architecture.md) — component boundaries and dependency direction.
3. [`space-model.md`](space-model.md) — how runtime Spaces, apps, and persistent intent are represented.
4. [`style-guide.md`](style-guide.md) — visual and interaction rules.
5. [`code-style.md`](code-style.md) — engineering conventions.
6. [`documentation-style.md`](documentation-style.md) — how project knowledge is written and maintained.

## Platform research

- [`research/macos-integration.md`](research/macos-integration.md) — current integration strategy, API tiers, and uncertainty rules.
- [`research/prior-art.md`](research/prior-art.md) — relevant macOS projects and what they demonstrate.

Research documents describe evidence and current hypotheses. They are not automatically architectural commitments.

## Decisions

Accepted architecture decisions live in [`decisions/`](decisions/). ADRs exist so that a future contributor can tell the difference between an intentional constraint and an accident of the first implementation.

## Public standard

The SpaceDress Style Manifest is maintained under [`../spec/`](../spec/). The specification is designed to remain useful even to alternate renderers and other tools.

## Maintainers

Repository configuration and release procedures are under [`maintainers/`](maintainers/).
