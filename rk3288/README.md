# Running seL4 VM on the NTablet

To run seL4 in HYP mode, the system must be left in HYP mode(or monitor mode)
prior running seL4. However, the upstream bootloader sets the processor to SVC
mode, which makes it impossible to run seL4 in HYP mode. The solution is to use
ATF to replace the close-sourced OPTEE-OS.

## Building ATF

```bash
git clone https://github.com/szhuang/arm-trusted-firmware.git
cd arm-trusted-firware
git checkout rk3288
make ARCH=aarch32 CROSS_COMPILE=arm-linux-gnueabihf- PLAT=rk3288 AARCH32_SP=sp_min bl32
ls build/rk3288/release/bl32.elf
```

## Building U-boot with ATF support

```bash
git clone https://github.com/szhuang/u-boot.git
cd u-boot
cp $(ATF)/build/rk3288/release/bl32.elf bl31.elf
make rk3288-ntablet-870a_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf- -j4 idbloader.img
make CROSS_COMPILE=arm-linux-gnueabihf- -j4 all

# Open u-boot.its, remove the atf@2 node and atf@3 node
./tools/mkimage -f u-boot.its -E u-boot.itb
```

## Building CAmkES VM

```bash
mkdir camkes-arm-vm
cd camkes-arm-vm
repo init -u https://github.com/seL4/camkes-arm-vm-manifest.git -b rk3288
repo sync
mkdir build
cd build
../init-build.sh -DCAMKES_VM_APP=vm_minimal -DPLATFORM=rk3288 -DCROSS_COMPILER_PREFIX=arm-linux-gnueabihf-
ninja
ls images/capdl-loader-image-arm-rk3288
```

## Flashing NTablet

```bash
# Get the tools
git clone https://github.com/yeacreate-opensources/rkbin.git

# Get rootfs
wget http://download.yeacreate.com/Ntablet/ubuntu18.tar.gz
tar -zxf ubuntu18.tar.gz

# Get NTablet ready for flashing
# Reboot NTablet to U-boot, run "rockusb 0 mmc 0"

# Flash rootfs
./rkbin/tools/upgrade_tool UF ubuntu18.img

# Flash bootloader, U-boot and VM image
./rkdeveloptool wl 0x40 ./u-boot/idbloader.img
./rkdeveloptool wl 0x4000 ./u-boot/u-boot.itb
./rkdeveloptool wl 0x8000 ./camkes-arm-vm/build/images/capdl-loader-image-arm-rk3288

# Running VM
# Reset the NTablet, execute the following in U-boot terminal
mmc read 0x8000000 0x8000 0x20000
go 0x8000000
```

