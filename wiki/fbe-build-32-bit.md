---
title: FBE 32-bit Build
tags: [fbe, qsdk, openwrt, firmware-build, ipq53xx, u-boot]
sources: [references/FBE-build-steps-32-bit.txt]
updated: 2026-06-10
---

# FBE 32-bit Build

*Configure and build the FBE 32-bit firmware; its U-Boot ELF feeds the 64-bit single image.*

> Sources: references/FBE-build-steps-32-bit.txt — sections "Creating the QSDK build" through "Check build files" (lines 103–140).

Stage 2 of the [pipeline](fbe-firmware-build-overview.md). Requires a prepared tree from
[source preparation](fbe-source-prep.md) (32-bit / `target-arm` variant). Run from inside
`qsdk/`.

> FBE is a 64-bit board, so this 32-bit build exists primarily to produce the **U-Boot ELF**
> consumed by the [64-bit single image](fbe-single-image.md). The 64-bit firmware itself comes
> from the [64-bit build](fbe-build-64-bit.md).

## 1. Install feeds

```bash
cd qsdk
./scripts/feeds update -a
./scripts/feeds install -a -f
```

## 2. Base config, retargeted to ipq53xx 32-bit

Copy the premium base config and switch the target from `ipq95xx` to the 32-bit `ipq53xx`.

```bash
cp qca/configs/qsdk/ipq_premium.config .config
sed -i 's/TARGET_ipq95xx_generic/TARGET_ipq53xx_ipq53xx_32/g' .config
sed -i 's/TARGET_ipq95xx/TARGET_ipq53xx/g' .config
cp -rf prebuilt/ipq53xx_32/ipq_premium/* prebuilt/ipq53xx_32/
```

## 3. EZMESH configuration (full mode)

```bash
echo "CONFIG_EZMESH_ALG_SOURCE=n" >> .config
echo "CONFIG_PACKAGE_whc-mesh=n" >> .config
echo "CONFIG_PACKAGE_hyfi-mesh=n" >> .config
echo "CONFIG_PACKAGE_whc-map=y" >> .config
echo "CONFIG_PACKAGE_hyfi-map=y" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh-cmn=y" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh=y" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh-alg=y" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh-ctrl=n" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh-agent=n" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh-agentalg=n" >> .config
echo "CONFIG_PACKAGE_qca-ezmesh-alg-static=n" >> .config
echo CONFIG_BUILD_SHORTENED_PATH=y >> .config
```

## 4. Regenerate config and build

```bash
make defconfig
make V=s
```

## 5. Verify outputs

Build artifacts land in `qsdk/bin/`. Expect files such as:

- `openwrt-ipq53xx*.elf` — ELF files for U-Boot variants
- `openwrt-ipq-ipq53xx-squashfs-root.img` — SquashFS
- `openwrt-ipq-ipq53xx-qcom_mixx-fit-uImage.itb` — ITB

> Keep the 32-bit **U-Boot ELF** (`openwrt-ipq53*-u-boot*.elf`). It is required by
> [single-image generation](fbe-single-image.md) (step 5), which copies it from this
> 32-bit workspace into the 64-bit workspace.
