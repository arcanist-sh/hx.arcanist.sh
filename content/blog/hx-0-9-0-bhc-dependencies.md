+++
title = "hx 0.9.0: Build, Run, Test — on BHC, Dependencies and All"
description = "The BHC backend grows from compiling a single module to building a project and its dependencies from source into a package database — and running and testing the result. 0.9.0 makes build, run, and test work the same way on BHC, and fixes the plumbing underneath."
date = 2026-06-29
template = "page.html"

[taxonomies]
tags = ["release", "bhc", "native", "package-database"]

[extra]
author = "arcanist.sh"
+++

A compiler backend earns its keep one boundary at a time. The first boundary is a single module: parse it, compile it, run it. The next is a *dependency* — a module that imports another package, compiled separately, linked together. That second boundary is where a backend stops being a demo and starts being a toolchain. **hx 0.9.0** is the release where the BHC backend crosses it.

Until now, an `hx --backend bhc --native` build worked for a project that depended on nothing but `base`. 0.9.0 takes the same command and gives it a dependency graph: it builds each package from source, into a real package database, in dependency order — and then builds, runs, or tests your project against it.

## A package database, not a pile of objects

Separate compilation needs somewhere to put the *interfaces*. BHC emits a `.bhi` for each module — its public surface, enough to type-check and compile a dependent without that dependency's source on disk. 0.9.0 wires hx around those interfaces properly.

When hx builds a dependency, it compiles each module to a `.bhi` and an object, archives the objects, **installs the interfaces where the package's registration says they live**, and registers the package in a database keyed by a content-addressed id. The next package in the graph is compiled against that database; it resolves the one before it by name. By the time the local project is built, every dependency it names is a registered package the compiler can find.

This is the part that was quietly broken before and is now correct: a built package's interfaces used to be left in the build tree while its registration pointed somewhere else, so consumers could not actually resolve it. They can now.

## `--package-id`, and what a package is allowed to see

A package database is not a flat bag of modules. Each registered package *exposes* a set of modules and *depends on* a set of other packages. 0.9.0 honors both. hx passes BHC's `--package-id` for each dependency, and the compiler makes visible exactly that package's exposed modules plus the transitive closure of what it depends on — not the whole database. Import a module a package does not expose, and it does not resolve. Select one package, and its dependencies come along; select a dependency, and its dependents do not.

This is GHC's model, and it is the right one: visibility is a property of the dependency graph, not an accident of what happens to be on disk.

## The same verbs, on BHC

A build tool that can build but not run or test is half a tool. 0.9.0 closes that:

```bash
hx build --backend bhc --native
hx run   --backend bhc --native -- arg1 arg2
hx test  --backend bhc --native
```

`run --native` builds the native executable and then *runs* it, forwarding your arguments. This is not a convenience over the interpreter — it is the only way to run code that calls into a dependency, because the interpreter only has the dependency's interface, not its compiled body. `test --native` compiles a conventional test entry point — `test/Main.hs`, `test/Spec.hs` — to a native binary and runs it; a zero exit is a pass.

All three now go through the same native pipeline, and all three were exercised end to end against the shipped compiler, not just type-checked in Rust.

## Honest about the edge

BHC is a young compiler, and 0.9.0 draws the boundary where it actually is. The dependency build runs only as far as BHC can compile the packages in question; when a dependency cannot be built, the build does not produce a half-correct result — it falls back to a local-only build, and offline projects and projects without a lockfile keep working unchanged. The full fetch-from-Hackage path is in place and proven on controlled dependency chains; pointing it at the long tail of real Hackage packages is bounded by what the compiler can compile today, and we say so rather than imply otherwise.

Testing this also turned up — and let us fix, upstream in BHC — a class of bug a test runner cannot tolerate: a program that *fails* must exit non-zero. A BHC binary that called `exitFailure` used to abort; one that raised an `error` used to exit **zero**, so a failing assertion looked like a passing test. Those now do what they say: `exitSuccess`/`exitFailure`/`exitWith` carry the right status, and an uncaught exception prints its message and exits non-zero. A test that fails now fails.

## The shape of the thing

0.9.0 is not a flag here and a flag there. It is the BHC backend acquiring the three verbs a toolchain is judged by — build, run, test — over a real dependency graph, with a package database that means something underneath. The boundary it crosses is the one that matters: from a compiler that can handle a module to a toolchain that can handle a project.

```bash
hx build --backend bhc --native
```

Same command as before. It just reaches further now.
