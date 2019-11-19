<!--
     Copyright 2019, Data61
     Commonwealth Scientific and Industrial Research Organisation (CSIRO)
     ABN 41 687 119 230.

     This software may be distributed and modified according to the terms of
     the BSD 2-Clause license. Note that NO WARRANTY is provided.
     See "LICENSE_BSD2.txt" for details.

     @TAG(DATA61_BSD)
-->
# qemu-arm-virt

## Compilation details:
### Linux image
* File: linux
* Git Remote: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
* Tag: v4.9.189 (Commit Hash: 4bd718dba6581ebd392539ad659642552fb5826c)
* Linux Config: Configuration used can be found in 'linux\_configs/config'
* Sourced from: Debian Stretch (9.11) Netboot Installation

### Buildroot Rootfs image
* File: rootfs.cpio.gz
* Version: 2019.08.1 (https://buildroot.org/downloads/buildroot-2019.08.1.tar.gz)
* Buildroot Config: Default configuration (no modifications)
* Compiled with: aarch64-linux-gnu-gcc (gcc version 7.4.0)
