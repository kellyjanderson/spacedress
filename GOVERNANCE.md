# Governance

SpaceDress currently uses a **maintainer-led open development** model.

## Maintainer

The upstream project maintainer is Kelly Eldritch (`@kellyjanderson`). The maintainer has final responsibility for releases, repository administration, security response, specification versioning, and resolving design deadlocks.

## Decision making

Routine changes are decided through pull-request review.

Material architectural or compatibility decisions should be recorded as Architecture Decision Records in [`docs/decisions/`](docs/decisions/). A decision should state the context, decision, consequences, and alternatives rather than merely recording who preferred what.

The strongest argument is reproducible evidence. Popularity, contributor seniority, and implementation effort may matter, but they do not turn an unverified platform assumption into a fact.

## Public standard stewardship

The SpaceDress Style Manifest is intended to be usable independently of the SpaceDress application. Changes therefore carry a higher compatibility obligation than internal implementation changes.

Before a stable `1.0` manifest:

- experimentation is expected;
- changes should still document migration impact;
- readers should ignore unknown optional fields when safe.

After `1.0`:

- compatible additions stay within the major version;
- incompatible semantic changes require a new major version;
- deprecated behavior receives a documented transition period where practical.

## Becoming a maintainer

As the project grows, maintainership may be extended to contributors who repeatedly demonstrate:

- technical judgment;
- careful handling of undocumented macOS behavior;
- reliable review;
- respect for compatibility and privacy;
- willingness to maintain work after merge.

This document should be revised before the project needs a more formal multi-maintainer voting or release model.
