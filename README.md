# cargo-ndk - Build Rust code for Android

[<img alt="CI" src="https://github.com/topminipie/cargo-ndk/actions/workflows/ci.yml/badge.svg">](https://github.com/topminipie/cargo-ndk/actions/workflows/ci.yml)
<img alt="Minimum supported Rust version: 1.73" src="https://img.shields.io/badge/MSRV-1.70-informational">
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](./LICENSE-APACHE)
<sup>or</sup>
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE-MIT)

This cargo extension handles all the environment configuration needed for successfully building libraries
for Android from a Rust codebase, with support for generating the correct `jniLibs` directory structure.

## Installing

```
cargo install --git https://github.com/topminipie/cargo-ndk.git
```

You'll also need to install all the toolchains you intend to use. Simplest way is with the following:

```
rustup target add \
    aarch64-linux-android \
    armv7-linux-androideabi \
    x86_64-linux-android \
    i686-linux-android
```

Modify as necessary for your use case.

## Usage

If you have installed the NDK with Android Studio to its default location, `cargo ndk` will automatically detect
the most recent NDK version and use it. This can be overriden by specifying the path to the NDK root directory in
the `ANDROID_NDK_HOME` environment variable.

### Examples

#### Building a library for 32-bit and 64-bit ARM systems

```
cargo ndk -t armeabi-v7a -t arm64-v8a -o ./jniLibs build --release
```

This specifies the Android targets to be built (ordinary triples are also supported), the output directory to use for placing the `.so` files in the layout
expected by Android, and then the ordinary flags to be passed to `cargo`.

![Example](./example/example.svg)

#### Linking against and copying `libc++_shared.so` into the relevant places in the output directory

Create a `build.rs` in your project with the following:

```rust
use std::{env, path::{Path, PathBuf}};

fn main() {
    if env::var("CARGO_CFG_TARGET_OS").unwrap() == "android" {
        android();
    }
}

fn android() {
    println!("cargo:rustc-link-lib=c++_shared");

    if let Ok(output_path) = env::var("CARGO_NDK_OUTPUT_PATH") {
        let sysroot_libs_path =
            PathBuf::from(env::var_os("CARGO_NDK_SYSROOT_LIBS_PATH").unwrap());
        let lib_path = sysroot_libs_path.join("libc++_shared.so");
        std::fs::copy(
            lib_path,
            Path::new(&output_path)
                .join(&env::var("CARGO_NDK_ANDROID_TARGET").unwrap())
                .join("libc++_shared.so"),
        )
        .unwrap();
    }
}
```

### Controlling verbosity

Add `-v` or `-vv` as you ordinarily would after the cargo command.

### Providing environment variables for C dependencies

`cargo-ndk` derives which environment variables to read the same way as the `cc` crate.

### `cargo-ndk`-specific environment variables

These environment variables are exported for use in build scripts and other downstream use cases:

- `CARGO_NDK_ANDROID_PLATFORM`: the Android platform API number as an integer (e.g. `21`)
- `CARGO_NDK_ANDROID_TARGET`: the Android name for the build target (e.g. `armeabi-v7a`)
- `CARGO_NDK_OUTPUT_PATH`: the output path as specified with the `-o` flag
- `CARGO_NDK_SYSROOT_PATH`: path to the sysroot inside the Android NDK
- `CARGO_NDK_SYSROOT_TARGET`: the target name for the files inside the sysroot (differs slightly from the standard LLVM triples)
- `CARGO_NDK_SYSROOT_LIBS_PATH`: path to the libraries inside the sysroot with the given sysroot target (e.g. `$CARGO_NDK_SYSROOT_PATH/usr/lib/$CARGO_NDK_SYSROOT_TARGET`)
## Supported hosts

- Linux
- macOS (`x86_64` and `arm64`)
- Windows

## Why not using cargo-ndk?

There is no feature [#43](https://github.com/bbqsrc/cargo-ndk/pull/43) in [cargo-ndk](https://github.com/bbqsrc/cargo-ndk).

[Cargo-xdk](https://github.com/tiann/cargo-xdk) is no updated and synchronized with [cargo-ndk](https://github.com/bbqsrc/cargo-ndk).

## Similar projects

* [cargo-cocoapods](https://github.com/bbqsrc/cargo-cocoapods) - for building .a files for all Apple platforms, and bundling for CocoaPods

## License

This project is licensed under either of

 * Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or https://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or https://opensource.org/licenses/MIT)

at your option.
