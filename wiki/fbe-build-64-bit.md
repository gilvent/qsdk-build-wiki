---
title: FBE 64-bit Build
tags: [fbe, qsdk, openwrt, firmware-build, ipq53xx, aarch64]
sources: [references/FBE-build-steps-64-bit.txt]
updated: 2026-06-10
---

# FBE 64-bit Build

*Configure and build the FBE 64-bit (aarch64) firmware — the primary target for the board.*

> Sources: references/FBE-build-steps-64-bit.txt — sections "Creating the QSDK build" through "Check build files" (lines 104–141).

Stage 3 of the [pipeline](fbe-firmware-build-overview.md). Requires a prepared tree from
[source preparation](fbe-source-prep.md) (64-bit / `target-aarch64` variant). Run from
inside `qsdk/`.

## 1. Install feeds

```bash
cd qsdk
./scripts/feeds update -a
./scripts/feeds install -a -f
```

## 2. Base config, kept on the generic (aarch64) target

Copy the premium base config and switch `ipq95xx` to `ipq53xx`. Unlike the
[32-bit build](fbe-build-32-bit.md), the target is **left as the generic** `ipq53xx`
(aarch64) — there is no `ipq53xx_32` conversion.

```bash
cp qca/configs/qsdk/ipq_premium.config .config
sed -i 's/TARGET_ipq95xx/TARGET_ipq53xx/g' .config
mkdir -p prebuilt/generic
cp -rf prebuilt/ipq53xx_generic/ipq_premium/* prebuilt/generic
```

## 3. EZMESH configuration

Identical to the 32-bit build — see [FBE 32-bit Build → EZMESH configuration](fbe-build-32-bit.md#3-ezmesh-configuration-full-mode).

## 4. Regenerate config and build

```bash
make defconfig
make V=s
```

## 5. Verify outputs

Build artifacts land in `qsdk/bin/`. Expect files such as:

- `openwrt-ipq53xx*.elf` — ELF files for U-Boot variants
- `openwrt-ipq53xx-generic-squashfs-root.img` — SquashFS
- `openwrt-ipq53xx-generic-qcom_mixx-fit-uImage.itb` — ITB

Next: [single-image generation](fbe-single-image.md) (step 5) combines these `generic`
outputs with the 32-bit U-Boot ELF into the flashable image. To recompile this workspace
later without re-preparing the source, see [rebuild](fbe-rebuild.md).
