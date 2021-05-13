<!--
     Copyright 2019, Data61, CSIRO (ABN 41 687 119 230)

     SPDX-License-Identifier: CC-BY-SA-4.0
-->

# TX1

## Compilation details:
### Linux image
* File: linux
* Git Remote: https://github.com/torvalds/linux.git
* Tag: v4.16 (Commit Hash: 0adb32858b0bddf4ada5f364a84ed60b196dbcda)
* Linux Config: Configuration used can be found in 'linux\_configs/config'
* Compiled with: aarch64-linux-gnu-gcc (gcc version 6.3.0)

### Buildroot Rootfs image
* File: rootfs.cpio.gz
* Version: 2019.08.1 (https://buildroot.org/downloads/buildroot-2019.08.1.tar.gz)
* Buildroot Config: Default configuration (no modifications)
* Compiled with: aarch64-linux-gnu-gcc (gcc version 7.4.0)
* Notes: Rootfs file is compiled into Linux image
