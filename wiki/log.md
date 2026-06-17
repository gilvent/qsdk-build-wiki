# Wiki Log

## [2026-06-10] QUERY | New wiki: build firmware image with QSDK for the FBE board (aarch64)
- Pages: created wiki/fbe-firmware-build-overview.md, wiki/fbe-source-prep-32-bit.md, wiki/fbe-build-32-bit.md, wiki/fbe-single-image.md; created wiki/index.md; created references/README.md.
- Sources: references/FBE-build-steps-32-bit.txt (prep lines 3–100, build lines 103–140, single image lines 142–181); references/FBE-build-steps-64-bit.txt (registry only).
- Notes: Moved FBE-rebuild-steps-64-bit.txt from repo root into references/FBE-build-steps-64-bit.txt. Confirmed with user: 32-bit and 64-bit builds use two separate workspaces. 64-bit build page deferred — user flagged the 64-bit reference as containing a mistake and will supply a correction.

## [2026-06-10] EDIT | Reconcile 64-bit build vs. rebuild after corrected references supplied
- Pages: added wiki/fbe-build-64-bit.md, wiki/fbe-rebuild.md; renamed wiki/fbe-source-prep-32-bit.md → wiki/fbe-source-prep.md (now shared across both arches); re-anchored wiki/fbe-single-image.md to the 64-bit flow; updated wiki/fbe-build-32-bit.md, wiki/fbe-firmware-build-overview.md, wiki/index.md, references/README.md.
- Sources: references/FBE-build-steps-64-bit.txt (full 64-bit build + single image, lines 104–181); references/FBE-rebuild-steps-64-bit.txt (rebuild, lines 1–38); references/FBE-build-steps-32-bit.txt (shared prep).
- Notes: User clarified the concepts — FBE-build-steps-64-bit.txt = the 64-bit (aarch64) target build; FBE-rebuild-steps-64-bit.txt = reusing the same prepared environment to rebuild. Introduced "build" vs "rebuild" (full / incremental kernel) terminology. Consolidated near-identical 32/64-bit prep into one page with a per-arch AFC staging block. Single-image generation re-homed to the 64-bit build (it is a 64-bit image; the block is duplicated in the 32-bit reference).

## [2026-06-10] EDIT | Resolve the single-image config.xml path "inconsistency"
- Pages: updated wiki/fbe-single-image.md.
- Sources: verified against the already-built workspace ../ipq5321-networking-2024-spf-12-5_csu1-64bit (sed -i.bak left config.xml.bak at apss_proc/out/meta-tools/ipq5332/, update_common_info.py resides in common/build/).
- Notes: The `../../apss_proc/...config.xml` paths are not a bug. The final block (config.xml seds + update_common_info.py) executes from common/build/, so `../../apss_proc` resolves to the workspace-root apss_proc/ — confirmed by the .bak location and applied substitution. Made the `cd common/build` step explicit and removed the inconsistency caveat.
