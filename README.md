[![pipeline](https://github.com/kixelated/test-trace/actions/workflows/test.yml/badge.svg?branch=main)](https://github.com/kixelated/test-trace/actions/workflows/test.yml)
[![crates.io](https://img.shields.io/crates/v/test-trace.svg)](https://crates.io/crates/test-trace)
[![Docs](https://docs.rs/test-trace/badge.svg)][docs-rs]
[![rustc](https://img.shields.io/badge/rustc-1.71+-blue.svg)](https://blog.rust-lang.org/2023/07/13/Rust-1.71.0.html)

# test-trace

- [Documentation][docs-rs]
- [Changelog](CHANGELOG.md)

**test-trace** is a fork of [test-log](https://github.com/d-e-s-o/test-log) that takes care of automatically initializing tracing for Rust tests.
Unlike `test-trace`, all output is at the `TRACE` level by default.

## Usage

The crate provides a custom `#[test]` attribute that, when used for
running a particular test, takes care of initializing `log` and/or
`tracing` beforehand.

#### Example

As such, usage is as simple as importing and using said attribute:

```rust
use test_trace::test;

#[test]
fn it_works() {
  info!("Checking whether it still works...");
  assert_eq!(2 + 2, 4);
  info!("Looks good!");
}
```

It is of course also possible to initialize logging for a chosen set of
tests, by only annotating these with the custom attribute:

```rust
#[test_trace::test]
fn it_still_works() {
  // ...
}
```

You can also wrap another attribute. For example, suppose you use
[`#[tokio::test]`][tokio-test] to run async tests:

```rust
use test_trace::test;

#[test(tokio::test)]
async fn it_still_works() {
  // ...
}
```

#### Features

The crate comes with two features pertaining "backend" initialization:

- `log`, enabled by default, controls initialization for the `log`
  crate.
- `trace`, enabled by default, controls initialization for the `tracing` crate.

Depending on what backend the crate-under-test (and its dependencies)
use, the respective feature(s) should be enabled to make messages that
are emitted by the test manifest on the terminal.

On top of that, the `color` feature (enabled by default) controls
whether to color output by default.

#### Logging Configuration

As usual when running `cargo test`, the output is captured by the
framework by default and only shown on test failure. The `--nocapture`
argument can be supplied in order to overwrite this setting. E.g.,

```bash
$ cargo test -- --nocapture
```

Furthermore, the `RUST_LOG` environment variable is honored and can be
used to influence the log level to work with (among other things).
Please refer to the [`env_logger` docs][env-docs-rs] and
[`tracing-subscriber`][tracing-env-docs-rs] documentation for supported
syntax and more information.

If the `trace` feature is enabled, the `RUST_LOG_SPAN_EVENTS`
environment variable can be used to configure the tracing subscriber to
log synthesized events at points in the span lifecycle. Set the variable
to a comma-separated list of events you want to see. For example,
`RUST_LOG_SPAN_EVENTS=full` or `RUST_LOG_SPAN_EVENTS=new,close`.

Valid events are `new`, `enter`, `exit`, `close`, `active`, and `full`.
See the [`tracing_subscriber` docs][tracing-events-docs-rs] for details
on what the events mean.

#### MSRV Policy

This crate adheres to Cargo's [semantic versioning rules][cargo-semver].
At a minimum, it builds with the most recent Rust stable release minus
five minor versions ("N - 5"). E.g., assuming the most recent Rust
stable is `1.68`, the crate is guaranteed to build with `1.63` and
higher.

[cargo-semver]: https://doc.rust-lang.org/cargo/reference/resolver.html#semver-compatibility
[docs-rs]: https://docs.rs/test-trace
[env-docs-rs]: https://docs.rs/env_logger/0.11.2/env_logger
[log]: https://crates.io/crates/log
[tokio-test]: https://docs.rs/tokio/1.4.0/tokio/attr.test.html
[tracing]: https://crates.io/crates/tracing
[tracing-env-docs-rs]: https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/filter/struct.EnvFilter.html#directives
[tracing-events-docs-rs]: https://docs.rs/tracing-subscriber/0.3.18/tracing_subscriber/fmt/struct.SubscriberBuilder.html#method.with_span_events
