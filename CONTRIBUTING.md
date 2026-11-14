# Contributing to cartouche

Thanks for your interest in contributing. This document covers the basics.

## Getting started

Relevant docs for contributors:

- [`doc/setup.md`](doc/setup.md) — build, test, and coverage commands
- [`doc/testing.md`](doc/testing.md) — testing strategy, fuzzing, and CI expectations
- [`doc/architecture.md`](doc/architecture.md) — wire format, encode/decode abstraction, and design principles
- [`doc/roadmap.md`](doc/roadmap.md) — planned features and known gaps

## Issues and pull requests

**Open an issue first** if you're unsure whether something is a bug or if you want to
discuss a change before implementing it. For small, self-contained fixes a PR on its own
is fine.

- Bug reports: describe the InfoFrame bytes that produce the wrong result. A hex dump of
  the 31-byte packet is ideal.
- Feature requests: a brief description of what you need and why is enough to start a
  conversation.
- PRs: keep them focused. One logical change per PR makes review faster and keeps
  history readable.

## Coding standards

- Run `cargo fmt` and `cargo clippy --all-targets -- -D warnings` before pushing; CI runs
  clippy in every feature set ([doc/setup.md](doc/setup.md) lists the commands).
- Public items need rustdoc comments, and intra-doc links must resolve
  (`cargo rustdoc --all-features -- -D warnings` must pass).
- Follow the existing patterns in the codebase — see [`doc/architecture.md`](doc/architecture.md)
  for the design principles behind them.
- `#![forbid(unsafe_code)]` is enforced; no unsafe code.
- A claim about HDMI behaviour (a rule, a default, a threshold, a register encoding) cites
  at least two independent public implementations (repository, commit, file and line).
  Where implementations differ, the behaviour is a setting with the majority as default.
- Keep `no_std` compatibility. All encoding and decoding must compile without `alloc` or
  `std`. Warning storage is feature-gated; use `iter_warnings()` for portable access.
- All public structs and enums must be `#[non_exhaustive]`.

## Commit and PR expectations

- Write commit messages in the imperative mood ("Add support for …", not "Added …").
- Keep commits logically atomic. A PR that touches three unrelated things should be three
  commits (or three PRs).
- Tests are expected for new encoding/decoding logic. A round-trip unit test with a
  handcrafted struct covering all fields and warning conditions is the minimum; a
  hex-fixture test against a real captured packet is a bonus.
- CI must be green before a PR can merge: fmt, clippy, docs, the tests in every feature
  set, the example, the bare-metal `no_std` builds, the same tests and builds on the MSRV
  (Rust 1.85), and coverage must not drop more than 0.1 percentage points below the
  baseline (stored in `.coverage-baseline`). New encoding/decoding logic without tests will
  likely trip this.

## Publishing and downstream dependencies

cartouche builds on [display-types](https://github.com/ashlar-display/display-types) and
uses its types in its API (`ColorFormat` in the AVI InfoFrame), so moving to a breaking
release of display-types is a breaking release of cartouche, unless the version
requirement keeps accepting the old release as well. Other
[Ashlar crates](https://ashlar-display.org/crates.html) depend on cartouche as a versioned
crate dependency. Changes that add or modify public API must be merged, tagged, and
published to crates.io before downstream crates can adopt them. The workflow is:

1. Merge and tag a release in this repo (`vX.Y.Z`).
2. The publish CI workflow uploads the new version to crates.io.
3. Update the version constraint in the downstream crate's `Cargo.toml`.

## Review process

PRs are reviewed on a best-effort basis. Expect feedback within a few days; if you
haven't heard back in a week feel free to ping the thread. Reviews aim to be
constructive — if something needs to change, the reviewer will explain why. Approval
from the maintainer is required to merge.

## Code of Conduct

This project follows the [Contributor Covenant 3.0](https://github.com/ashlar-display/.github/blob/HEAD/CODE_OF_CONDUCT.md),
the code of conduct of every [Ashlar](https://ashlar-display.org) repository. Please read it
before participating.
