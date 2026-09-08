# Security Policy

SpaceDress interacts with privileged-feeling parts of the desktop: Accessibility, application metadata, Mission Control, and potentially private read-only macOS interfaces. Security and privacy reports are welcome even before the first release.

## Reporting a vulnerability

**Do not open a public issue containing exploit details, private user data, or a working proof of concept for a serious vulnerability.**

Preferred reporting path:

1. use GitHub Private Vulnerability Reporting for this repository when available;
2. if that is unavailable, open a minimal public issue asking the maintainer for a private reporting channel without including sensitive details.

Include enough information to reproduce the problem:

- affected SpaceDress revision/version;
- macOS version and build;
- required permissions;
- attack prerequisites;
- impact;
- minimal reproduction steps;
- proposed mitigation if known.

## Current support status

There is no stable SpaceDress release yet. Reports against `main` and current development branches are still useful.

## Security boundaries

The project intends to preserve these boundaries:

- **No SIP weakening** as an installation requirement.
- **No code injection into the Dock or WindowServer** in the normal product architecture.
- **No executable code in DSAM documents.**
- **No remote DSAM assets** in DSAS 0.1.
- **No network service required** for normal operation.
- Private SkyLight/CGS use, when present, is isolated and preferably read-only.
- Application/document metadata stays local unless a future feature explicitly and transparently changes that policy.
- SpaceDress's internal multi-app user manifest is local product configuration and is not an executable plugin format.

## Sensitive metadata

Window titles, document titles, document paths, bundle paths, and screenshots can contain private information. Diagnostics should collect the minimum necessary data, label sensitive fields, and provide redaction or omission paths before sharing.

## Out of scope

Issues that require an attacker to already control the user's macOS account, replace the SpaceDress binary, or modify the user's own local configuration are generally outside the application security boundary unless SpaceDress magnifies that control into a distinct vulnerability.
