# PY32 Peripheral Access Crates

> This repo is modified from [stm32-rs](https://github.com/stm32-rs/stm32-rs)

[![CI](https://github.com/creatoy/py32-rs/workflows/CI/badge.svg?branch=master)](https://github.com/creatoy/py32-rs)
[![crates.io](https://img.shields.io/crates/v/py32f0.svg?label=py32f0)](https://crates.io/crates/py32f0)

This repository provides Rust device support crates for all py32
microcontrollers, providing a safe API to that device's peripherals using
[svd2rust] and a community-built collection of patches to the basic SVD files.
There is one crate per device family, and each supported device is a
feature-gated module in that crate. These crates are commonly known as
peripheral access crates or "PACs".

[svd2rust]: https://github.com/rust-embedded/svd2rust

The py32-rs repository contains the patches to the underlying SVD files and
the tooling to generate the crates.

While these crates are widely used, not every register of every device will
have been tested on hardware, and so errors or omissions may remain. We can't
make any guarantee of correctness. Please report any bugs you find!

## Using Device Crates In Your Own Project

In your own project's `Cargo.toml`:
```toml
[dependencies.py32f0]
version = "0.1.1"
features = ["py32f030"]
```

The `rt` feature is optional but helpful. See
[svd2rust](https://docs.rs/svd2rust/latest/svd2rust/#the-rt-feature) for
details.

Then, in your code:

```rust
use py32f0::py32f030;

let mut peripherals = py32f030::Peripherals::take().unwrap();
```

Refer to `svd2rust` [documentation](https://docs.rs/svd2rust) for further usage.

Replace `py32f0` and `py32f030` with your own device; see the individual
crate READMEs for the complete list of supported devices. All current py32
devices should be supported to some level.

## Generating Device Crates / Building Locally

* Install `svd2rust`, `svdtools`, and `form`:
    * On x86-64 Linux, run `make install` to download pre-built binaries at the
      current version used by py32-rs
    * Otherwise, build using `cargo` (double check versions against `scripts/tool_install.sh`):
        * `cargo install form --version 0.10.0`
        * `cargo install svdtools --version 0.3.0`
        * `cargo install svd2rust --version 0.28.0`
        * `cargo install svd2html --version 0.1.3`
* Install rustfmt: `rustup component add rustfmt`
* Unzip bundled SVD zip files: `cd svd; ./extract.sh; cd ..`
* Generate patched SVD files: `make patch` (you probably want `-j` for all `make` invocations)
    * Alternatively you could install `cargo-make` runner and then use it instead of `make`. Works on MS Windows natively:
        * `cargo install cargo-make`
        * `cargo make patch`
* Generate svd2rust device crates: `make svd2rust`
* Optional: Format device crates: `make form`

## Motivation and Objectives

This project serves two purposes:

* Create a source of high-quality py32 SVD files, with manufacturer errors
  and inconsistencies fixed. These files could be used with svd2rust or other
  tools, or in other projects. They should hopefully be useful in their own
  right.
* Create and publish svd2rust-generated crates covering all py32s, using
  the SVD files.

When this project began, many individual crates existed for specific py32
devices, typically maintained separately with hand-edited updates to the SVD
files. This project hopes to reduce that duplication of effort and centralise
the community's py32 device support in one place.

## Helping

This project is still young and there's a lot to do!

* More peripheral patches need to be written, most of all. See what we've got
  in `peripherals/` and grab a reference manual!
* Also everything needs testing, and you can't so easily automate finding bugs
  in the SVD files...

## Supported Device Families

[![crates.io](https://img.shields.io/crates/v/py32f0.svg?label=py32f0)](https://crates.io/crates/py32f0)

Please see the individual crate READMEs for the full list of devices each crate
supports.

## Adding New Devices

* Update SVD zips in `svd/vendor` to include new SVD.
* Run `svd/extract.sh` to extract the zips into `svd` (ignored in git).
* Add new YAML file in `devices/` with the new SVD path and include any
  required SVD patches for this device, such as renaming or merging fields.
* Add the new devices to `py32_part_table.yaml`.
* Add the new devices `scripts/makecrates.py`.
* You can run `scripts/matchperipherals.py` script to find out what existing
  peripherals could be cleanly applied to this new SVD. If they look sensible,
  you can include them in your device YAML.  This requires a Python environment with the `pyyaml`
  and `svdtools` dependencies.
  Example command: `python scripts/matchperipherals.py peripherals/rcc devices/py32f030.yaml`
* Re-run `scripts/makecrates.py devices/` to update the crates with the new devices.
* Run `make` to rebuild, which will make a patched SVD and then run `svd2rust`
  on it to generate the final library.

If adding a new py32 family (not just a new device to an existing family), complete
these steps as well:

* Add the new devices to the `CRATES` field in `Makefile`.
* Update this Readme to include the new devices.
* Add the devices to `workflows/ci.yaml`.

## Updating Existing Devices/Peripherals

* Using Linux, run `svd/extract.sh` at least once to pull the SVDs out.
* Edit the device or peripheral YAML (see below for format).
* Using Linux, run `make` to rebuild all the crates using `svd patch` and `svd2rust`.
* Test your new stuff compiles: `cd py32f0; cargo build --features py32f030`

If you've added a new peripheral, consider using the `matchperipherals.py`
script to see which devices it would cleanly apply to.

To generate a new peripheral file from scratch, consider using
`periphtemplate.py`, which creates an empty peripheral file based on a single
SVD file, with registers and fields ready to be populated. For single bit wide
fields with names ending in 'E' or 'D' it additionally generates sample
"Enabled"/"Disabled" entries to save time.

## Device and Peripheral YAML Format

Please see the [svdtools](https://github.com/stm32-rs/svdtools) documentation
for full details of the patch file format.


### Style Guide

* Enumerated values should be named in the past tense ("enabled", "masked",
  etc).
* Descriptions should start with capital letters but do not end with a period

## Releasing

Notes for maintainers:

1. Create PR preparing for new release:
    * Update `CHANGELOG.md` with changes since last release and new contributors
    * Update `README.md` to bump version number in example snippet
    * Update `scripts/makecrates.py` to update version number for generated PACs
2. Merge PR once CI passes, pull master locally.
3. `make clean`
4. `make -j16 form`
5. `cd py32f0; pwd; cargo publish --allow-dirty --no-default-features`
6. `git tag -a vX.X.X -m vX.X.X`
7. `git push vX.X.X`

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
