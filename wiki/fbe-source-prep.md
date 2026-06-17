---
title: FBE Source Preparation
tags: [fbe, qsdk, openwrt, source-prep, ipq53xx, aarch64]
sources: [references/FBE-build-steps-32-bit.txt, references/FBE-build-steps-64-bit.txt]
updated: 2026-06-10
---

# FBE Source Preparation

*Prepare a QSDK source tree (repo sync, proprietary components, patches) for an FBE build. The procedure is shared by both architectures; only the AFC staging paths differ.*

> Sources: references/FBE-build-steps-32-bit.txt and references/FBE-build-steps-64-bit.txt — sections "Prepare necessary files" through "Update OpenWrt git urls" (lines 3–101). The two files are identical here except for §4.

Stage 1 of the [pipeline](fbe-firmware-build-overview.md). Run once **per workspace** (the
32-bit and 64-bit builds live in [separate workspaces](fbe-firmware-build-overview.md)),
from the Qualcomm code working directory (the parent of `qsdk/`). The result is a populated
local `qsdk/` tree, ready for the [32-bit](fbe-build-32-bit.md) or
[64-bit](fbe-build-64-bit.md) build.

## 1. Prepare necessary files

Remove stale component drops and flatten the per-component subdirectories into the working dir.

```bash
rm -rf CNSS.PS.* IPQ6018.ILQ.* WLAN.BL.* IPQ5018.ILQ.* IPQ8074.ILQ.* NSS.FW.* \
  IPQ9574.ILQ.* WLAN.HK.* TZ.WNS.4.0 BOOT.BF.* BTFW.MAPLE.* BOOT.XF.0.3 TZ.WNS.5.1 \
  RPM.BF.2.4.1 TMEL.WNS.1.0 TZ.BF.*
cp -rf */* .
umask 022
```

## 2. Repo init & sync

Fetch the `repo` tool and sync the QSDK 12.5 manifest.

```bash
mkdir -p ~/bin
curl -o ~/bin/repo https://storage.googleapis.com/git-repo-downloads/repo
chmod a+x ~/bin/repo
export PATH="$HOME/bin:$PATH"
hash -r
repo init \
  -u https://git.codelinaro.org/clo/qsdk/releases/manifest/qstak \
  -b release \
  -m AU_LINUX_QSDK_NHSS.QSDK.12.5_TARGET_ALL.12.5.2783.2987.xml \
  --repo-url=https://gerrit.googlesource.com/git-repo \
  --repo-rev=stable \
  --no-clone-bundle

# Fix the remote name in the manifest, then sync
sed -i 's/remote="qclo"/remote="clo"/g' \
  .repo/manifests/AU_LINUX_QSDK_NHSS.QSDK.12.5_TARGET_ALL.12.5.2783.2987.xml
repo sync -j8 --no-tags -qc
```

## 3. Copy proprietary components into qsdk

```bash
mkdir -p qsdk/dl
cp -rf apss_proc/out/proprietary/Wifi/qsdk-qca-wifi/* qsdk
cp -rf apss_proc/out/proprietary/Wifi/qsdk-qca-wlan/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/meta-tools/ .
cp -rf apss_proc/out/proprietary/QSDK-Base/common-tools/* qsdk/
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-lib/* qsdk/
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-mcs-apps/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-qmi-framework/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/gpio-debug/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-diag/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-cnss-daemon/* qsdk
cp -rf apss_proc/out/proprietary/qca-diag-bin/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/athtestcmd/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/fw-qca-stats/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/qsdk-qca-athdiag/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/nss-prop-user/* qsdk
cp -rf apss_proc/out/proprietary/Hyfi/qsdk-ieee1905-security/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/minidump/* qsdk
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-afcsec-bin/* qsdk/
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-pkgmgr-bin/* qsdk/
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-prov-src/* qsdk/
cp -rf apss_proc/out/proprietary/QSDK-Base/qca-native-bin/* qsdk/
rm -rfv qsdk/qca/feeds/qca/utils/ctrl_app_dut
cp -rf apss_proc/out/proprietary/RBIN-SOFTSKU/qca-softsku-bin/* qsdk
```

## 4. AFC adjustments

Copy the AFC components and keep only the provisioning/encrypt feeds:

```bash
cp -rf apss_proc/out/proprietary/RBIN-AFCAgent/qca-afc-bin/* qsdk
cp -rf apss_proc/out/proprietary/RBIN-AFCAgent/qti_provisioning/* qsdk
cd qsdk/qca/feeds/afc
find -maxdepth 1 ! -name qti-mfg-provision ! -name qti-encrypt ! -name . -exec rm -rv {} \;
cd ../../../../
```

Then stage the prebuilt provisioning library. **This is the only architecture-specific step** — use the staging dir and prebuilt path matching the workspace:

**32-bit workspace** (`target-arm`, prebuilt `ipq53xx_32`):

```bash
mkdir -pv qsdk/staging_dir/target-arm/usr/lib/
cd qsdk/prebuilt/ipq53xx_32/ipq_premium
cp -fv ./libprovision.so ../../../staging_dir/target-arm/usr/lib/
cd ../../../../
mkdir -p qsdk/staging_dir/target-arm/pkginfo/
touch qsdk/staging_dir/target-arm/pkginfo/qti-mfg-provision.provides
echo libprovision.so > qsdk/staging_dir/target-arm/pkginfo/qti-mfg-provision.provides
```

**64-bit workspace** (`target-aarch64`, prebuilt `ipq53xx_generic`):

```bash
mkdir -p qsdk/staging_dir/target-aarch64/usr/lib/
cd qsdk/prebuilt/ipq53xx_generic/ipq_premium/
cp -fv ./libprovision.so ../../../staging_dir/target-aarch64/usr/lib/
cd ../../../../
mkdir -p qsdk/staging_dir/target-aarch64/pkginfo/
touch qsdk/staging_dir/target-aarch64/pkginfo/qti-mfg-provision.provides
echo libprovision.so > qsdk/staging_dir/target-aarch64/pkginfo/qti-mfg-provision.provides
```

## 5. HyFi and WHC packages

```bash
cp -rf apss_proc/out/proprietary/Hyfi/hyfi/* qsdk
cp -rf apss_proc/out/proprietary/Hyfi/qsdk-whc/* qsdk
mkdir qsdk/qca/feeds/qca-son-mem-debug/qca-son-mem-debug
mv qsdk/qca/feeds/qca-son-mem-debug/Makefile \
   qsdk/qca/feeds/qca-son-mem-debug/Config.in \
   qsdk/qca/feeds/qca-son-mem-debug/qca-son-mem-debug
cp -rf apss_proc/out/proprietary/Hyfi/qsdk-whcpy/* qsdk
```

The local `qsdk/` directory is now created.

## 6. Apply patches

`patches/` is placed in the same directory; apply the series with quilt.

```bash
quilt push -a
```

## 7. Make files executable

```bash
chmod a+x ./qsdk/package/base-files/files/etc/rc.local
chmod a+x ./qsdk/package/base-files/files/etc/set-eth01.sh
chmod a+x ./qsdk/package/base-files/files/sbin/rgb_pwm.sh
chmod a+x ./qsdk/package/base-files/files/sbin/poestat
```

## 8. Update OpenWrt git URLs to mirrors

```bash
sed -i 's|git.openwrt.org/project/luci.git|github.com/openwrt/luci.git|g' feeds.conf
sed -i 's|git.openwrt.org/feed/packages.git|github.com/openwrt/packages.git|g' feeds.conf.default
sed -i 's|git.openwrt.org/feed/routing.git|github.com/openwrt/routing.git|g' feeds.conf.default
```

> Some entries in `feeds.conf.default` are not covered by the commands above; update those to their mirrors too.

Next: [32-bit build](fbe-build-32-bit.md) or [64-bit build](fbe-build-64-bit.md).
