# OnePlus 15 GKI Build

Stock-equivalent direct-GKI build workflow for OnePlus 15.

## Source

- Kernel source: `https://github.com/OnePlusOSS/android_kernel_common_oneplus_sm8850`
- Branch: `oneplus/sm8850_b_16.0.0_oneplus_15`
- Pinned commit: `d9053b907db4bb5da938e9cf947d0ae32302ceaf`
- Kernel release: `6.12.23-4k`

## Toolchain

- Android clang `r536225` / `14043575`
- Android Rust toolchain `linux-12909517`
- Build tools from `cctv18/oneplus_sm8650_toolchain`, release `LLVM-Clang19-r536225`

## Build Method

The workflow downloads the official source archive and toolchain, then builds:

```sh
make O=out gki_defconfig
make O=out olddefconfig
make O=out Image
```

Required baseline outputs and metadata are kept in `reference/`:

- `BUILD_INFO.txt`
- `kernel_built_config`
- `modules.builtin`
- `System.map`
- `vmlinux.symvers`
- `SHA256SUMS.txt`

## Repack Notes

Use `magiskboot.exe` with a stock OnePlus 15 boot image. Do not use AnyKernel.

Recommended flow:

```sh
magiskboot unpack stock-boot.img
rm kernel_dtb
cp Image kernel
magiskboot repack stock-boot.img new-boot.img
```

OP15 stock boot images are header v4 with a raw ARM64 kernel and no boot
ramdisk. The ramdisk is in `init_boot`, and the device DTB is in `vendor_boot`.
Do not modify `vendor_boot`, `init_boot`, standalone `vbmeta`, or
`vbmeta_system` for this kernel-only repack.

`magiskboot unpack` may split the raw kernel at an FDT-looking stub and create a
`kernel_dtb` file. For this device, that file is the kernel tail from an
incorrect split, not a device DTB. Delete `kernel_dtb` before repacking, replace
`kernel` with the direct-GKI `Image`, and verify the repacked boot image by
unpacking it again and confirming the reconstructed kernel bytes match the
`Image` used for repack.

Always RAM boot-test before flashing persistently:

```sh
fastboot boot new-boot.img
fastboot flash boot_a new-boot.img
fastboot set_active a
fastboot reboot
```
