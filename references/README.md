# References

Raw, immutable build sources for the QSDK Build Wiki. Read, never edit. Text sources are
tracked here; binaries (PDFs, images) are gitignored and only their provenance is noted.

| File | Description |
|------|-------------|
| `FBE-build-steps-32-bit.txt` | FBE board — full 32-bit firmware build (source preparation → build). Produces the U-Boot ELF consumed by the 64-bit single image. |
| `FBE-build-steps-64-bit.txt` | FBE board — full 64-bit (aarch64) firmware build (source preparation → build → single-image generation). The primary target. |
| `FBE-rebuild-steps-64-bit.txt` | FBE board — rebuild process: reuse an already-prepared workspace to recompile (full `make clean` rebuild or incremental kernel rebuild) and regenerate the single image. |
