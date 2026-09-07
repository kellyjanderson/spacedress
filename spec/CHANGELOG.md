# SpaceDress Style Manifest changelog

The manifest standard is versioned independently of the SpaceDress application.

## 0.1 — Draft

Initial experimental specification:

- app-bundle discovery through `SpaceDressStyleManifest` or the conventional `Contents/Resources/SpaceDress/manifest.json` path;
- ordered declarative layers;
- border and tint layers;
- icon layer using actual bundle icon or local resource;
- text layer using app title, document title, or literal text;
- local bitmap image layer;
- semantic placement and sizing;
- split/tiled participant clipping rules;
- local-resource containment requirements;
- optional namespaced extension data.
