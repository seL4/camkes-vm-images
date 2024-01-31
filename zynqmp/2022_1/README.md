<!--
     Copyright 2024, DornerWorks

     SPDX-License-Identifier: CC-BY-SA-4.0
-->

# zynqmp

## Compilation details:
### Linux image
* File: linux
* Git Remote: https://github.com/Xilinx/linux-xlnx
* Tag: xilinx-v2022.1 (Commit Hash: 0b70857ca66da7d471f5c17d1af67a2af273a960)
* Linux Config: Configuration used can be found in 'linux\_configs/config'

### File system
* File: rootfs-minimal.cpio.gz
* Toolchain: Petalinux 2022.1
* Link: https://www.xilinx.com/downloadNav/embedded-design-tools/2022-1.html
* BSP: ZCU102
* Rootfs modified to add arping
