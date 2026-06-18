+++
title = "Speaking BHC Natively"
description = "hx 0.7.0 drives BHC through its own command-line interface instead of approximating it with GHC flags — native build flags, a filesystem package database, and a pinned toolchain."
date = 2026-06-17
template = "page.html"

[taxonomies]
tags = ["bhc", "release", "build"]

[extra]
author = "arcanist.sh"
+++

**hx 0.7.0** is about correctness underneath the build. The 0.6.0 release made starting a BHC project a single command; this one makes the build pipeline speak BHC's actual command-line interface rather than approximating it with GHC-style flags. If you build with BHC, your invocations are now the ones BHC actually expects — correct by construction, not by coincidence.

## Native flags, not GHC in disguise

Early BHC support reused the flag vocabulary hx already knew — GHC's. It mostly worked, because BHC's surface is GHC-shaped, but "mostly" is the wrong target for a build tool. 0.7.0 generates BHC's own flags directly:

| GHC-style (before) | BHC-native (0.7.0) |
|---|---|
| `-hidir=<dir>` / `-odir=<dir>` | `--hidir <dir>` / `--odir <dir>` |
| `-i<dir>` | `--import-path <dir>` |
| `-package-db=<path>` | `--package-db <path>` |
| `-O2` | `-O 2` |
| `-Wall` / `-Werror` | `--Wall` / `--Werror` |
| `.hi` interface files | `.bhi` interface files |

The `--make` flag is gone from the BHC build path entirely — hx orchestrates per-package builds itself rather than handing BHC a GHC-ism it has to tolerate. The result is fewer surprises when BHC's interface and GHC's drift apart, which they will.

## A package database without `bhc-pkg`

hx used to register and read packages by shelling out to `bhc-pkg`. 0.7.0 talks to the package database directly: it scans `.conf` files on disk to read the database, and copies `.conf` files into the database directory on register. No subprocess, no parsing another tool's stdout, no dependency on `bhc-pkg` being on `PATH` in the exact shape hx expected.

A nice side effect: hx no longer needs the `which` crate to locate that helper, so there's one less dependency in the build graph.

## Skipping what BHC already provides

BHC ships a standard library — `base`, `text`, `containers`, and friends — as builtins. Compiling those from source under BHC is wasted work at best and a conflict at worst. hx 0.7.0 carries a mapping of BHC's builtin packages and skips compiling them, deferring to the ones BHC already provides. Builds touch only the packages that are actually yours to build.

## A pinned, reproducible toolchain

hx itself is now pinned to Rust 1.96.0 via `rust-toolchain.toml` and `mise.toml`, with `rust-version = "1.96"` set as the workspace MSRV and inherited by every crate. Building hx from source resolves to one toolchain, everywhere — contributors, CI, and release builds all compile against the same compiler. Dependencies were refreshed within semver in the same pass (including `clap_mangen` 0.2 → 0.3), and the release archives — Linux x86_64/aarch64 (gnu + musl), macOS Intel/Apple Silicon, and Windows — each ship with a matching `.sha256` plus a combined `checksums.txt`.

## Upgrading

```bash
# Self-update an existing install (verifies the release .sha256 first)
hx upgrade

# Or install fresh
curl -fsSL https://arcanist.sh/hx/install.sh | sh
```

```
$ hx --version
hx 0.7.0
```

## Resources

- [Compiler Backends](https://docs.arcanist.sh/hx/docs/features/compiler-backends/) — GHC vs BHC comparison
- [BHC Platform Snapshots](https://docs.arcanist.sh/hx/docs/features/bhc-platform/) — Curated package set guide
- [Installation](https://docs.arcanist.sh/hx/docs/installation/) — Install paths and verifying release checksums
- [Changelog](https://github.com/arcanist-sh/hx/blob/main/CHANGELOG.md) — Full 0.7.0 notes
