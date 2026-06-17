---
title: FBE Single-Image Generation (64-bit)
tags: [fbe, qsdk, single-image, ipq53xx, aarch64]
sources: [references/FBE-build-steps-64-bit.txt]
updated: 2026-06-10
---

# FBE Single-Image Generation (64-bit)

*Assemble the flashable 64-bit single image, combining the 64-bit build output with the 32-bit U-Boot ELF.*

> Sources: references/FBE-build-steps-64-bit.txt — section "Finally, generate a single firmware image" (lines 143–181). (The same block also appears in the 32-bit reference.)

Stage 5 of the [pipeline](fbe-firmware-build-overview.md), run in the **64-bit (main)
workspace** after the [64-bit build](fbe-build-64-bit.md). Step 1 runs from the Qualcomm
code working directory (the parent of `qsdk/`); step 2 continues from `common/build/`. It
pulls the U-Boot ELF from the separate [32-bit build](fbe-build-32-bit.md) workspace.

## 1. Stage images and tooling into `common/build/ipq_x64`

```bash
mkdir -p common/build/ipq_x64
mkdir -p apss_proc/out/meta-scripts
cp qsdk/qca/src/u-boot-2016/tools/pack_v2.py apss_proc/out/meta-scripts/
sed -i 's#</linux_root_path>#/</linux_root_path>#' contents.xml
sed -i 's#</windows_root_path>#\\</windows_root_path>#' contents.xml
cp qsdk/bin/targets/ipq53xx/generic/openwrt* common/build/ipq_x64
```

Copy the saved 32-bit U-Boot ELF (step 4) from the separate 32-bit workspace:

```bash
cp ../ipq5321-networking-2024-spf-12-5_csu1-32bit/qsdk/bin/targets/ipq53xx/ipq53xx_32/openwrt-ipq53*-u-boot*.elf common/build/ipq_x64/
```

Copy the remaining tools, DTBs, and firmware images:

```bash
cp -rf apss_proc/out/proprietary/QSDK-Base/meta-tools apss_proc/out/
cp qsdk/staging_dir/host/bin/mkimage apss_proc/out/meta-tools/
cp -rf qsdk/bin/targets/ipq53xx/generic/dtbs/* common/build/ipq_x64/
cp -rf skales/* common/build/ipq_x64/
cp qsdk/staging_dir/host/bin/ubinize common/build/ipq_x64/
cp -rf WLAN.WBE*/wlan_proc/build/ms/bin/*/FW_IMAGES/* common/build/ipq_x64/
cp -rf TMEL.WNS*/patch/signed/tmel-ipq53xx-patch.elf common/build/ipq_x64/
```

## 2. Select the wifi firmware images and assemble (from `common/build`)

The remaining steps run from `common/build/`, where `update_common_info.py` lives. The
`../../apss_proc/...` paths therefore resolve back to the workspace-root `apss_proc/`.

```bash
cd common/build

# Pick the IPQ53xx + QCN64xx wifi firmware images
sed -i.bak 's#wifi_fw_ipq5332_qcn9224_v2_qcn9160_squashfs.img#wifi_fw_ipq5332_qcn9224_v2_single_dualmac_squashfs.img#g' ../../apss_proc/out/meta-tools/ipq5332/config.xml
sed -i.bak 's#wifi_fw_ipq5332_qcn9224_v2_qcn6432_qcn9160_squashfs.img#wifi_fw_ipq5332_qcn6432_squashfs.img#g' ../../apss_proc/out/meta-tools/ipq5332/config.xml
sed -i.bak 's#wifi_fw_ipq5332_qcn6432cs_qcn9160_squashfs.img#wifi_fw_ipq5332_qcn6432_squashfs.img#g' ../../apss_proc/out/meta-tools/ipq5332/config.xml

# Build the single image
export BLD_ENV_BUILD_ID=P
python update_common_info.py 64
```

## 3. Verify

The flashable image should appear in `common/build/bin`:

```
norplusnand-ipq5332_64-single.img
```
