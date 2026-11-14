# Development Setup

**Requirements:** Rust 1.85+ (stable). Install via [rustup](https://rustup.rs/).

## Clone and build

```sh
git clone https://github.com/ashlar-display/cartouche.git
cd cartouche
cargo build
```

## Running checks

```sh
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo clippy --all-targets --no-default-features -- -D warnings
cargo clippy --all-targets --no-default-features --features alloc -- -D warnings
cargo clippy --all-targets --features serde -- -D warnings
cargo rustdoc --all-features -- -D warnings
```

## Running tests

```sh
cargo test                                         # std (default)
cargo test --features serde                        # std + serde
cargo test --no-default-features --features alloc  # alloc only
cargo test --no-default-features                   # bare no_std
cargo run --example roundtrip                      # encode and decode each InfoFrame type
```

The example needs `alloc`; without it Cargo skips it. The README's examples run as
doctests with the others. See [testing.md](testing.md) for what the tests cover.

## `no_std` builds

Without `std` the crate is `no_std`, with or without `alloc`. CI checks both on a bare
target, which has no `std` for a dependency to pull in:

```sh
rustup target add thumbv7em-none-eabi
cargo build --no-default-features --features alloc --target thumbv7em-none-eabi
cargo build --no-default-features --target thumbv7em-none-eabi
```

## Checking the MSRV

The MSRV is `rust-version` in `Cargo.toml`. CI checks it with
[`cargo-msrv`](https://github.com/foresterre/cargo-msrv), which runs the checks on that
toolchain:

```sh
cargo install cargo-msrv --version 0.19.6 --locked
cargo msrv verify -- cargo test --locked
cargo msrv verify -- cargo test --locked --features serde
cargo msrv verify -- cargo test --locked --no-default-features --features alloc
cargo msrv verify -- cargo test --locked --no-default-features
cargo msrv verify --target thumbv7em-none-eabi \
  -- cargo build --locked --no-default-features --features alloc --target thumbv7em-none-eabi
cargo msrv verify --target thumbv7em-none-eabi \
  -- cargo build --locked --no-default-features --target thumbv7em-none-eabi
```

## Coverage

Install [`cargo-llvm-cov`](https://github.com/taiki-e/cargo-llvm-cov) and the LLVM
tools component:

```sh
rustup component add llvm-tools-preview
cargo install cargo-llvm-cov --locked
```

Measure coverage over the default and `serde` feature sets, as CI does:

```sh
cargo llvm-cov clean --workspace
cargo llvm-cov --no-report --locked
cargo llvm-cov --no-report --locked --features serde
cargo llvm-cov report --summary-only
```

Rust versions count lines differently, so the same code can measure differently on two
versions (cartouche: 99.16% on Rust 1.99, 99.01% on Rust 1.100). Measure the baseline with
the Rust version CI's `stable` uses, `cargo +<version> llvm-cov …`, or CI can fail on a
baseline that a different local version produced.

To update `.coverage-baseline` after improving coverage:

```sh
cargo llvm-cov clean --workspace
cargo llvm-cov --no-report --locked
cargo llvm-cov --no-report --locked --features serde
cargo llvm-cov report --json --summary-only \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(round(d['data'][0]['totals']['lines']['percent'], 2))" \
  > .coverage-baseline
```

CI fails when line coverage drops more than 0.1 percentage points below
`.coverage-baseline`. Commit `.coverage-baseline` alongside the tests that raised it. CI
will open a PR
automatically when coverage improves on a push to `main` or `develop`, but you can also
update it manually as part of your feature branch.

## Fuzzing

Fuzzing requires nightly:

```sh
cargo +nightly fuzz run avi
cargo +nightly fuzz run audio
cargo +nightly fuzz run hdr_static
cargo +nightly fuzz run hdmi_forum_vsi
cargo +nightly fuzz run dynamic_hdr
```

See [`doc/testing.md`](testing.md) for long-campaign workflows and corpus management.
