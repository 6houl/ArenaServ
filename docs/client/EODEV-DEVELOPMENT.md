# EODev Development Guide

**Status:** AUTHORITATIVE  
**Last Updated:** 2026-09-23

## Scope

EODev is the custom client in `EODev/`. Client work is limited to `EODev/` and supporting notes in `docs/` unless the task is expanded explicitly.

The existing EODev artwork, fixed panel layout, and interaction style remain authoritative. EndlessClient is a behavior reference. Its floating panels, zoom, scaling, and screen layout are not targets for EODev.

## Source Priority

Use this order when packet or gameplay sources disagree:

1. EOProtocol packet definitions from `Moffat.EndlessOnline.SDK`
2. EOLib packet handlers, tests, and domain behavior in `EODev/EndlessClient-master/`
3. ArenaServ and Etheos handlers in the repository root
4. Protocol and client notes in `docs/`
5. Existing EODev behavior

Do not invent a packet layout. If the sources still disagree, record the uncertainty in `docs/research/UNKNOWN-BEHAVIOR.md` and verify it against the server before implementation.

## Client Reference Rules

- Keep EODev's existing UI layout and graphics.
- Use EndlessClient to confirm packet order, field types, state changes, validation, and error handling.
- Add only behavior supported by the current client assets, EOProtocol, EOLib, and ArenaServ.
- Prefer a complete, tested flow over several partial features.
- Keep comments short and focused on why the code needs a non-obvious rule.
- Do not commit `EndlessClient-master`, generated research files, build output, IDE state, downloaded dependencies, maps, pubs, or private configuration.

## Build Target

`EoDEV++.sln` and `EODev++ 2010.vcxproj` are the authoritative build files.

The similarly named obsolete DirectX-era project was removed. It was not referenced by the solution and contained machine-specific paths.

The supported verification target is:

```bat
build-release.cmd
```

Use `build-release.cmd -BuildOnly` for an incremental build.

The script verifies the local PTypes and SFML dependencies before invoking the `Release|Win32` solution build.

## Working Order

1. Keep one reproducible build target.
2. Add packet and connection verification.
3. Stabilize socket framing and validation.
4. Stabilize account creation and login.
5. Repair map and pub synchronization.
6. Remove known crash and corruption paths.
7. Implement gameplay systems and their existing EODev UI controls.

## Documentation

Record confirmed behavior with file and function references. Separate confirmed behavior from assumptions. Update this guide when a source-of-truth rule, build requirement, or client constraint changes.
