# 0004 — Declarative Application Appearance Manifest

**Status:** Accepted  
**Date:** 2026-09-07

## Context

SpaceDress can derive a useful appearance from ordinary application metadata, so an app-specific manifest must never be required for the product to work.

Applications may nevertheless know how they want to be represented better than a generic renderer can infer. A future interoperability format should let an application optionally refine its own switcher appearance.

An extension/plugin API with executable code would create security, compatibility, signing, sandbox, and lifecycle problems disproportionate to the task.

The public format must also remain independent from SpaceDress's private multi-app user configuration.

## Decision

The application-facing manifest is declarative, versioned data defined by the **Desktop Switcher Appearance Specification (DSAS)**. A document conforming to it is a **Desktop Switcher Appearance Manifest (DSAM)**.

SpaceDress is an implementation of DSAS, not part of the standard's name or wire vocabulary.

The base format may describe visual layers such as:

- border;
- tint;
- app icon;
- text sourced from app/document/user-visible metadata;
- local bitmap resources.

It may not execute:

- scripts;
- shell commands;
- JavaScript;
- dynamic libraries;
- app-supplied callbacks;
- remote code;
- arbitrary URL handlers.

Base manifest resources must resolve within the approved local application resource root.

A DSAM supplements or refines application-derived defaults. Its absence is normal.

SpaceDress user preferences and its internal multi-app manifest override application-side appearance. The internal user manifest is **not part of DSAS**.

## Consequences

- Applications require no SpaceDress integration for useful defaults.
- Manifests can be validated before rendering.
- Alternate renderers can implement DSAS without adopting SpaceDress branding or configuration semantics.
- Apps cannot use appearance metadata as a code-execution channel.
- SpaceDress remains free to evolve its multi-app user configuration independently.
- Some sophisticated dynamic ideas will require new declarative primitives rather than arbitrary callbacks.

## Alternatives considered

### SpaceDress-branded public manifest

Rejected. It couples an interoperability contract to one implementation and creates the same naming/ownership problem that mature standards often later need to unwind.

### Use the public manifest as SpaceDress's user rule database

Rejected. Application self-description and multi-app user targeting have different trust, identity, precedence, and lifecycle requirements.

### Plugin bundle API

Rejected because it solves a much larger extensibility problem than the project needs.

### CSS/HTML renderer

Rejected for v0.1 because web rendering semantics, remote-resource behavior, and arbitrary layout complexity would make the security/compatibility surface unnecessarily large.

## Revisit when

A concrete application-facing requirement cannot be expressed declaratively and adding a bounded primitive would be worse than introducing a more capable extension mechanism.
