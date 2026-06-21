+++
title = "hx 0.8.0: The Projects You Already Have"
description = "0.8.0 is a milestone of correctness. Across the 0.7 series, hx learned to adopt, lock, and build real Haskell projects of every shape — single packages, multi-package workspaces, conditional dependencies, custom Setup.hs — and to be honest when it cannot."
date = 2026-06-21
template = "page.html"

[taxonomies]
tags = ["release", "milestone", "lockfile", "workspace"]

[extra]
author = "arcanist.sh"
+++

A toolchain earns trust on the projects people actually have, not the ones in its own test suite. **hx 0.8.0** is the release where that distinction stopped being aspirational. The 0.7 series was a single long sweep with one rule: point hx at a real Haskell project, and fix whatever reality returns. 0.8.0 is what is left once that loop runs out of new failures.

The work was not glamorous. It was correctness — the kind a build tool is supposed to have before anyone notices it at all.

## Adopt what already exists

A new project is the easy case. The real test is the project you did not start with hx.

`hx import --from cabal` now adopts an existing Cabal project from whatever it finds: a `cabal.project`, or just a bare `.cabal` file with no project file at all — the common shape for a single-package library. `hx import --from stack` does the same for Stack, workspace members and all. And when you run a command inside an existing Cabal or Stack project that hx has not adopted yet, the error does not dead-end. It names the project it found and tells you the command to adopt it.

```bash
cd some-library/
hx import --from cabal
hx build
```

This came out of adopting real packages off Hackage and from source — `optparse-applicative`, `pretty-simple`, the `prettyprinter` repository — and fixing each thing that broke. Every fix is now a line in CI that reality cannot reopen quietly.

## A lockfile that means something

For a long time hx wrote a lockfile that was technically a file and practically a lie. Single-package projects locked nothing. Platform-specific dependencies leaked across platforms. Disabled components contributed their dependencies anyway.

0.8.0 ends that. `hx lock` produces a lockfile that records what your project actually builds — and only that. The native solver reads each `.cabal` file's conditionals — `os(…)`, `arch(…)`, `impl(ghc …)`, `flag(…)` — and evaluates them against your target compiler and platform. It excludes the dependencies of components turned off with `buildable: False`. A Windows-only `Win32`, a legacy-GHC `semigroups`: they do not appear in a lockfile that is not for them.

Because the lockfile is real, it answers questions:

```bash
hx why aeson
hx deps tree
hx outdated
```

Adopting `optparse-applicative` on macOS dropped its lockfile from 36 packages to 22. The build still passed. The difference was 14 packages that were never going to be built — now absent, because the lockfile is correct by construction, not by coincidence.

## Every package in the project

A `cabal.project` with several local packages is a workspace, and a workspace has no single thing to build. `hx build` and `hx test` now cover every member through Cabal's `all` target, and `hx build --package <name>` narrows to one. Packages with a custom `Setup.hs` — `build-type: Custom` — build correctly through the same path.

Three project shapes that did not build untargeted before this series — the multi-package workspace, the disabled-component package, the custom-Setup package — build now.

## Honest about its limits

Correctness includes knowing what you cannot do. The BHC backend drives the shipped compiler through its real command-line interface, sets the library path its linker needs, and — because BHC can exit zero on a failed compile — reads the output rather than trusting the exit code. Where BHC cannot yet compile a program, hx says so and fails, instead of reporting a success that did not happen. The boundary is drawn where it actually is.

## Correctness did not cost speed

A common bargain is that getting a thing right makes it slower. We re-measured at 0.8.0 to check, against cabal, on the workflows you run all day:

| Operation | hx | cabal | |
|---|---|---|---|
| CLI startup | 3.2 ms | 18.6 ms | 5.8× |
| Cold build | 0.39 s | 2.04 s | 5.2× |
| Incremental (no-op) | 3.2 ms | 18.2 ms | 5.7× |
| Clean | 4.7 ms | 18.9 ms | 4.1× |

Every operation held or improved on 0.7. The correctness work lives in the lockfile path, not the build path, so the numbers above were never going to move much — but there was one place it could, and did. Evaluating every package's conditionals while parsing the full Hackage index made the first cold lock about a third slower. We measured it, found a lowercased copy of every line being allocated across a ninety-megabyte index, removed it, and got most of that back. The cold parse is a one-time cost, cached for a day; a warm lock is around thirty-seven milliseconds, unchanged.

The point is not the thirty-seven milliseconds. The point is that the regression was found by measuring, not by a user filing it. That is the same discipline as the rest of the release, pointed at performance.

## The state of hx

0.8.0 is not a feature release. It is the release where the foundation stopped shifting. hx adopts the project you have, locks exactly what it builds, builds every shape of project Haskell programmers actually write, and tells you the truth about the result.

The tooling is the thesis. This is the part of the thesis that had to be true first.

## Upgrading

```bash
# Self-update an existing install (verifies the release .sha256 first)
hx upgrade

# Or install fresh
curl -fsSL https://arcanist.sh/hx/install.sh | sh
```

```
$ hx --version
hx 0.8.0
```

## Resources

- [Adopting an existing project](https://docs.arcanist.sh/hx/docs/commands/import/) — `hx import`
- [Workspaces](https://docs.arcanist.sh/hx/docs/guides/workspaces/) — multi-package projects
- [Lockfiles](https://docs.arcanist.sh/hx/docs/commands/lock/) — what `hx lock` guarantees
- [Changelog](https://github.com/arcanist-sh/hx/blob/main/CHANGELOG.md) — full 0.8.0 notes
