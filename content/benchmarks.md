+++
title = "Benchmarks"
description = "Measured performance comparing hx to cabal, with honest results — where hx wins and where it doesn't"
template = "page.html"
+++

# Performance Benchmarks

Numbers, not adjectives. Here's the methodology and the measurements behind hx's speed claims — and an honest account of where hx is *not* faster.

> **Measured with hx 0.7.7.** hx is faster than cabal on all four operations measured here — cold builds, CLI startup, no-op incremental rebuilds, and clean. stack was not re-measured for this release, so it is omitted rather than estimated.

## Test Environment

| Property | Value |
|----------|-------|
| **hx version** | 0.7.7 |
| **GHC version** | 9.8.2 |
| **Cabal version** | 3.12.1.0 |
| **stack** | not measured |
| **Platform** | macOS, Apple M4 (10-core) |
| **Tooling** | hyperfine 1.20.0 (8–20 runs, 2–3 warmup) |
| **Date** | 2026-06-18 |

## Results

**Test project:** a simple 3-module executable (`Main.hs`, `Lib.hs`, `Utils.hs`) depending only on `base` — the case hx's native build mode targets.

| Operation | hx (`--native`) | cabal | Result |
|-----------|-----------------|-------|--------|
| CLI startup (`--help`) | **4.0 ms** | 18.0 ms | hx **4.5× faster** |
| Cold build (clean state) | **0.45 s** | 2.02 s | hx **4.4× faster** |
| Incremental (no changes) | **3.3 ms** | 18.2 ms | hx **5.4× faster** |
| Clean | **3.7 ms** | 17.8 ms | hx **4.8× faster** |

## Where hx wins

**hx is faster than cabal on all four operations measured here** — cold builds (≈4.4×), CLI startup (≈4.5×), no-op incremental rebuilds (≈5.4×), and clean (≈4.8×). The native build path invokes GHC directly, skipping cabal's package-database queries and build-plan calculation, and hx is a native Rust binary with no GHC-runtime startup cost.

Two of those weren't always wins, and we'd rather say so than airbrush it: the no-op rebuild used to spend ~74 ms spawning `ghc`/`ghc-pkg` before realizing nothing had changed, and `clean` used to spin up the plugin runtime needlessly. Both paths were short-circuited. Earlier published cabal figures for incremental and clean were also overstated; these numbers are measured fresh.

## Native Build Mode

hx's native mode bypasses cabal entirely for simple projects: direct GHC invocation, no package-database queries or build-plan calculation, content-hash fingerprint caching, and native parallel compilation.

### When native builds apply

| Scenario | Native build? |
|----------|---------------|
| Single-package project | Yes |
| Only `base` dependencies | Yes |
| Multiple external dependencies | No (falls back to cabal) |
| Custom `Setup.hs` | No |
| C FFI / foreign libraries | No |

## Reproduce These Numbers

```bash
cargo install hyperfine

mkdir /tmp/hx-bench && cd /tmp/hx-bench
hx init bench --name bench
# (3-module base-only project — see docs/BENCHMARKS.md for the exact files)

# cold build (clean before each run)
hyperfine --warmup 1 --prepare 'rm -rf .hx dist-newstyle' 'hx build --native' 'cabal build'

# incremental, no changes (warm up first)
hx build --native && cabal build
hyperfine --warmup 3 'hx build --native' 'cabal build'
```

Full methodology and the exact test files are in [`docs/BENCHMARKS.md`](https://github.com/arcanist-sh/hx/blob/main/docs/BENCHMARKS.md).

## Not re-measured for 0.7.7

Project init, single-file-change incremental builds, preprocessor overhead, dependency-resolution/solver scaling, and memory usage were measured on older releases but have **not** been re-run for 0.7.7. Rather than present stale figures as current, they're omitted here. Contributions welcome — [open an issue](https://github.com/arcanist-sh/hx/issues).
