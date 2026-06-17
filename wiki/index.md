# Wiki Index

Registry of all wiki pages, grouped by category. One line per page.

## FBE Board — Firmware Build

- [FBE Firmware Build — Overview](fbe-firmware-build-overview.md) — end-to-end pipeline for the FBE board (aarch64 / IPQ53xx), the two-workspace model, and build vs. rebuild concepts.
- [FBE Source Preparation](fbe-source-prep.md) — shared repo sync, proprietary components, patches, and feed mirrors; per-architecture AFC staging.
- [FBE 32-bit Build](fbe-build-32-bit.md) — build the 32-bit firmware; its purpose is the U-Boot ELF needed for the single image.
- [FBE 64-bit Build](fbe-build-64-bit.md) — build the primary aarch64 firmware on the `generic` target.
- [FBE Single-Image Generation (64-bit)](fbe-single-image.md) — assemble the flashable `norplusnand-ipq5332_64-single.img` from the 64-bit build plus the 32-bit U-Boot.
- [FBE Firmware Rebuild](fbe-rebuild.md) — reuse a prepared workspace to recompile after patches/source edits (full or incremental kernel rebuild).
