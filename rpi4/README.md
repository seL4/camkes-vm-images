<!--
     Copyright 2026, Shankaraditya N S

     SPDX-License-Identifier: CC-BY-SA-4.0
-->

# rpi4

Guest images for running a Linux VM on the Raspberry Pi 4 Model B
(`KernelPlatform=bcm2711`, `KernelARMPlatform=rpi4`).

Both a mainline and a Raspberry Pi vendor kernel have been tested and boot to
an interactive shell. The committed `linux` is the mainline build; the vendor
tree is documented as an alternative but is not required.

## Compilation details:

### Linux image
* File: linux
* Git Remote: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
* Tag: v5.10.234
* Tarball: https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.234.tar.xz
* Linux Config: arm64 `defconfig`, unmodified. A copy is in
  'linux\_configs/config'
* Reported version: `5.10.234`
* Compiled with: aarch64-linux-gnu-gcc (Ubuntu 13.3.0-6ubuntu2~24.04.1) 13.3.0

```
curl -O https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.234.tar.xz
tar xf linux-5.10.234.tar.xz
cd linux-5.10.234
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) Image
cp arch/arm64/boot/Image <camkes-vm-images>/rpi4/linux
```

#### Alternative: Raspberry Pi vendor tree
Also tested, and reported as `5.10.110-v8+`:

* Git Remote: https://github.com/raspberrypi/linux.git
* Branch: rpi-5.10.y
* Commit: 8e1110a580887f4b82303b9354c25d7e2ff5860e
* Linux Config: `bcm2711_defconfig`, unmodified

```
git clone --depth 1 --branch rpi-5.10.y https://github.com/raspberrypi/linux.git
cd linux
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) Image
```

The vendor tree is listed because it is what the two other known seL4 RPi4B
guest setups use (tiiuae/tii\_sel4\_build and au-ts/libvmm). Mainline works
equally well here and needs no fork.

### Buildroot Rootfs image
* File: rootfs.cpio.gz
* Version: 2024.02.9 (https://buildroot.org/downloads/buildroot-2024.02.9.tar.gz)
* Buildroot Config: `qemu_aarch64_virt_defconfig` with `BR2_LINUX_KERNEL`
  disabled and `BR2_TARGET_ROOTFS_CPIO` / `BR2_TARGET_ROOTFS_CPIO_GZIP`
  enabled. A copy of the resulting .config is in 'buildroot\_config'
* Compiled with: the buildroot-internal toolchain

```
curl -O https://buildroot.org/downloads/buildroot-2024.02.9.tar.gz
tar xf buildroot-2024.02.9.tar.gz
cd buildroot-2024.02.9
make qemu_aarch64_virt_defconfig
./utils/config --disable BR2_LINUX_KERNEL
./utils/config --enable BR2_TARGET_ROOTFS_CPIO
./utils/config --enable BR2_TARGET_ROOTFS_CPIO_GZIP
make olddefconfig
make -j$(nproc)
cp output/images/rootfs.cpio.gz <camkes-vm-images>/rpi4/rootfs.cpio.gz
```

A 2024-era buildroot is used deliberately. Its `/init` guards the console
redirect in a subshell:

```
if (exec 0</dev/console) 2>/dev/null; then
    exec 0</dev/console
    ...
fi
```

Older images (including the 2019 `qemu-arm-virt/rootfs.cpio.gz` in this
repository) have an unguarded redirect and a `/dev/console` that is a regular
file rather than a character device. On this platform that causes
`console_on_rootfs()` to succeed on a 0-byte file, so init's stdin/stdout point
at nothing: writes vanish, reads return EOF, and init exits immediately with
`Kernel panic - not syncing: Attempted to kill init!`.

If the rootfs is repacked, it must be unpacked and repacked **as root**, or
`/dev/console` is silently converted to a regular file:

```
sudo sh -c 'zcat rootfs.cpio.gz | cpio -idm'
# ... modify ...
sudo sh -c 'find . | cpio -o -H newc | gzip -9 > rootfs.cpio.gz'
zcat rootfs.cpio.gz | cpio -itv | grep dev/console   # must show 'crw', not '-rw'
```

### Guest device tree
* File: dts/rpi4-guest.dts (compiled to dts/rpi4-guest.dtb), in the
  application directory `vm_minimal/rpi4/dts/`

The VMM generates the guest's DTB (`generate_dtb`) from a base DTB supplied as
`dtb_base_name`. That base is the kernel's own generated `kernel.dts` for
bcm2711, with three changes. Each was verified as necessary by removing it and
observing the resulting failure.

Note the bootstrap order: `kernel.dts` is produced by configuring a build, so
configure once before creating the DTS that the build then consumes.

```
# 1. configure once to generate the kernel's device tree
mkdir build && cd build
../init-build.sh -DCAMKES_VM_APP=vm_minimal -DPLATFORM=bcm2711 -DAARCH64=1
cp kernel/kernel.dts ../rpi4-guest.dts
cd ..

# 2. apply the three changes below, then place the .dts in the app
cp rpi4-guest.dts <app>/rpi4/dts/

# 3. reconfigure and build -- the build compiles the .dts with dtc
cd build && ninja
```

**1. Fixed UART clock.** Add a root-level node, placed before the first child
node since DTS requires properties to precede subnodes, using a phandle not
already in use:

```
	uart_fixed_clk {
		compatible = "fixed-clock";
		#clock-cells = <0x00>;
		clock-frequency = <0x2dc6c00>;	/* 48 MHz */
		clock-output-names = "uart_fixed";
		phandle = <0x28>;
	};
```

and point the PL011 at it, replacing `clocks = <0x06 0x13 0x06 0x14>`:

```
			clocks = <0x28 0x28>;
```

The guest is not given the clock controller (cprman), so without this the
PL011 driver reads a rate of 0 and `uart_get_baud_rate()` raises a WARN,
falling back to 9600 baud.

**2. Remove the `bluetooth` child node** from `serial@7e201000`. Its
`shutdown-gpios` phandle does not survive DTB generation, and the dangling
reference prevents the node from probing
(`OF: /soc/serial@7e201000/bluetooth: could not find phandle`).

**3. Mux the PL011 to GPIO 14/15.** Give the `uart0_gpio14` pin group a
phandle:

```
			uart0_gpio14 {
				brcm,pins = <0x0e 0x0f>;
				brcm,function = <0x04>;
				phandle = <0x29>;
			};
```

and point the PL011's `pinctrl-0` at it, replacing `<0x07 0x08>`:

```
			pinctrl-0 = <0x29>;
```

The stock value selects GPIO 30/31 and 32/33, which are not brought out to the
40-pin header. This is required *in addition to* `dtoverlay=disable-bt`;
neither alone is sufficient.

## Board setup

### config.txt
On the SD card's boot partition:

```
arm_64bit=1
enable_uart=1
uart_2ndstage=1
kernel=u-boot.bin
dtoverlay=disable-bt
```

`dtoverlay=disable-bt` moves the PL011 from GPIO 32/33 (Bluetooth) to
GPIO 14/15 on the 40-pin header. The firmware applies this before seL4 boots.

### Serial console wiring
* Pin 6 (GND) -> adapter GND
* Pin 8 (GPIO14, TX) -> adapter RXD
* Pin 10 (GPIO15, RX) -> adapter TXD
* 115200 8N1

## Building and running

```
../init-build.sh -DCAMKES_VM_APP=vm_minimal -DPLATFORM=bcm2711 -DAARCH64=1
ninja
```

The output is `images/capdl-loader-image-arm-bcm2711`, which must be a raw
binary rather than an ELF:

```
head -c4 images/capdl-loader-image-arm-bcm2711 | xxd   # expect df4f 03d5, not 7f45 4c46
```

If it is an ELF, the `seL4_tools` change noted below is missing.

The board is netbooted from U-Boot over TFTP:

```
sudo cp images/capdl-loader-image-arm-bcm2711 /srv/tftp/
```

`boot.scr` on the SD card, built from this `boot.cmd`:

```
setenv ipaddr 10.42.0.50
setenv serverip 10.42.0.1
setenv netmask 255.255.255.0
sleep 6
tftpboot 0x10000000 capdl-loader-image-arm-bcm2711
go 0x10000000
```

```
mkimage -A arm64 -T script -C none -n camkesvm -d boot.cmd boot.scr
```

The `sleep` is needed because gigabit auto-negotiation is slower than U-Boot
reaching its TFTP request after a power cycle.

Expected output ends with:

```
Welcome to Buildroot
buildroot login:
```

Log in as `root` (no password).

## Related changes required elsewhere

Running this app on bcm2711 also requires:

* `seL4_projects_libs`: a `GIC_PADDR` branch for BCM2711 in
  `libsel4vm/src/arch/arm/vgic/gicv2.h` (0xff840000, the ARM view of the
  GIC-400 — the rpi4 DTS gives the VideoCore view), and a
  `libsel4vmmplatsupport/plat_include/bcm2711` tree.
* `camkes-vm`: `components/VM_Arm/plat_include/bcm2711/plat/vmlinux.h`.
* `seL4_tools`: `ApplyData61ElfLoaderSettings()` in
  `cmake-tool/helpers/application_settings.cmake` is called from
  `camkes-vm-examples/settings.cmake` with `${KernelARMPlatform}`, which the
  kernel sets to `rpi4` for `KernelPlatform=bcm2711`. `binary_list` contains
  `bcm2711` but not `rpi4`, so the board silently gets `ElfloaderImage=elf` and
  U-Boot's `go` aborts on the ELF header. The same mismatch affects rpi3
  (bcm2837), rpi5 (bcm2712) and rock3b (rk3568).

## Known limitations

* The guest is given the GPIO controller (`/soc/gpio@7e200000`) as a
  passthrough device; without it the guest stalls early in boot. This lets the
  guest reprogram pins the host relies on, and is not appropriate for a
  production configuration. Emulating GPIO in the VMM, with a per-pin access
  policy, would be the correct approach.
* The guest has no network interface. Buildroot's startup reports
  `Waiting for interface eth0 to appear... timeout!` and continues; this is
  expected. Enabling `VmPCISupport` and `VmVirtioNet` was tried and produced no
  PCI bus in the guest, so the `vpci.h` values for this platform remain
  unverified.
* Single VM, single vCPU.
