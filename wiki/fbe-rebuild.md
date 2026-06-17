---
title: FBE Firmware Rebuild
tags: [fbe, qsdk, rebuild, incremental, ipq53xx, aarch64]
sources: [references/FBE-rebuild-steps-64-bit.txt]
updated: 2026-06-10
---

# FBE Firmware Rebuild

*Reuse an already-prepared workspace to recompile the firmware after applying patches or editing source — without re-running source preparation.*

> Sources: references/FBE-rebuild-steps-64-bit.txt (lines 1–38).

A **rebuild** skips [source preparation](fbe-source-prep.md) (no repo sync, no proprietary
copy) and recompiles an existing `qsdk/` tree. Use it after changing patches or source.
Two modes: a **full rebuild** (clean) and an **incremental kernel rebuild**.

## 1. Apply patches

```bash
quilt push -a
```

## 2. Full rebuild (clean)

`make clean` wipes build output, so the per-architecture staging of the prebuilt
provisioning library must be redone. Use the block for the workspace you are rebuilding.

**32-bit workspace** (`target-arm`):

```bash
cd qsdk
make clean
mkdir -pv staging_dir/target-arm/usr/lib/
cp -fv prebuilt/ipq53xx_32/ipq_premium/libprovision.so staging_dir/target-arm/usr/lib/
mkdir -p staging_dir/target-arm/pkginfo/
touch staging_dir/target-arm/pkginfo/qti-mfg-provision.provides
echo libprovision.so > staging_dir/target-arm/pkginfo/qti-mfg-provision.provides
```

**64-bit workspace** (`target-aarch64`):

```bash
cd qsdk
make clean
mkdir -pv staging_dir/target-aarch64/usr/lib/
cp -fv prebuilt/ipq53xx_generic/ipq_premium/libprovision.so staging_dir/target-aarch64/usr/lib/
mkdir -p staging_dir/target-aarch64/pkginfo/
touch staging_dir/target-aarch64/pkginfo/qti-mfg-provision.provides
echo libprovision.so > staging_dir/target-aarch64/pkginfo/qti-mfg-provision.provides
```

Then build:

```bash
make V=s
```

## 3. Incremental kernel rebuild

When only the Linux kernel source or config changed, skip `make clean` and rebuild just the kernel.

```bash
cd qsdk
make target/linux/compile V=s
make target/linux/install V=s
```

## 4. Regenerate the single image

Re-run [single-image generation](fbe-single-image.md) to produce the updated flashable
`norplusnand-ipq5332_64-single.img`.
