<<<<<<< HEAD
<<<<<<< HEAD
# Preparation

Before we start development of an ERC20 contract, we'll first prepare the repository. The code will be organized into 3 separate crates: `logic`, `contract` and `tests`.

We'll then start with generating a new project and applying a few changes as follows:

1. Generate a new project using [cargo-casperLabs](setup-of-rust-contract-sdk.html#cargo-casperlabs).

=======
=======
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720
# Prepare

Before we start the development, let's prepare the repository first.

The code will be divided into 3 crates: `logic`, `contract` and `tests`. Let's start with generating a new project and applying a few changes.

Generate a new project using [Cargo CasperLabs](../setup-of-rust-contract-sdk#cargo-casperlabs).
<<<<<<< HEAD
>>>>>>> 22e6a7df7d83eae009f0f66a080f8b69a1b8e108
=======
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720
```bash
$ cargo casperlabs erc20-tutorial
$ cd erc20-tutorial
$ tree
.
├── contract
│   ├── Cargo.toml
│   ├── rust-toolchain
│   └── src
│       └── lib.rs
└── tests
    ├── build.rs
    ├── Cargo.toml
    ├── rust-toolchain
    ├── src
    │   └── integration_tests.rs
    └── wasm
        ├── mint_install.wasm
        ├── pos_install.wasm
        └── standard_payment.wasm

5 directories, 10 files
```

<<<<<<< HEAD
<<<<<<< HEAD
Note: We highly recommend working within one [Cargo Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html), rather than separated crates. This is until and unless we are creating multiple implementations of an ERC20, in which case, each implementation should have it's own workspace.

1. Create a new file `Cargo.toml` in the root directory:

=======
It's better to have one [Cargo Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html) rather than separated crates. Create a new file `Cargo.toml` in the root directory.
>>>>>>> 22e6a7df7d83eae009f0f66a080f8b69a1b8e108
=======
It's better to have one [Cargo Workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html) rather than separated crates. Create a new file `Cargo.toml` in the root directory.
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720
```toml
# Cargo.toml

[workspace]

members = [
    "contract",
    "tests"
]
```

<<<<<<< HEAD
<<<<<<< HEAD
The project will have only one `target` directory placed in the root, so `tests/build.rs` has to be adjusted as follows:

1. Change 

```rust
const ORIGINAL_WASM_DIR: &str = "../contract/target/wasm32-unknown-unknown/release";
```
to
```rust
const ORIGINAL_WASM_DIR: &str = "../target/wasm32-unknown-unknown/release";
```

When building the WASM file we only want to build the `contract`, not all the crates:

1. Change

```rust
const BUILD_ARGS: [&str; 2] = ["build", "--release"];
```
to
```rust
const BUILD_ARGS: [&str; 4] = ["build", "--release", "-p", "contract"];
```

1. Finally, remove `rust-toolchain` files from `contract` and `tests` crates, and make just one crate in the root directory:

=======
=======
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720

Now the project will have only one `target` directory placed in the root directory, so `tests/build.rs` must be adjusted. Change
```
const ORIGINAL_WASM_DIR: &str = "../contract/target/wasm32-unknown-unknown/release";
```
to
```
const ORIGINAL_WASM_DIR: &str = "../target/wasm32-unknown-unknown/release";
```

When building WASM file we don't want to build all crates, but only the `contract` one. Change
```
const BUILD_ARGS: [&str; 2] = ["build", "--release"];
```
to
```
const BUILD_ARGS: [&str; 4] = ["build", "--release", "-p", "contract"];
```

Finally remove `rust-toolchain` files from `contract` and `tests` crates and make just one in the root directory.
<<<<<<< HEAD
>>>>>>> 22e6a7df7d83eae009f0f66a080f8b69a1b8e108
=======
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720
```bash
$ rm contract/rust-toolchain
$ mv tests/rust-toolchain .
```

<<<<<<< HEAD
<<<<<<< HEAD
1. Test the changes by compiling `contract` crate, and then running the test on `tests` crate.
=======
Test the changes by compiling `contract` crate and executing `cargo test` on `tests` crate.
>>>>>>> 22e6a7df7d83eae009f0f66a080f8b69a1b8e108
=======
Test the changes by compiling `contract` crate and executing `cargo test` on `tests` crate.
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720

```bash
$ cargo build --release -p contract 
$ cargo test -p tests
```
<<<<<<< HEAD
<<<<<<< HEAD

TODO: fix required `--target wasm32-unknown-unknown`
=======
>>>>>>> 22e6a7df7d83eae009f0f66a080f8b69a1b8e108
=======
>>>>>>> b3992d12a37fede262d693ed174bc2fd31c2f720
