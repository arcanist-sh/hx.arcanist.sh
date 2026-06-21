+++
title = "Verified Against Reality"
description = "hx 0.7.11 and 0.7.12 came out of pointing nightly CI at real Haskell projects and a real BHC toolchain. Correct by construction is the design; this is what it takes to keep it true."
date = 2026-06-21
template = "page.html"

[taxonomies]
tags = ["release", "testing", "bhc", "lockfile"]

[extra]
author = "arcanist.sh"
+++

hx is built to be correct by construction — generate the command a tool actually expects, write the lockfile the solver actually resolved, never approximate. But "by construction" is a claim about design, and a design is only ever as correct as the assumptions underneath it. The way to find the wrong assumptions is to run the real thing against the real world.

So we pointed nightly CI at real projects. Not a hello-world that exercises the happy path, but scaffolded applications that pull real dependencies from Hackage, and — more recently — a real BHC toolchain downloaded straight from its releases. The job runs the commands a person actually runs in a day: `build`, `lock`, `sync`, `add`, `rm`, `tree`, `outdated`, `why`. Then it checks not just that they exit zero, but that their output is clean and their results are real.

Two releases came directly out of what that found. Both were bugs that passed every hello-world and failed the moment a real project was involved.

## A lockfile that resolved nothing

`hx lock` exited zero. It wrote an `hx.lock`. The build worked. Everything looked fine — and the lockfile was empty:

```toml
version = 1
packages = []
```

For every single-package project — which is to say, every project `hx init` or `hx new` produces — the native solver was collecting its dependencies by walking the *workspace members*, a list that is only populated when a `cabal.project` exists. A single-package project has none, so the solver resolved an empty set, wrote `packages = []`, and reported success. `build` never noticed, because it hands off to the underlying build directly. But `hx why`, `hx deps`, and `hx outdated` all read the lockfile — so they were quietly answering questions about nothing.

The exit-code check in CI didn't catch it; the *populated-lockfile* check did. Fixing the collection step then surfaced the next layer, and the next: the solver was trying to resolve GHC's bundled packages (`base`, `rts`, `ghc-prim`) from Hackage and failing on the ones that aren't published there; a cycle detector was misreading a diamond dependency as a cycle; the constraint parser was choking on real-world Cabal syntax it had never seen on a toy project — wildcards (`== 0.5.*`), set notation (`base ^>= {4.14, 4.17}`), parenthesised bounds. Each one was invisible until there were real dependencies to trip over.

**0.7.11** fixed the chain. A real project now locks the packages it actually uses, and `why` / `deps` / `outdated` answer from a lockfile that means something.

## A build tool that wasn't speaking BHC

[0.7.0](@/blog/hx-0-7-0-native-bhc-builds.md) was titled "Speaking BHC Natively." It generated BHC's own flags instead of GHC's — correct by construction, against the BHC command-line interface as we understood it. Then we installed the *shipped* compiler in CI and discovered the interface had moved.

hx was invoking:

```
bhc build --profile=numeric --tensor-fusion --emit-kernel-report
```

BHC 0.2.3 rejects nearly all of that. `--tensor-fusion` doesn't exist. `--emit-kernel-report` doesn't exist. The `build` subcommand is a stub; every real option is global and has to precede the source files. The command BHC actually wants is:

```
bhc --profile numeric -O 2 -I src src/Compute.hs app/Main.hs -o app
```

And even once the arguments are right, two things bite. BHC emits `-lbhc_rts` to the linker without telling the linker where its runtime libraries live, so every link failed with `library 'bhc_rts' not found` until hx learned to set `LIBRARY_PATH` to BHC's own `lib/` directory. And BHC frequently exits zero while printing compile and link errors — so a build tool that trusts the exit code reports success on a failed build. hx now reads the output, not just the status, and reports failure when BHC failed.

The install path had drifted too: `hx toolchain install --bhc` was pointed at a repository that no longer exists. It now fetches the real release, verifies it against the published checksums, preserves the flat layout BHC needs to find its runtime, and activates a version so the compiler works from `PATH`.

**0.7.12** makes the 0.7.0 promise true against the compiler people can actually download.

## A guard, not just a fix

Finding a bug once is luck. The point of the harness is that the fix stays fixed. Alongside the real-world job there's now a `bhc-pipeline` job that installs a real BHC toolchain on every run and builds and runs a BHC program through hx — asserting both that a good build succeeds *and* that a broken build is reported as a failure. The empty-lockfile case has its own assertion: it is not enough for `hx lock` to exit zero; the lockfile has to contain packages.

These checks exist because the corresponding bug shipped. That's the deal: every gap reality finds becomes a line in CI that reality can't reopen quietly.

## Honest about what's not ours to fix

Pointing hx at a real BHC also drew a clean line around what hx can and can't do. The `numeric` and `server` templates still don't build on BHC 0.2.3 — not because hx invokes the compiler wrong (it no longer does), but because the compiler can't yet compile them. BHC 0.2.3 rejects polymorphic numeric code (`sum` over a list of `Double`, `fromIntegral` into a fraction), and it ships no Servant. Those are compiler features, not build-tool bugs, and we filed them upstream rather than paper over them in a template. hx's job is to drive BHC correctly and tell you the truth about the result; for those templates, the truth is "not yet," and hx now says so instead of exiting zero on a build that didn't happen.

## Upgrading

```bash
# Self-update an existing install (verifies the release .sha256 first)
hx upgrade

# Or install fresh
curl -fsSL https://arcanist.sh/hx/install.sh | sh
```

```
$ hx --version
hx 0.7.12
```

## Resources

- [Compiler Backends](https://docs.arcanist.sh/hx/docs/features/compiler-backends/) — GHC vs BHC comparison
- [Lockfiles](https://docs.arcanist.sh/hx/docs/features/lockfiles/) — How `hx lock` and `hx sync` work
- [Installation](https://docs.arcanist.sh/hx/docs/installation/) — Install paths and verifying release checksums
- [Changelog](https://github.com/arcanist-sh/hx/blob/main/CHANGELOG.md) — Full 0.7.11 and 0.7.12 notes
