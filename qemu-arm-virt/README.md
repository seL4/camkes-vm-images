<!--
     Copyright 2019, Data61, CSIRO (ABN 41 687 119 230)

     SPDX-License-Identifier: CC-BY-SA-4.0
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


* File: rootfs_crossvm.cpio.gz
* Derived from rootfs.cpio.gz
* Contains the following additional files:
  * `/usr/bin/consumes_event_wait`
  * `/usr/bin/dataport_read`
  * `/usr/bin/dataport_write`
  * `/usr/bin/emits_event_emit`
  * `/lib/modules/4.14.87/kernel/drivers/vmm/connection.ko`
  * `/etc/init.d/S90crossvm_module_init`
  * Files build from sources located: https://github.com/SEL4PROJ/camkes-vm-linux/tree/122562b460110fb7134d78203d84c2081f57cb49/camkes-linux-artifacts
  * Using build rules located: https://github.com/SEL4PROJ/camkes-arm-vm/blob/0aa7be72d1335e5a708128426951b65e36ad5f1c/apps/vm_cross_connector/CMakeLists.txt#L44
