+++
title = "Features"
description = "Explore hx features: fast builds, managed toolchains, deterministic lockfiles, excellent errors, and more"
template = "page.html"
+++

# Features

hx is a modern, fast, batteries-included toolchain for Haskell. Here's everything it can do.

---

## Batteries Included

One tool for your entire Haskell workflow. No more switching between cabal, stack, ghcup, fourmolu, and hlint.

### Build & Run

```bash
hx build              # Build the project
hx build --release    # Optimized release build
hx build --native     # Native build (bypasses cabal for simple projects)
hx run                # Build and run the executable
hx run -- --args      # Pass arguments to your program
```

### Test & Bench

```bash
hx test               # Run test suite
hx test --coverage    # Run with coverage reporting
hx bench              # Run benchmarks
hx bench --baseline   # Compare against saved baseline
```

### Format & Lint

```bash
hx fmt                # Format code with fourmolu
hx fmt --check        # Check formatting without modifying
hx lint               # Run hlint
hx lint --fix         # Auto-apply safe suggestions
```

### Documentation

```bash
hx doc                # Generate documentation
hx doc --open         # Generate and open in browser
hx doc --deps         # Include dependency documentation
```

### Watch Mode

```bash
hx watch              # Rebuild on file changes
hx watch test         # Re-run tests on changes
hx watch --check      # Type-check only (faster feedback)
```

### REPL

```bash
hx repl               # Start GHCi with project loaded
hx repl --test        # Include test modules
```

---

## Managed Toolchains

hx automatically manages GHC, Cabal, and HLS versions. No need for ghcup.

### Automatic Installation

When you run `hx build` in a project that specifies a GHC version, hx automatically downloads and installs the correct version:

```toml
# hx.toml
[toolchain]
ghc = "9.8.2"
```

```bash
$ hx build
→ Installing GHC 9.8.2...
→ Building project...
```

### Per-Project Versions

Different projects can use different GHC versions without conflicts:

```
project-a/
  hx.toml         # ghc = "9.6.4"

project-b/
  hx.toml         # ghc = "9.8.2"
```

### Toolchain Commands

```bash
hx toolchain list              # List installed toolchains
hx toolchain install 9.8.2     # Install specific GHC version
hx toolchain install --lts     # Install latest LTS GHC
hx toolchain remove 9.4.8      # Remove a version
hx toolchain default 9.8.2     # Set default version
```

### HLS Integration

hx ensures your HLS version matches your GHC version:

```bash
hx doctor
✓ GHC 9.8.2 installed
✓ HLS 2.6.0.0 compatible with GHC 9.8.2
✓ Cabal 3.12.1.0 installed
```

---

## Deterministic Builds

Every build is reproducible. hx uses TOML lockfiles with cryptographic verification.

### Lockfile Format

```toml
# hx.lock
version = 1
ghc = "9.8.2"

[[package]]
name = "aeson"
version = "2.2.1.0"
sha256 = "a5a5b8a..."
flags = []
deps = ["base", "text", "bytestring", ...]

[[package]]
name = "text"
version = "2.1"
sha256 = "b7c8d9e..."
flags = []
deps = ["base", "bytestring"]
```

### Lock Commands

```bash
hx lock               # Generate/update lockfile
hx lock --check       # Verify lockfile is up-to-date
hx lock --upgrade     # Upgrade all dependencies
hx lock --upgrade text aeson  # Upgrade specific packages
```

### Stackage Snapshots

Pin to a Stackage LTS or Nightly for curated package sets:

```toml
# hx.toml
[toolchain]
snapshot = "lts-22.7"
```

```bash
hx stackage list              # List available snapshots
hx stackage info lts-22.7     # Show snapshot details
hx stackage set lts-22.7      # Set snapshot for project
```

### CI Reproducibility

The same lockfile guarantees identical builds across machines:

```yaml
# .github/workflows/ci.yml
- name: Build
  run: |
    hx lock --check    # Fail if lockfile is stale
    hx build
    hx test
```

---

## Excellent Errors

hx transforms cryptic error messages into actionable guidance.

### Build Errors

Instead of raw GHC output, hx provides structured diagnostics:

```
error[E0001]: Type mismatch
  → src/Api.hs:42:15
  │
42│     return (show x)
  │            ^^^^^^^^
  │
  expected: Text
  found:    String

  help: Use `Data.Text.pack` to convert String to Text
        return (pack (show x))
```

### Missing Dependencies

```
error[E0012]: Package not found

  The package 'aeson' is used in src/Api.hs but not listed in dependencies.

  fix: Run `hx add aeson` to add it to your project
```

### Toolchain Issues

```
error[E0020]: GHC version mismatch

  expected: 9.8.2 (from hx.toml)
  found:    9.6.4 (installed)

  fix: Run `hx toolchain install 9.8.2`
       or change [toolchain] ghc in hx.toml
```

### hx doctor

Comprehensive environment diagnostics:

```bash
$ hx doctor

Environment Check
─────────────────
✓ hx 0.7.0
✓ GHC 9.8.2
✓ Cabal 3.12.1.0
✓ HLS 2.6.0.0 (compatible)

Project Check
─────────────
✓ hx.toml found
✓ hx.lock up-to-date
✓ .cabal file valid

Recommendations
───────────────
! Consider upgrading HLS to 2.7.0.0 for improved performance
  Run: hx toolchain install --hls latest
```

---

## Drop-in Compatible

hx works with your existing Haskell projects. No migration required.

### Cabal Compatibility

hx reads and writes standard `.cabal` files:

```bash
cd existing-cabal-project/
hx init --from-cabal     # Generate hx.toml from .cabal
hx build                 # Works immediately
```

### Stack Migration

Import Stack projects:

```bash
cd existing-stack-project/
hx init --from-stack     # Import from stack.yaml
hx build
```

### Hackage Integration

Full Hackage compatibility:

```bash
hx add aeson             # Add from Hackage
hx add aeson@2.2.1.0     # Specific version
hx add ./local-package   # Local path
hx add git@github.com:user/repo  # Git dependency
```

### Preserve Your Workflow

hx enhances without forcing changes:

| Your workflow | hx equivalent |
|---------------|---------------|
| `cabal build` | `hx build` |
| `cabal test` | `hx test` |
| `cabal run` | `hx run` |
| `stack build` | `hx build` |
| `ghcup install ghc` | `hx toolchain install` |
| `fourmolu -i .` | `hx fmt` |

---

## Native Build Mode

For simple projects, hx can bypass Cabal entirely for dramatically faster builds.

### How It Works

Native mode constructs the module graph and invokes GHC directly:

```bash
hx build --native
```

| Metric | Native | Cabal Backend |
|--------|--------|---------------|
| Cold build | 0.48s | 2.68s |
| Incremental | 0.05s | 0.39s |
| Overhead | Minimal | Package DB queries |

### When Native Mode Applies

| Project Type | Native? |
|--------------|---------|
| Single package, base only | Yes |
| Single package, few deps | Yes |
| Multi-package workspace | No |
| Custom Setup.hs | No |
| C FFI dependencies | No |

### Automatic Fallback

hx automatically falls back to cabal when native mode isn't applicable:

```bash
$ hx build --native
→ Project has external dependencies, using cabal backend
→ Building with cabal...
```

---

## Cross-Compilation

Build for different targets from a single machine.

### Supported Targets

```bash
hx build --target x86_64-linux-gnu
hx build --target aarch64-linux-gnu
hx build --target x86_64-windows-mingw
hx build --target wasm32-wasi
```

### Target Configuration

```toml
# hx.toml
[build]
default-target = "x86_64-linux-gnu"

[target.aarch64-linux-gnu]
ghc-options = ["-optl-static"]
```

### Docker Integration

```bash
hx build --target x86_64-linux-musl --static
# Produces fully static binary for Alpine/scratch containers
```

---

## Extensibility

Customize hx with hooks and plugins.

### Build Hooks

```toml
# hx.toml
[hooks]
pre-build = "scripts/generate-version.sh"
post-build = "scripts/copy-assets.sh"
pre-test = "scripts/setup-db.sh"
```

### Steel Plugins

Extend hx with Steel (Scheme) scripts:

```scheme
;; .hx/plugins/custom-lint.scm
(define (on-build-success project)
  (when (file-exists? "TODO.md")
    (warn "Don't forget to update TODO.md!")))

(register-hook 'post-build on-build-success)
```

### Nix Integration

Generate Nix expressions:

```bash
hx nix generate          # Generate flake.nix
hx nix shell             # Enter Nix shell with deps
```

### Distribution

Generate install scripts and package manifests:

```bash
hx dist homebrew         # Generate Homebrew formula
hx dist deb              # Generate .deb package spec
hx dist rpm              # Generate .rpm spec
hx dist installer        # Generate install.sh script
```

---

## Multiple Compiler Backends

hx supports both GHC and BHC (Basel Haskell Compiler).

### Configuration

```toml
# hx.toml
[compiler]
backend = "bhc"          # or "ghc" (default)

[compiler.bhc]
profile = "numeric"      # Optimized for numeric code
```

### Backend Selection

```bash
hx build                 # Use configured backend
hx build --backend ghc   # Override to GHC
hx build --backend bhc   # Override to BHC
```

### Why BHC?

BHC is a next-generation Haskell compiler focused on:
- Predictable performance (no lazy evaluation surprises)
- Modern concurrency primitives
- Tensor/array optimizations for ML workloads

---

## IDE Support

First-class editor integration.

### VSCode

```bash
hx setup vscode          # Configure VSCode settings
```

Creates `.vscode/settings.json` with correct HLS configuration.

### Neovim

```bash
hx setup nvim            # Generate lua config snippet
```

### HLS Configuration

```toml
# hx.toml
[hls]
formatter = "fourmolu"
plugins.hlint = true
plugins.retrie = false
```

### LSP Server Management

```bash
hx server start          # Start HLS in background
hx server status         # Check HLS status
hx server restart        # Restart HLS
hx server logs           # View HLS logs
```

---

## Summary

| Feature | hx | cabal | stack |
|---------|-----|-------|-------|
| Single binary | Yes | Yes | Yes |
| Managed toolchains | Yes | No (needs ghcup) | Yes |
| TOML lockfiles | Yes | No | No |
| Native fast builds | Yes | No | No |
| Built-in formatting | Yes | No | No |
| Built-in linting | Yes | No | No |
| Watch mode | Yes | No | Yes |
| Cross-compilation | Yes | Yes | Yes |
| Doctor diagnostics | Yes | No | No |
| Plugin system | Yes | No | No |
| BHC support | Yes | No | No |

---

## Getting Started

Ready to try hx? Install it in seconds:

```bash
curl -fsSL https://raw.githubusercontent.com/arcanist-sh/hx/main/install.sh | sh
```

Then create your first project:

```bash
hx init my-project
cd my-project
hx build
hx run
```

See the [Documentation](https://docs.arcanist.sh/hx/) for complete guides and reference.
