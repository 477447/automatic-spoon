# automatic-spoon
@@ -0,0 +1,8 @@
/target
**/*.rs.bk
Cargo.lock
bin/
pkg/
dist/
wasm-pack.log
node_modules
 69  .travis.yml 
@@ -0,0 +1,69 @@
language: rust
sudo: false

cache: cargo

matrix:
  include:

  # Builds with wasm-pack.
  - rust: beta
    env: RUST_BACKTRACE=1
    addons:
      firefox: latest
      chrome: stable
    before_script:
      - (test -x $HOME/.cargo/bin/cargo-install-update || cargo install cargo-update)
      - (test -x $HOME/.cargo/bin/cargo-generate || cargo install --vers "^0.2" cargo-generate)
      - cargo install-update -a
      - curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh -s -- -f
    script:
      - cargo generate --git . --name testing
      # Having a broken Cargo.toml (in that it has curlies in fields) anywhere
      # in any of our parent dirs is problematic.
      - mv Cargo.toml Cargo.toml.tmpl
      - cd testing
      - wasm-pack build
      - wasm-pack test --chrome --firefox --headless

  # Builds on nightly.
  - rust: nightly
    env: RUST_BACKTRACE=1
    before_script:
      - (test -x $HOME/.cargo/bin/cargo-install-update || cargo install cargo-update)
      - (test -x $HOME/.cargo/bin/cargo-generate || cargo install --vers "^0.2" cargo-generate)
      - cargo install-update -a
      - rustup target add wasm32-unknown-unknown
    script:
      - cargo generate --git . --name testing
      - mv Cargo.toml Cargo.toml.tmpl
      - cd testing
      - cargo check
      - cargo check --target wasm32-unknown-unknown
      - cargo check                                 --no-default-features
      - cargo check --target wasm32-unknown-unknown --no-default-features
      - cargo check                                 --no-default-features --features console_error_panic_hook
      - cargo check --target wasm32-unknown-unknown --no-default-features --features console_error_panic_hook
      - cargo check                                 --no-default-features --features "console_error_panic_hook wee_alloc"
      - cargo check --target wasm32-unknown-unknown --no-default-features --features "console_error_panic_hook wee_alloc"

  # Builds on beta.
  - rust: beta
    env: RUST_BACKTRACE=1
    before_script:
      - (test -x $HOME/.cargo/bin/cargo-install-update || cargo install cargo-update)
      - (test -x $HOME/.cargo/bin/cargo-generate || cargo install --vers "^0.2" cargo-generate)
      - cargo install-update -a
      - rustup target add wasm32-unknown-unknown
    script:
      - cargo generate --git . --name testing
      - mv Cargo.toml Cargo.toml.tmpl
      - cd testing
      - cargo check
      - cargo check --target wasm32-unknown-unknown
      - cargo check                                 --no-default-features
      - cargo check --target wasm32-unknown-unknown --no-default-features
      - cargo check                                 --no-default-features --features console_error_panic_hook
      - cargo check --target wasm32-unknown-unknown --no-default-features --features console_error_panic_hook
      # Note: no enabling the `wee_alloc` feature here because it requires
      # nightly for now.
 32  Cargo.toml 
@@ -0,0 +1,32 @@
[package]
name = "yew-wasm-pack-template"
version = "0.1.0"
authors = ["Justin Starry <justin.starry@icloud.com"]
edition = "2018"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
log = "0.4"
strum = "0.17"
strum_macros = "0.17"
serde = "1"
serde_derive = "1"
wasm-bindgen = "0.2.58"
web_logger = "0.2"
yew = { version = "0.13", features = ["web_sys"] }

# `wee_alloc` is a tiny allocator for wasm that is only ~1K in code size
# compared to the default allocator's ~10K. It is slower than the default
# allocator, however.
wee_alloc = { version = "0.4.4", optional = true }

[dev-dependencies]
wasm-bindgen-test = "0.3"

[dependencies.web-sys]
version = "0.3.4"
features = [
  'KeyboardEvent',
]
