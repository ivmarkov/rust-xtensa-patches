# Rust Xtensa patches from [MabezDev](https://github.com/MabezDev/rust-xtensa) and Rust ESP32 STD patches

This repo keeps all patches that enable Rust support for the ESP32 chip family.
Currently, these patches are applied in the `stable_V1.53.0` branch of [https://github.com/ivmarkov/rust](https://github.com/ivmarkov/rust),
which is to become the official `stable` branch of that repo.

# Rust Xtensa patches

The following patches are a direct copy of the patchset applied at the [MabezDev](https://github.com/MabezDev/rust-xtensa) repository
They were generated with `git format-patch` directly from the `xtensa-target` branch, as I was unsure whether the upstream of this repo contains patches which are up-to-date for Rust V1.53.0:
* [0001](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0001-Use-llvm-submodule-with-Xtensa-arch-support.patch)
* [0002](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0002-Teach-rustc-about-the-Xtensa-arch.patch)
* [0003](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0003-Teach-rustc-about-the-Xtensa-call-ABI.patch)
* [0004](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0004-Add-some-Xtensa-targets.patch)
* [0005](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0005-Teach-rust-core-about-Xtensa-VaListImpl.patch)
* [0006](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0006-Update-readme.patch)
* [0007](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0007-Fix-missing-import-in-unsupported-args.patch)

These patches essentially switch the Rust compiler to the [forked Espressif LLVM repo](https://github.com/espressif/llvm-project) which does have support for the Xtensa family of processors (the ESP32 non-'C' family of MCUs), and then implements the following new Rust targets:
* `xtensa-esp32-none-elf`
* `xtensa-esp32s2-none-elf`
* `xtensa-esp32s3-none-elf` - future, not there yet because seemingly the Espressif LLVM repo does not contain support for the ESP32-S3 Xtensa processor yet
* `xtensa-esp8266-none-elf`

# Patch [0008](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0008-STD-support-fill-in-os_family-env-and-vendor-informa.patch) - fill-in `target_family`, `target_os`, `target_env` and `target_vendor` for the Xtensa targets from above

These parameters are necessary, because as described in patch 0012 below, the Rust STD support is implemented based on conditional statements by these target parameters.
The parameters are set as follows:
  *  `target_family` = unix
  *  `target_os` = none
  *  `target_env` = newlib
  *  `target_vendor` = espressif

**NOTE:** The `target_family` = unix parameter is  key, as it directs Rust STD to apply its `sys/unix` implementation on the ESP32 chip family. This is the easiest way to implement Rust STD for the ESP-IDF, as the ESP-IDF SDK has a relatively complete POSIX API.

# Removal of atomics for `xtensa-esp32s2-none-elf`

Patch [0009](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0009-Remove-claims-for-atomics-support-from-the-esp32-s2-.patch) removes the declared atomics support for ESP32-S2 as it actually does not exist and is not emulated yet either (see [this issue](https://github.com/espressif/rust-esp32-example/issues/3) for more details).

**NOTE:** This patch can be independently applied from all others or skipped completely, however if you decide to apply it, make sure you've applied patches 0001 to 0004 first.

# Patch [0010](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0010-Create-a-dedicated-riscv32imc-target-for-esp32-c3-wi.patch) - custom RISCV32IMC target

ESP32-C3 is using the RISCV32IMC instruction set.
* This patch implements a custom RISCV32IMC target - `riscv32imc-esp32c3-none-elf`, based off from the standard Rust `riscv32imc-unknown-unknown`.
* The `riscv32imc-unknown-unknown` standard Rust target does not have information about `target_family`, `target_os`, `target_env` and `target_vendor`.
* The `riscv32imc-esp32c3-none-elf` sets these as follows:
  *  `target_family` = unix
  *  `target_os` = none
  *  `target_env` = newlib
  *  `target_vendor` = espressif

The above is necessary once the Rust STD support get enabled for ESP32-C3 too, which will happen once the atomics support required for Rust STD is implemented, as per [this issue](https://github.com/espressif/rust-esp32-example/issues/3).

**NOTE:** This patch can be independently applied from all others or skipped completely.

# Patch [0011](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0011-Extra-branch-in-panic_unwind-for-target_family-unix-.patch) - instruct the `panic_unwind` crate that targets with `target_os` = none and `target_vendor` = espressif are NOT supporting stack unwinding

This patch is necessary, because even though the `panic_unwind` crate treats `target_os` = none correctly, it does NOT do so for targets which have specified `target_family` = unix, as we did in patch 0010 from above.

**NOTE:** This patch can be independently applied from all others or skipped completely, but it is necessary for the Rust STD support on ESP32 to compile with one of the targets defined above.

# Patch [0012](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0012-ESP32-STD-support-based-on-ESP-IDF.patch) - Support for Rust STD for ESP32 chips family

This is an all-in-one patch that patches ONLY the STD library of Rust (i.e. `rust/library/src/std`).
It enables the `sys/unix` codepath in the Rust STD library for the Espressif chip family, by using `#[cfg]` conditionals based on the `target_family`, `target_os`, `target_env` (rarely) and `target_vendor`, as defined up to 0010.

**NOTE:** This patch can be independently applied from all others or skipped completely, but it is necessary for the Rust STD support on ESP32 to compile with one of the targets defined above.

# Patch [0013](https://github.com/ivmarkov/rust-xtensa-patches/blob/master/0013-Custom-Readme-file-describing-the-changes-done-on-to.patch) - Custom Readme file

This is the `README.md` file you see when opening [The Rust ESP32 STD GitHub repo](https://github.com/ivmarkov/rust/tree/stable_V1.53.0).

**NOTE:** This patch can be independently applied from all others or skipped completely.
