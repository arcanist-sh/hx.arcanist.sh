+++
title = "hx and the 2026 Haskell Toolchain"
description = "GHC 9.14, Cabal 3.18, HLS 2.14, Stackage LTS 24 and BHC 0.2.22: the next hx release installs and understands all of them, with or without ghcup. A tour of what is new and how to use it."
date = 2026-10-19
template = "page.html"

[taxonomies]
tags = ["release", "toolchain", "ghc", "stackage"]

[extra]
author = "arcanist.sh"
+++

The Haskell toolchain moved a long way this year. GHC 9.14 arrived, Cabal reached 3.18, HLS 2.14 supports every maintained compiler, and Stackage LTS 24 settled on GHC 9.10. The next hx release brings all of that within one command: install it, pin it, and get told when two pieces do not fit together.

This post walks through what is new and how to use it.

## One command, no ghcup required

hx installs GHC and Cabal directly from downloads.haskell.org, verifies each download against the published checksums, and keeps every version side by side under `~/.hx/toolchains`. ghcup is supported but optional.

```bash
hx toolchain install 9.10.3 --cabal 3.18.2.0 --set
```

That works the same on Linux and macOS, on x86_64 and ARM. On Linux, hx picks the GHC build made for maximum compatibility, which runs on any current glibc distribution. If you prefer ghcup, add `--ghcup` and hx hands the install over to it.

To see what you have and switch between versions:

```bash
hx toolchain status
hx toolchain use 9.14.1
```

## A recommended toolchain for 2026

When a project does not pin anything, hx now recommends **GHC 9.10.3, Cabal 3.16.1.0 and HLS 2.14.0.0**. These are the versions ghcup recommends, and GHC 9.10.3 is the compiler of Stackage LTS 24, so the default lines up with the largest curated package set.

The newer releases are a pin away. hx knows GHC 9.12 and 9.14 and Cabal 3.14 through 3.18:

```toml
# hx.toml
[toolchain]
ghc = "9.14.1"
cabal = "3.18.2.0"
```

Run `hx build` and hx offers to install whatever the pin needs.

## New projects on any current GHC

Projects created with `hx init` or `hx new` are ready for every GHC from 9.6 to 9.14:

```bash
hx new cli my-tool
cd my-tool
hx run -- --help
```

The templates (`cli`, `library`, `webapp`, `server` and `numeric`) accept `base` up to 4.22, the version that ships with GHC 9.14.

## An editor setup that is checked, not guessed

Each HLS release ships one binary per supported GHC release, so the right HLS depends on the exact compiler, not just its major version. hx now carries the upstream support table. `hx doctor` and `hx ide status` tell you when your HLS and GHC fit together, and recommend the right HLS when they do not. That includes the last HLS release for an older GHC you are not ready to leave:

```bash
hx doctor
hx ide setup    # writes hie.yaml for HLS
```

For GHC 9.6.7, 9.8.4, 9.10.3, 9.12 and 9.14, the answer is HLS 2.14.0.0.

## Stackage snapshots, in full

hx reads Stackage snapshots directly, including the compiler each one is built with:

```
$ hx stackage info lts-24
Snapshot: lts-24
GHC Version: 9.10.3
Packages: 3406

To use this snapshot: hx stackage set lts-24
```

`hx stackage set lts-24` pins the snapshot for your project, so its 3,406 curated package versions are the ones you build with. Moving over from Stack is a single command, and hx reads the GHC version from your resolver, LTS 24 included:

```bash
hx import --from stack
```

## Nix, BHC and the rest of the family

- **Nix.** `hx nix flake` generates a `flake.nix` for your project, using the compiler you pinned or the recommended GHC 9.10.3 from nixpkgs.
- **BHC.** Every BHC release through 0.2.22 installs with one command, and 0.2.22 is the new default:

  ```bash
  hx toolchain install --bhc latest
  ```

## Current, top to bottom

hx itself builds with Rust 1.99 on up-to-date dependencies, and `cargo audit` comes back clean. The solver keeps a compact binary cache of the package index: all of Hackage, more than 19,000 packages, loads in around a tenth of a second.

The nightly CI does what you do. One job installs GHC and Cabal with hx on a fresh Linux machine, with ghcup out of the picture, and checks that both run. Another builds real projects with real Hackage dependencies, from `hx new` to `hx run`.

## Getting it

These changes land in the next hx release. To install hx, or to update an existing install when it ships:

```bash
curl -fsSL https://arcanist.sh/hx/install.sh | sh
hx upgrade
```

Then `hx toolchain install` gives you the 2026 toolchain in one step.
