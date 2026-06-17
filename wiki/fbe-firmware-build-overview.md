---
title: FBE Firmware Build — Overview
tags: [fbe, qsdk, openwrt, firmware-build, ipq53xx, aarch64]
sources: [references/FBE-build-steps-32-bit.txt, references/FBE-build-steps-64-bit.txt, references/FBE-rebuild-steps-64-bit.txt]
updated: 2026-06-10
---

# FBE Firmware Build — Overview

*End-to-end pipeline for building the flashable single firmware image for the FBE board (aarch64 / IPQ53xx) from QSDK.*

> Sources: references/FBE-build-steps-32-bit.txt (32-bit build); references/FBE-build-steps-64-bit.txt (64-bit build + single image); references/FBE-rebuild-steps-64-bit.txt (rebuild).

The FBE board runs an **aarch64 (64-bit)** firmware, but its boot chain still needs the
**32-bit U-Boot** produced by a separate 32-bit build. The 32-bit and 64-bit builds live in
**two separate workspace checkouts**: the 64-bit (main) workspace assembles the final image
and reaches into the 32-bit workspace for the U-Boot ELF.

**Board**
- Name: FBE
- Architecture: aarch64 (64-bit) — IPQ53xx, target `ipq53xx_generic`
- 32-bit side: target `ipq53xx_ipq53xx_32` / `target-arm` (U-Boot only)

## Concepts

- **Build** — prepare the source, then compile from scratch. Done once per workspace/architecture.
- **Rebuild** — reuse an already-prepared workspace to recompile after patches or source edits, *without* re-running source preparation. Can be full (`make clean`) or an incremental kernel rebuild. See [rebuild](fbe-rebuild.md).
- **Single image** — the flashable composite (`norplusnand-ipq5332_64-single.img`) assembled from the 64-bit build output plus the 32-bit U-Boot.

## Pipeline

1. **Prepare source** — [source preparation](fbe-source-prep.md). Run once per workspace (32-bit and 64-bit variants differ only in the AFC staging paths).
2. **Build 32-bit** — [32-bit build](fbe-build-32-bit.md). Produces the 32-bit U-Boot ELF.
3. **Build 64-bit** — [64-bit build](fbe-build-64-bit.md). Produces the aarch64 firmware (`generic` target).
4. **Reuse the 32-bit U-Boot** — the ELF from step 2 is copied into the 64-bit workspace for image assembly (see step 5).
5. **Generate the single image** — [single-image generation](fbe-single-image.md). Produces `norplusnand-ipq5332_64-single.img`.

To recompile after changes without redoing steps 1–3, see [rebuild](fbe-rebuild.md).
