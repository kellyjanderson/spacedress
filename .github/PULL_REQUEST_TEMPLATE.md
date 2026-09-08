## What is being changed?

<!-- Describe the problem, decision, or behavior this PR addresses. -->

## Why does it belong in SpaceDress or DSAS?

<!-- SpaceDress product changes should tie to product goals. DSAS changes should state the application-facing interoperability need. -->

## Evidence / verification

<!-- For macOS-sensitive work, include OS version/build, configuration, and reproducible observations. -->

- [ ] Automated tests added/updated where practical
- [ ] Manual verification described where automation is insufficient
- [ ] Single-app full-screen considered
- [ ] Split/tiled full-screen considered
- [ ] Split geometry/ratio considered when relevant
- [ ] Multi-display behavior considered when relevant

## Architecture and compatibility

- [ ] No persistent identity depends on a native Space ID/index/UUID
- [ ] Split side is not inferred from owner/PID/window array order
- [ ] Private macOS interfaces remain isolated behind adapters
- [ ] No feature requires SIP to be weakened
- [ ] DSAS compatibility is preserved or explicitly documented
- [ ] SpaceDress internal user configuration has not been accidentally promoted into DSAS
- [ ] ADR added/updated if a core invariant changes

## Privacy / security

<!-- Note new permissions, metadata collection, logging, screenshots, network access, or DSAM capabilities. -->

- [ ] No sensitive document titles/paths are added to routine logs
- [ ] No executable or remote behavior is introduced through application appearance data

## Documentation

- [ ] User-visible behavior is documented
- [ ] Research claims are labeled and sourced
- [ ] Examples/schema updated for DSAS changes

## Visual changes

<!-- Add screenshots/video when useful. Remove this section if not applicable. -->
