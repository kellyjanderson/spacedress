# Release process

This is a placeholder contract for future releases. It intentionally defines checks before implementation chooses a packaging tool.

## Versioning

Use semantic versioning for the SpaceDress application once public releases begin.

The **Style Manifest version is separate** from the app version. A SpaceDress app release may support multiple manifest versions.

## Pre-release checklist

### Compatibility

- test every supported macOS release;
- test current Apple-silicon hardware;
- verify single full-screen;
- verify split/tiled full-screen;
- verify multi-display with separate Spaces enabled;
- verify Mission Control overlay lifecycle and scaling;
- document any private-SPI degradation.

### Privacy and permissions

- verify Accessibility prompt/permission recovery;
- verify Screen Recording is requested only if a feature actually needs it;
- review diagnostics for document titles/paths;
- review entitlements and code-signing identity.

### Style Manifest

- validate bundled schema and examples;
- run compatibility fixtures for every supported manifest version;
- document additions/deprecations;
- ensure app-bundled resources cannot escape their approved root.

### Distribution

- build from a clean checkout;
- sign with the intended Developer ID;
- notarize and staple where applicable;
- verify Gatekeeper behavior on a clean test user/machine;
- generate SHA-256 checksums;
- create GitHub Release notes;
- attach artifacts and checksums;
- verify the downloaded artifact, not only the local build.

## Release notes

Lead with user-visible behavior. Put private API refactoring in implementation notes unless it changes compatibility.

Always call out:

- newly supported/unsupported macOS versions;
- changed permissions;
- manifest compatibility changes;
- known Mission Control regressions;
- security fixes.

## Hotfixes

A platform hotfix may intentionally disable a broken capability on a new macOS release rather than continue showing incorrectly aligned or misidentified overlays.

Correct absence is preferable to confidently wrong decoration.
