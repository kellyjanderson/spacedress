# 0004 — Declarative Style Manifest

**Status:** Accepted  
**Date:** 2026-09-07

## Context

Applications can provide much better self-identification than a generic tool can infer. SpaceDress should therefore publish a standard that lets an app include preferred thumbnail styling in its own bundle.

An extension/plugin API with executable code would create security, compatibility, signing, sandbox, and lifecycle problems disproportionate to the task.

## Decision

The SpaceDress Style Manifest is declarative, versioned data.

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

Base manifest resources must resolve within the approved local style/app resource root.

User preferences override app-provided styling.

## Consequences

- Manifests can be validated before rendering.
- Alternate renderers can implement the standard without loading app code.
- Apps cannot use styling as a code-execution channel.
- Some sophisticated dynamic ideas will require new declarative primitives rather than arbitrary callbacks.

## Alternatives considered

### Plugin bundle API

Rejected because it solves a much larger extensibility problem than SpaceDress needs.

### CSS/HTML renderer

Rejected for v0.1 because web rendering semantics, remote-resource behavior, and arbitrary layout complexity would make the security/compatibility surface unnecessarily large.

## Revisit when

A concrete styling requirement cannot be expressed declaratively and adding a bounded primitive would be worse than introducing a more capable extension mechanism.
