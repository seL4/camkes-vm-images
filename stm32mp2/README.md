<!--
     Copyright 2026, STMicroelectronics

     SPDX-License-Identifier: CC-BY-SA-4.0
-->

# STM32MP2
To be used with camkes-vm-examples, the Linux and Buildroot binaries must
be generated using the manifest below and installed in this directory.

* File: linux
* From: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
* Version: v6.18-rc7
* Config: `linux-config`

## Compilation details:
* File: rootfs.cpio.gz
* Version: 2025.08.1 (https://buildroot.org/downloads/buildroot-2025.08.1.tar.gz)
* Config: `buildroot-config`

* File: rootfs_crossvm.cpio.gz
* Derived from rootfs.cpio.gz
* Contains the following additional files:
  * `/usr/bin/consumes_event_wait`
  * `/usr/bin/dataport_read`
  * `/usr/bin/dataport_write`
  * `/usr/bin/emits_event_emit`
  * `/lib/modules/6.6.101/kernel/drivers/vmm/uio.ko`
  * `/lib/modules/6.6.101/kernel/drivers/vmm/connection.ko`
  * `/etc/init.d/S90crossvm_module_init`

