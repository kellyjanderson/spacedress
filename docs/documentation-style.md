# Documentation style

SpaceDress depends on undocumented platform behavior, so documentation quality is part of technical correctness.

## Write claims at the right confidence level

Use these categories deliberately.

### Contract

A public API/specification guarantee or a SpaceDress invariant.

> SpaceDress Style Manifest v0.1 does not allow remote resources.

### Observation

Something reproduced in a stated environment.

> On macOS 26.x in the tested configuration, the managed-space payload contained ...

### Inference

A conclusion supported by observations but not guaranteed by Apple.

> This field appears to represent ...

### Hypothesis

An explanation waiting for a discriminating test.

> The overlay drift may be caused by ...

Do not rewrite observations as contracts because they have remained true for several releases.

## Platform research notes

A useful note contains:

```text
Date:
macOS version/build:
Hardware:
Displays:
Mission Control settings:
Permissions:
SIP state:
Space configuration:
Interface/symbol tested:
Procedure:
Raw observation:
Interpretation:
Confidence:
Next falsifying test:
```

Sanitize private paths, titles, user names, and screenshots before committing fixtures.

## Voice

- Be direct.
- Prefer concrete nouns and verbs.
- State limitations next to the claim they limit.
- Avoid marketing superlatives in technical documents.
- Use the clothing metaphor for memorable navigation, not as a substitute for technical vocabulary.

## Terminology

Use these terms consistently:

| Term | Meaning |
| --- | --- |
| Space | A macOS Mission Control Space in the general sense |
| full-screen Space | A Space created/managed for native full-screen content |
| split/tiled full-screen Space | A full-screen Space with two app/window participants |
| Desktop Space | A regular multi-window desktop Space |
| participant | An app/process/window group contributing to a Space |
| native reference | Any OS-provided Space ID, UUID, index, or equivalent observation |
| title | A textual name displayed to identify content |
| style | Declarative visual decoration |
| manifest | Versioned data describing an app-preferred style |
| chrome | Decoration around/on top of the native thumbnail, not the thumbnail content itself |

Use `SpaceDress` with a capital S and D for the product/standard. Use `spacedress` only where an identifier/path convention requires lowercase.

## Markdown

- Start each document with one H1.
- Keep headings descriptive enough to work in GitHub's outline.
- Use tables for comparison, not prose layout.
- Use fenced code blocks for schemas, command output, and conceptual structures.
- Relative links are preferred for repository files.
- Link to primary sources for platform claims when available.

## Decisions vs research

Research documents can change as evidence changes.

An ADR records a project decision even if the underlying platform later changes. If new evidence invalidates the decision, supersede the ADR rather than silently rewriting history.

## Changelog-worthy documentation

Update user-facing docs with the same PR that changes behavior. A release note should not be the first place a behavior contract is written down.
