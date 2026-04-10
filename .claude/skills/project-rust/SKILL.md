---
name: project-rust
description: Rust Project Management Guide. Covers Rust project setup, workspace configuration, dependency management, and build optimization. Use when creating or managing Rust projects.
version: 1.0.0
---

# Rust Project Management Skill

Skill for managing Rust projects, from setup to deployment.

## Purpose

Rust projects have unique requirements:
- Workspace management
- Dependency management with Cargo
- Build optimization
- Cross-compilation
- Testing and benchmarking

## When to Use

- Creating new Rust projects
- Setting up Rust workspaces
- Managing dependencies
- Optimizing build times
- Setting up CI/CD
- Cross-compilation

---

## Project Structure

### Single Crate

```
my-project/
├── Cargo.toml
├── src/
│   └── main.rs
├── tests/
│   └── integration_test.rs
└── benches/
    └── benchmark.rs
```

### Workspace

```
my-workspace/
├── Cargo.toml          # Workspace root
├── crates/
│   ├── core/
│   │   ├── Cargo.toml
│   │   └── src/
│   └── utils/
│       ├── Cargo.toml
│       └── src/
└── examples/
```

### Workspace Cargo.toml

```toml
[workspace]
members = ["crates/*", "examples/*"]
resolver = "2"

[workspace.package]
version = "0.1.0"
edition = "2021"
authors = ["Author Name <email@example.com>"]

[workspace.dependencies]
# Simple version
serde = "1.0"

# With features
tokio = { version = "1.35", features = ["rt", "sync", "macros"] }

# Specific version with defaults
serde = { version = "1.0.190", default-features = false }
```

### Inheriting Workspace Dependencies

In crate Cargo.toml:

```toml
[dependencies]
# Inherit from workspace (recommended)
serde = { workspace = true }

# Inherit with additional features
tokio = { workspace = true, features = ["rt-multi-thread"] }

# Override for specific crate
this-crate-only-dep = "1.0"
```

---

## Dependency Management

### Adding Dependencies

```bash
# Add dependency
cargo add serde

# Add dev dependency
cargo add --dev tokio

# Add build dependency
cargo add --build cc

# Add with specific version
cargo add serde@1.0
```

### Dependency Categories

```toml
[dependencies]
# Production dependencies
serde = "1.0"
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
# Test only
mockall = "0.12"
criterion = "0.5"

[build-dependencies]
# Build scripts
cc = "1.0"
```

### Feature Flags

```toml
[features]
default = ["client", "server"]
client = []
server = []
experimental = []
```

---

## Project Setup

### New Project

```bash
# Create binary
cargo new my-project

# Create library
cargo new --lib my-library

# Create workspace
mkdir my-workspace
cd my-workspace
cargo new crates/core
```

### Initialize with Template

```bash
# Use cargo-generate
cargo install cargo-generate
cargo generate --git https://github.com/rustwasm/wasm-pack-template

# Use axum template
cargo generate --git https://github.com/tokio-rs/axum-template

# Or use a specific template
cargo generate --git https://github.com/tauri-apps/tauri-app-template
```

---

## Build Optimization

### Release Profile

```toml
[profile.release]
opt-level = 3
lto = "fat"           # Link-time optimization
codegen-units = 1     # Better optimization
strip = true          # Strip symbols
panic = "abort"       # Smaller binary
```

### Tauri-Specific Profile

For Tauri desktop applications:

```toml
# For smaller binaries (recommended for desktop)
[profile.release]
opt-level = "s"       # Optimize for size
lto = true
codegen-units = 1
panic = "abort"       # Tauri recommended
strip = true
```

Or for faster execution:

```toml
# For faster execution
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
```

### Dev Profile

```toml
[profile.dev]
opt-level = 0
debug = true
```

### Incremental Compilation

```bash
# Set CARGO_INCREMENTAL
export CARGO_INCREMENTAL=1

# Or in .cargo/config.toml
[build]
incremental = true
```

---

## Workspace Management

### Crate Types

```toml
# Library
[lib]
name = "my_crate"
crate-type = ["lib", "cdylib", "staticlib"]

# Binary
[[bin]]
name = "my_binary"
path = "src/main.rs"
```

### Internal Dependencies

```toml
[dependencies]
my-core = { path = "../core" }

# With features
my-utils = { path = "../utils", features = ["json"] }
```

### Publishing

```toml
[package]
name = "my-crate"
version = "0.1.0"
edition = "2021"
license = "MIT"
repository = "https://github.com/user/repo"
description = "A short description"

[package.metadata.docs.rs]
all-features = true

[package.metadata.cargo-make]
# Make tasks
```

### Rust Edition Migration

When migrating between editions:

```bash
# Check for edition issues
cargo fix --edition

# Preview changes without applying
cargo fix --edition --dry-run

# Apply migration
cargo fix --edition
```

| Edition | Status | Notes |
|---------|--------|-------|
| 2015 | Stable | Legacy |
| 2018 | Stable | Most code compatible |
| 2021 | Stable | Default for new projects |
| 2024 | Upcoming | Coming late 2024/2025 |

For Tauri projects, use `edition = "2021"` (current standard).

---

## Testing

### Test Configuration

```toml
[lib]
test = true

# Integration tests in tests/ directory are automatically discovered
# Only use [[test]] for custom configuration:
[[test]]
name = "integration"
path = "tests/integration/main.rs"
required-features = ["integration"]
```

### Running Tests

```bash
# All tests
cargo test

# Specific test
cargo test test_name

# With output
cargo test -- --nocapture

# Doc tests
cargo test --doc

# Release tests
cargo test --release

# Run benchmarks
cargo bench
```

### Test Organization

```rust
// Unit tests in same file
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}

// Integration tests in tests/
#[cfg(test)]
mod integration {
    #[test]
    fn test_api() {
        // Test external behavior
    }
}
```

---

## CI/CD

### Modern GitHub Actions (Recommended)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

env:
  CARGO_TERM_COLOR: always
  SCCACHE_GHA_ENABLED: "true"
  RUSTC_WRAPPER: "sccache"

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Use sccache for faster builds
      - name: Run sccache-cache
        uses: mozilla-actions/sccache-action@v0.0.3

      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy

      # Cache dependencies
      - uses: Swatinem/rust-cache@v2

      - run: cargo fmt -- --check
      - run: cargo clippy -- -D warnings
      - run: cargo test

  msrv:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          toolchain: 1.75.0
      - run: cargo test
```

### Legacy GitHub Actions (Still Working)

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy
      - run: cargo test
      - run: cargo clippy -- -D warnings
      - run: cargo fmt -- --check
```

### Cargo Make

```toml
# cargo-make.toml
[tasks.build]
command = "cargo"
args = ["build", "--release"]

[tasks.test]
command = "cargo"
args = ["test", "--all"]

[tasks.lint]
command = "cargo"
args = ["clippy", "--", "-D", "warnings"]
```

---

## Cross-Compilation

### Target Triple

```bash
# Add target
rustup target add x86_64-unknown-linux-gnu

# Build for target
cargo build --target x86_64-unknown-linux-gnu

# Windows
rustup target add x86_64-pc-windows-gnu
cargo build --target x86_64-pc-windows-gnu

# ARM64 Linux
rustup target add aarch64-unknown-linux-gnu
cargo build --target aarch64-unknown-linux-gnu
```

### Using cross crate (Recommended)

```bash
# Install cross
cargo install cross

# Build for different targets
cross build --target x86_64-unknown-linux-gnu
cross build --target aarch64-unknown-linux-gnu
```

### .cargo/config.toml

```toml
[build]
target = "x86_64-unknown-linux-gnu"

# Default linker for all targets
[target.x86_64-unknown-linux-gnu]
linker = "clang"

# Windows using lld
[target.x86_64-pc-windows-msvc]
linker = "lld-link"

# ARM cross-compilation
[target.aarch64-unknown-linux-gnu]
linker = "aarch64-linux-gnu-gcc"
```

---

## Documentation

### Doc Comments

```rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// assert_eq!(add(2, 2), 4);
/// ```
///
/// # Panics
///
/// The function panics if...
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// A struct representing a person.
pub struct Person {
    /// The person's name
    name: String,
}
```

### Generate Docs

```bash
# Build docs
cargo doc

# Open docs
cargo doc --open

# With all features
cargo doc --all-features --no-deps
```

---

## Best Practices

### DO

- Use workspace for multi-crate projects
- Specify MSRV (Minimum Supported Rust Version)
- Use feature flags for optional functionality
- Run `cargo update` regularly
- Use `cargo clippy` for linting
- Format code with `cargo fmt`

### DO NOT

- Don't commit lock files for libraries (but do commit for binaries)
- Don't use `*` in version specifications
- Don't skip tests before publishing
- Don't forget to update version in Cargo.toml

### Cargo.lock Strategy

| Project Type | Commit Cargo.lock? | CI Strategy |
|--------------|-------------------|-------------|
| **Binary** | Yes (required) | Cache in CI |
| **Library** | No | Cache generated lock in CI |
| **Workspace** | Yes (if contains binaries) | Cache in CI |

```bash
# In CI for libraries - generate and cache
- name: Cache Cargo.lock
  uses: actions/cache@v3
  with:
    path: |
      ~/.cargo/bin/
      ~/.cargo/registry/index/
      ~/.cargo/registry/cache/
      ~/.cargo/git/db/
      target/
    key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
    restore-keys: |
      ${{ runner.os }}-cargo-
```

---

## Common Commands

```bash
# Build
cargo build              # Debug
cargo build --release    # Release

# Test
cargo test              # Run tests
cargo test --lib        # Library tests only
cargo test --doc        # Doc tests only

# Lint
cargo clippy            # Lint with warnings
cargo clippy -- -D warnings

# Format
cargo fmt              # Format code

# Dependencies
cargo tree             # Show dependency tree
cargo update           # Update dependencies
cargo outdated         # Check for updates

# Documentation
cargo doc              # Build docs
cargo doc --open       # Open docs

# Publishing
cargo publish          # Publish to crates.io
cargo package          # Create package
```

---

## Modern Tools (Recommended)

### cargo-nextest (Faster Test Runner)

```bash
# Install
cargo install cargo-nextest

# Run tests (2-10x faster than cargo test)
cargo nextest run

# With coverage
cargo nextest run --codec=lcov

# Configuration
mkdir -p .config/nextest.toml
```

```toml
# .config/nextest.toml
[profile.default]
retries = 2              # Retry flaky tests
slow-timeout = "60s"     # Timeout for slow tests

[profile.ci]
retries = 3
```

### cargo-deny (Dependency Audit)

```bash
# Install
cargo install cargo-deny

# Check licenses and vulnerabilities
cargo deny check
```

```toml
# deny.toml
[advisories]
db-path = "~/.cargo/advisory-db"
db-urls = ["https://github.com/rustsec/advisory-db"]
vulnerability = "deny"
unmaintained = "warn"
yanked = "deny"

[licenses]
allow = ["MIT", "Apache-2.0", "BSD-3-Clause"]
deny = ["GPL-2.0", "GPL-3.0"]

[bans]
multiple-versions = "warn"
wildcards = "allow"  # Allow in workspaces
```

### cargo-machete (Unused Dependencies)

```bash
# Install
cargo install cargo-machete

# Detect unused dependencies
cargo machete
```

### cargo-udeps (Nightly - Unused Dependencies)

```bash
# Install (requires nightly)
cargo +nightly install cargo-udeps

# Detect unused dependencies
cargo +nightly udeps
```

### Complete Validation Pipeline

```bash
# Modern Rust project validation
cargo check --all-features
cargo nextest run --all-features
cargo clippy -- -D warnings
cargo fmt -- --check
cargo deny check
cargo machete
cargo doc --no-deps --all-features
```

---

## Self-Check / Validation

### Validation Commands

```bash
# 1. Basic checks
cargo check

# 2. Run tests (use nextest for speed)
cargo nextest run

# 3. Code quality
cargo clippy -- -D warnings

# 4. Formatting
cargo fmt -- --check

# 5. Security & licenses
cargo deny check
cargo audit

# 6. Unused dependencies
cargo machete

# 7. Documentation
cargo doc --no-deps

# 8. MSRV check
cargo +1.75.0 check

# 9. Release build
cargo build --release
```

### Validation Checklist

- [ ] Project compiles without errors
- [ ] All tests pass (via cargo test or cargo nextest)
- [ ] Clippy passes (no warnings)
- [ ] Code is formatted
- [ ] Documentation builds
- [ ] Workspace structure is correct
- [ ] Dependencies are locked
- [ ] No security vulnerabilities (cargo deny check)
- [ ] No unused dependencies (cargo machete)
- [ ] MSRV verified
- [ ] Documentation builds
- [ ] Workspace structure is correct
- [ ] Dependencies are locked
