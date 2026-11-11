---
title: WhyFi IPQ5321 — Build Environment and CI/CD Design
tags: [whyfi, ipq5321, ci-cd, github-actions, hitl, product-repo, design]
sources: [references/FBE-build-steps-64-bit.txt, references/FBE-build-steps-32-bit.txt, references/FBE-rebuild-steps-64-bit.txt, https://docs.github.com/en/actions/tutorials/store-and-share-data]
updated: 2026-11-11
---

# WhyFi IPQ5321 — Build Environment and CI/CD Design

*Proposed design: one product repo, `whyfi-ipq5321`, turns the manual FBE procedure into pinned inputs and scripts, and a CI pipeline builds packages and installs them on a lab router in a HITL environment.*

> Sources: references/FBE-build-steps-64-bit.txt (feeds and config, lines 108–135; proprietary copies and quilt, lines 29–85); references/FBE-build-steps-32-bit.txt (32-bit target sed, lines 111–114); references/FBE-rebuild-steps-64-bit.txt (quilt and rebuild, lines 1–37); GitHub Docs "Store and share data with workflow artifacts" (artifact transport between jobs).

This page is a design proposal, not a record of a built system. The build steps it automates come from the references. The repo layout, the pipeline, and the selection rules are design decisions. No reference describes them.

## Goals

- Recreate the same QSDK build environment on any machine without manual steps.
- Enable, build, and deploy packages through a CI pipeline.
- Tie every research result to an exact firmware or package build.

The first use case is enabling the upstream `usteer` package and installing it on the lab router.

## Inputs that stay outside Git

Two inputs are provided by hand on the build machine and are never committed:

| Input | Location | Identity in the repo |
|---|---|---|
| Vendor proprietary drop | `QSDK_BASE_DIR` | `sources/drop.lock` (name, expected files, sha256) |
| Patch set (the quilt series applied by `quilt push -a`, 64-bit lines 83–85) | `QSDK_PATCHES_DIR` | `sources/patches.lock` (name, per-file sha256) |

`bootstrap.sh` refuses to run if the files on disk do not match the lock files.

## Repositories

| Repo | Status | Purpose |
|---|---|---|
| `whyfi-ipq5321` | Create now, private | The product repo. It holds the pins, the config overlay, the scripts, and the CI workflows for the firmware and its packages. |
| `qsdk-build-wiki` | Exists | This knowledge base. Pages explain why. The scripts become the procedure. |
| `whyfi-builder` | Later, only if needed | A container image for the host build environment. It starts as `env/` in the product repo. |

The design leaves out a packages-feed repo (no own packages yet), feed mirrors, and a fleet repo. Most feeds in the QSDK `feeds.conf` are `src-link` entries into the vendor tree, so the vendor drop already pins them.

## Product repo layout

```
whyfi-ipq5321/
├── VERSION
├── CHANGELOG.md
├── sources/
│   ├── drop.lock               # vendor drop identity
│   ├── patches.lock            # patch set identity
│   ├── manifest.xml            # pinned repo manifest (repo manifest -r)
│   └── feeds.conf              # feeds snapshot
├── config/
│   ├── overlay-64.config       # all deltas on the vendor seed (EZMESH, added packages)
│   ├── overlay-32.config       # 32-bit variant, for the U-Boot ELF
│   └── experiments/<exp>.config
├── env/
│   ├── host-packages.txt
│   ├── host-setup.sh           # includes make 4.1
│   └── Dockerfile              # optional
├── scripts/
│   ├── bootstrap.sh            # verify inputs, repo sync, copy proprietary, quilt push
│   ├── configure.sh            # vendor seed, target sed, overlay, defconfig, verify
│   ├── classify-config-delta.sh
│   ├── build.sh                # full build (32 or 64)
│   ├── rebuild.sh              # incremental or kernel-only
│   ├── build-package.sh        # compile requested packages, collect missing deps
│   ├── single-image.sh
│   ├── collect.sh              # artifacts, effective .config, diffconfig
│   └── deploy-ipks.sh          # runs on the HITL runner
└── .github/workflows/
    ├── build.yml               # [self-hosted, qsdk]
    ├── hitl.yml                # [self-hosted, hitl]
    └── release.yml             # tag: clean full build, archive
```

## Configuration model

The vendor file `qca/configs/qsdk/ipq_premium.config` is a short seed, and `make defconfig` expands it into the full `.config` (64-bit lines 112–135). The procedure already layers changes on the seed. It runs a target `sed` (64-bit line 113; 32-bit lines 112–113), appends the EZMESH symbols (64-bit lines 117–132), and then runs `make defconfig`.

The design keeps that model with two pieces and does not split the vendor seed:

1. The vendor seed, unmodified, so a new drop replaces it cleanly.
2. One overlay file per architecture, with every WhyFi delta.

`configure.sh` copies the seed, applies the target `sed`, appends the overlay, and runs `make defconfig`. It then fails if a requested overlay symbol lost its value. `make defconfig` drops a symbol silently when a dependency is missing. Each build archives the effective `.config` and its `diffconfig`.

## Pipeline

```
PR / push ─► configure ─► classify ─┬─ package path  ─► build-package ─► artifact ─► hitl: opkg install
                                    └─ firmware path ─► build 32/64 ─► single image ─► archive only
tag v*    ─► clean bootstrap + full build ─► archive
```

`classify-config-delta.sh` compares the new effective `.config` with the one that built the firmware on the router:

| Delta | Path | Result |
|---|---|---|
| Only new `=m` package symbols | Package path | Build the packages and install them on the router |
| Any `=y`, kernel, or `kmod-*` change | Firmware path | Build and archive the image. Nothing flashes boards. |

The package path is safe only when the build tree still matches the deployed firmware. `configure.sh` resets `.config` at the start of every job. A recorded input hash lets the package path refuse a tree that has drifted.

## Package selection

`bin/` keeps output across builds, so the pipeline never uploads `bin/**/*.ipk`. The 64-bit workspace on the build machine held 717 `.ipk` files on 2026-11-11. `build-package.sh` selects:

1. The packages the PR enabled, from the new `=m` symbols in the config delta.
2. A fresh build of each, with `make package/<pkg>/clean` and then `make package/<pkg>/compile`.
3. The dependency closure, minus the packages already in the firmware image manifest.
4. One version of each package. The script fails on ambiguity.

The script stamps `PKG_RELEASE` with the run ID, or the deploy uses `--force-reinstall`, so every run installs.

## Artifact transport to HITL

The two runners never connect to each other. GitHub artifact storage relays the files:

1. The `build` job on `[self-hosted, qsdk]` writes `dist/*.ipk` and uploads it with `actions/upload-artifact`.
2. The `hitl` job declares `needs: build`, so it starts only after a successful build.
3. The `hitl` job on `[self-hosted, hitl]` downloads the artifact with `actions/download-artifact`.
4. `deploy-ipks.sh` copies the files with `scp -O` and runs one `opkg install` call over SSH. Dropbear has no SFTP server, so `scp` needs the `-O` flag.

Both runners make only outbound HTTPS connections to GitHub. Router SSH credentials exist only on the HITL machine. Deploy and verification run in one job, so a deploy from another run cannot land between them.

The artifacts are stored on GitHub until they expire. That is acceptable for open-source packages such as `usteer`. Packages that contain proprietary QCA binaries need a local relay instead, such as a direct push from the build machine to the HITL machine or an HTTP feed on the build machine.

## Research traceability

- Tag the firmware or package build used in each experiment, and record the tag with the results.
- Stamp the version and input hash into the image, so each board reports what it runs.
- Keep experiment variants as overlays in `config/experiments/`. A with-and-without comparison is then two tagged builds that differ by one documented line.

## Related pages

- [FBE Source Preparation](fbe-source-prep.md) is the procedure that `bootstrap.sh` automates.
- [FBE 64-bit Build](fbe-build-64-bit.md) and [FBE 32-bit Build](fbe-build-32-bit.md) are the procedures that `configure.sh` and `build.sh` automate.
- [FBE Firmware Rebuild](fbe-rebuild.md) is the procedure that `rebuild.sh` automates.
