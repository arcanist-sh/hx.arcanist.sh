+++
title = "hx at Zurihac 2026"
description = "hx will be at Zurihac 2026, 6–8 June in Rapperswil. What we'll demo, what install path to bring, and the questions most worth raising at the booth."
date = 2026-05-26
template = "page.html"

[taxonomies]
tags = ["events", "release"]

[extra]
author = "arcanist.sh"
+++

[Zurihac 2026](https://zfoh.ch/zurihac2026/) — 6–8 June, OST Eastern
Switzerland University of Applied Sciences, Rapperswil-Jona — is in
about eleven days, and **hx** will be there alongside
[BHC](https://arcanist.sh/bhc/blog/bhc-at-zurihac-2026/). This is the
short pre-event note: what we'll have running, how to install it
fresh on the venue wifi, and where your feedback will be most useful.

## What hx is, briefly

`hx` is a single command-line tool that replaces the cluster of
toolchain pieces a Haskell project normally needs to bolt together.
One binary covers what `cabal`, `stack`, `ghcup`, `fourmolu`, `hlint`,
and `hpc` cover today, plus a watch mode and integrated test
coverage. Written in Rust. No GHC plugin contract — it drives GHC
(and BHC) as subprocesses.

The 0.6.0 release added [zero-config BHC integration](@/blog/hx-0-6-0-zero-config-bhc.md):
`hx new numeric my-app && cd my-app && hx run` produces a working
BHC-compiled binary from a single `hx toolchain install` step.

## At the booth

Three things we'll demo on demand:

1. **From zero to a running project in under a minute.**
   `hx toolchain install --bhc latest && hx new numeric demo && hx run`.
   This is the boring-is-the-point demo. No `cabal update`, no
   `ghcup install`, no per-platform shell config — one command per
   line, three commands, one binary.
2. **Watch mode on a real project.** `hx watch test` re-runs the
   suite the moment a `.hs` file changes. The feedback loop is
   measured in single-digit milliseconds for the diff cycle, plus
   whatever GHC takes.
3. **Backend switching.** Same project, two compilers. `hx run` to
   compile through GHC; `hx run --backend=bhc` to compile through
   the [Basel Haskell Compiler](https://arcanist.sh/bhc/). When BHC
   can't compile something, the diagnostic comes back through the
   `hx` shell with the same paths and colours.

A printed install card will be at the table — the install lines below
plus QR codes back to GitHub and the [features page](@/features.md).

## What feedback we'd most like

A few questions are particularly useful coming from people who
actually maintain Haskell projects:

- **"Does it build *my* project?"** Bring a `.cabal` file, ideally a
  small one. `hx` consumes `.cabal` directly — no manifest rewrite.
  Build failures against real projects are the highest-signal bug
  reports we get.
- **"How does `hx watch` compare to `ghcid`?"** ghcid is excellent;
  we don't try to displace it. Direct comparison is welcome.
- **"What's the story for libraries vs binaries?"** Today's strongest
  story is binaries with a single executable; library publishing
  through Hackage is wired but less battle-tested.
- **"Why Rust?"** Asked frequently. Short answer: cross-platform
  static binaries, fast startup, and no chicken-and-egg with the
  Haskell installation we're trying to manage.

Less useful at the booth: questions about replacing GHC itself — that
is what [BHC](https://arcanist.sh/bhc/) is for, not hx.

## How to try it

```bash
# macOS / Linux
curl -fsSL https://arcanist.sh/hx/install.sh | sh

# Or via Homebrew
brew install arcanist-sh/tap/hx

# Then:
hx init my-app
cd my-app
hx run
```

Add `--bhc latest` to `hx toolchain install` to pull the
[v0.2.2 BHC release](https://github.com/arcanist-sh/bhc/releases/tag/v0.2.2)
that just shipped — it now bundles the full standard library, so
hello-world links on the first try.

## If something breaks

Open an issue with the `hx` version (`hx --version`), the OS, and
the exact command. If `hx --backend=bhc` is involved, a minimal `.hs`
that reproduces the failure is the highest-signal artifact.

See you in Rapperswil.
