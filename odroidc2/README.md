<!--
     Copyright 2019, Data61, CSIRO (ABN 41 687 119 230)

     SPDX-License-Identifier: CC-BY-SA-4.0
-->

# odroidc2
## Compilation details:
### Linux image
* File: linux
* From: https://github.com/torvalds/linux
* Commit hash: 7566ec393f4161572ba6f11ad5171fd5d59b0fbd (v4.20-rc7)
* Config: `linux-config`
* Patch: `gic_disable_check.patch`

### Buildroot Rootfs image
* File: rootfs.cpio.gz
* From: git://git.buildroot.net/buildroot
* Commit hash: ca0547ffeaea77b1b59ddcf77a2f3713167f8a7e
* Config: `buildroot/buildroot-config`
