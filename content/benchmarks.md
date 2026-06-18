+++
title = "Benchmarks"
description = "Measured performance comparing hx to cabal, with honest results — where hx wins and where it doesn't"
template = "page.html"
+++

# Performance Benchmarks

Numbers, not adjectives. Here's the methodology and the measurements behind hx's speed claims — and an honest account of where hx is *not* faster.

> **Measured with hx 0.7.5 on 2026-06-18.** hx's native build path is meaningfully faster than cabal on **cold builds** and **CLI startup**; it is **not** faster on no-op incremental rebuilds or `clean`. stack was not re-measured for this release, so it is omitted rather than estimated.

## Test Environment

| Property | Value |
|----------|-------|
| **hx version** | 0.7.5 |
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
| Incremental (no changes) | 78.6 ms | **21.6 ms** | cabal 3.6× faster |
| Clean | 31.9 ms | **17.6 ms** | cabal 1.8× faster |

## Where hx wins — and where it doesn't

**hx's real, repeatable advantages are cold builds (≈4.4×) and CLI startup (≈4.5×).** The native build path constructs the module graph and invokes GHC directly, skipping cabal's package-database queries and build-plan calculation; and hx is a native Rust binary with no GHC-runtime startup cost.

**No-op incremental rebuilds and `clean` are not faster today.** hx's native no-op spends ~74 ms hashing sources and checking its fingerprint cache, where cabal's no-op check is ~3 ms. We'd rather publish that honestly than dress it up — it's a gap we're tracking, not a feature.

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

## Not re-measured for 0.7.5

Project init, single-file-change incremental builds, preprocessor overhead, dependency-resolution/solver scaling, and memory usage were measured on older releases but have **not** been re-run for 0.7.5. Rather than present stale figures as current, they're omitted here. Contributions welcome — [open an issue](https://github.com/arcanist-sh/hx/issues).
