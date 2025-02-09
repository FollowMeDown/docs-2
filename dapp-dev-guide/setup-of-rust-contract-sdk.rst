Getting Started
===============

This guide covers the basic tools you will need to set up your first Casper smart contract. You will also be able to build a sample smart contract and run a few basic tests on it on your local machine.

Casper’s blockchain is built upon the Rust programming language and compiles down to WebAssembly. The Rust contract SDK is the easiest way to get started with smart contract development. This guide will walk you through the steps to set up your development environment and build your first contract.

Video Tutorial
^^^^^^^^^^^^^^

For a video walkthrough of this guide, feel free to check out this quick-start video.

.. raw:: html 

   <iframe width="560" height="315" src="https://www.youtube.com/embed?v=XZsc7YiJ12M&list=PL8oWxbJ-csErqfzYvbWsMUr4IvwRVenni&index=1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Prerequisites 
^^^^^^^^^^^^^

Installing Rust
###############
Install the `Rust programming language <https://www.rust-lang.org>`_ if you don’t already have it on your computer. 

The `official Rust guide <https://www.rust-lang.org/tools/install>`_ recommends installing Rust by using ``curl``:

.. code::

   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

You can also use ``brew`` on MacOS or ``apt`` on Linux to install Rust.

You will need to customize the build so that you are installing the nightly build as your default toolchain. Once you finish installing Rust, check your version:

.. code::

   rustup --version

Your terminal output should resemble something like the following (note: at the time of the writing of this tutorial, the latest version of Rust was 1.23.1 and may differ for you):

.. code::

   $ rustup --version
   rustup 1.23.1 (3df2264a9 2020-11-30)
   info: This is the version for the rustup toolchain manager, not the rustc compiler.
   info: The currently active `rustc` version is `rustc 1.52.0-nightly (0fc6756b4 2021-02-08)`

**Casper Rust Packages**

We publish three crates on `crates.io <https://crates.io/>`_ to support smart contract development with Rust:

*  `Casper Contract <https://crates.io/crates/casper-contract>`_ - a library supporting communication with the blockchain. This is the main library you will need to write smart contracts. 
*  `Casper Test Support <https://crates.io/crates/casper-engine-test-support>`_ - an in-memory virtual machine against which you can test your smart contracts.
*  `Casper Types <https://crates.io/crates/casper-types>`_ - a library with types we use across the Rust ecosystem.

A crate is a compilation unit, which can be compiled into a binary or a library.

**API Documentation for Smart Contracts**

Each of the Casper crates comes with API documentation and examples for each function, located at `https://docs.rs <https://docs.rs/releases/search?query=casper>`_. The contract API documentation is specific for a given version. For example, you can find documentation for version **0.7.6** at `<https://docs.rs/casper-contract/0.7.6>`_.


Installing Dependencies
#######################

**1. CMake**

CMake is a popular build tool that we will utilize, and you may very well have it already installed. If you do, make sure that you have the latest version (currently 3.19.4). If you need to install or upgrade it, follow the steps located here: https://cmake.org/install/. Once installed, check your version as shown below. It should resemble this output:

.. code::

   $ cmake --version
   cmake version 3.19.4

   CMake suite maintained and supported by Kitware (kitware.com/cmake).


**2. Google Protobuf Compiler**

Next, we need to install ``protoc``, which is used for serializing structured data. Installation on MacOS or Linux is shown below, but other operating systems should refer to the documentation: https://google.github.io/proto-lens/installing-protoc.html.


Linux with :code:`apt`: 

.. code::

   sudo apt install protobuf-compiler


MacOS with :code:`brew`:

.. code::

   brew install protobuf

Once the Google protobuf compiler is installed, you can check your version:

.. code::

   $ protoc --version
   libprotoc 3.14.0


Development Environment Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Installing the Casper Crates
############################
The best and fastest way to set up a Casper Rust Smart Contract project is to use :code:`cargo-casper`.  When you use this, it will set the project up with a simple contract, a runtime environment and a testing framework with a simple test. *Cargo* is a build system and package manager for Rust (much like *pip* if you are familiar with Python). It is possible to use this configuration in your CI/CD pipeline as well.

.. code::

   cargo install cargo-casper


Creating a Project
##################
You can create a new sample project very easily with the Casper crate. For example, let’s say that I want to create a project named **my-project** for this tutorial (you can choose a different name if you wish), then I can simply run the command:

.. code::

   cargo casper my-project

If you look inside the newly-created *my-project* folder, you will find two crates: ``contract`` and ``tests``. This is a complete basic smart contract that saves a value, passed as an argument, on the blockchain. The ``tests`` crate provides a runtime environment of the Casper virtual machine, and a basic smart contract test.

Compiling to WASM
#################

The Casper blockchain uses WebAssembly (WASM) in its runtime environment. Compilation targets for WASM are available for Rust, giving developers access to all the Rust ecosystem tools when developing smart contracts.

Casper contracts support Rust tooling such as :code:`clippy` for linting contracts. Feel free to use them!

The project requires a specific nightly version of Rust and requires a WASM target to be added to that Rust version. You can see more information by running:

.. code::

   cargo casper --help

To build the project, go into the ``contract`` folder, install the Rust toolchain and specify the target build as WebAssembly (wasm32):

.. code::

   cd contract
   rustup install $(cat rust-toolchain)
   rustup target add --toolchain $(cat rust-toolchain) wasm32-unknown-unknown


Build the Contract
##################

The next step is to compile the smart contract into WASM. Inside the contract folder, run the following command:

.. code::

   cargo build --release

**NOTE: It's important to build the contract using ``--release`` as a debug build will produce a contract which is much larger and more expensive to execute.**

Inside of the folder, you will now see a *target* folder that will contain the compiled smart contract named *contract.wasm* at :code:`my-project/contract/target/wasm32-unknown-unknown/release/contract.wasm`.


Test the Contract
#################

In addition to creating the contract, the Casper crate also automatically created sample tests in the *my-project/tests* folder.

The Casper local environment provides an in-memory virtual machine against which you can run your contract for testing. When you run the test crate, it will automatically build the smart contract in release mode and then run a series of tests against it in the Casper runtime environment. The custom build script is named ``build.rs`` if you are interested in looking more into it.

**Note**: Since the test script automatically builds the contract, during development you only need to run the command ``cargo test`` without the need for ``cargo build``.

A successful test run indicates that your smart contract environment is set up correctly.

.. code::

   cd ../tests
   cargo test

After the compilation finishes, the test should run and you should see output similar to this message in your terminal:

.. code::

   running 1 test
   test tests::should_store_hello_world ... ok

   test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.30s

As a brief example, open up *my-project/contract/src/main.rs* in your editor, modify the *KEY* value in the contract, and then rerun the ``cargo test`` command. You should observe that the smart contract recompiles and the test fails now.

Rust Resources
^^^^^^^^^^^^^^

These Rust resources are excellent and we highly recommend them:

#. https://doc.rust-lang.org/book/foreword.html 
#. https://rustwasm.github.io/docs/book/ 
#. https://doc.rust-lang.org/stable/rust-by-example 