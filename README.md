# x6100
Documentation for the Xiegu x6100 Radio

There really isn't enough documentation on custom firmware and getting a custom build running on the radio, so let's fix that.

# Useful Projects/Notes
https://github.com/AetherRadio/X6100Buildroot

https://github.com/AetherRadio/X6100Control

https://github.com/AetherRadio/X6100Study

https://github.com/Links2004/x6100-armbian

# uboot Header
```
U-Boot SPL 2022.10 (Jun 24 2023 - 22:33:58 -0400)                               
DRAM: 1024 MiB                                                                  
Trying to boot from MMC1                                                        
                                                                                
                                                                                
U-Boot 2022.10 (Jun 24 2023 - 22:33:58 -0400) Allwinner Technology              
                                                                                
CPU:   Allwinner A33 (SUN8I 1667)                                               
Model: XIEGU Tech X6100 HF+6m Transceiver                                       
DRAM:  1 GiB
```

# Default Partitions
## SD CARD
### uboot
### RootFS
### User FS (optional)
## MMC

# Default Firmware
Connect USB-C to USB-A cable provided by Xiegu to the DEV port on the radio. This interface provides the following interfaces:
* Serial - Serial Console
* Serial
* Audio
## Kernel Boot Log
## Login
Default login/password: root/123

# Building R1CBU with buildroot
## Clone the repository
```
git clone --recurse-submodules git@github.com:strijar/AetherX6100Buildroot.git
git switch R1CBU
git submodule update --init --recursive
```
## Build the project
Note: My distro has bash as /bin/bash not /usr/bin/bash, you may need to modify this script
```
./br_config.sh
cd build
make
```
## Write the SD Card
Note change sdz to your SD Card device
```
sudo dd if=build/images/sdcard.img of=/dev/sdz bs=1M
eject /dev/sdz
```

## Starting the radio
Sometimes it appears that the radio doesn't start properly. Perhaps the ST32 is running but the ARM is not? In any case, I was able to get things to boot properly by first booting the default firmware, then powering off, then booting from the SD Image. I did run into an issue where the DRAM was detected as 2GB causing the system to not boot. There is a patch in the TOADs discord about hardcoding to 1GB to fix this issue (Thanks DreamNik):
```
--- a/arch/arm/mach-sunxi/dram_sun8i_a33.c
--- b/arch/arm/mach-sunxi/dram_sun8i_a33.c
@@ -348,6 +348,10 @@
         return 0;
 
     auto_detect_dram_size(&para);
+    // Fix: force set DRAM size to 1024MiB, because auto-detection is unstable
+    para.rows      = 16;
+    para.page_size = 2048;
+    mctl_set_cr(&para);
 
     /* Enable master software clk */
     writel(readl(&mctl_com->swonr) | 0x3ffff, &mctl_com->swonr);
```

Note: The default build from the buildroot above is a starting point for a custom firmware. It does not include all of the applications to run the radio!


# Building Arbmian for the x6100

## Check out the Armbian build environment
```
git clone https://github.com/Links2004/x6100-armbian
cd x6100-armbian
git clone https://github.com/armbian/build --depth=1
```

## Collect uboot
Download the Xiegu firmware: https://radioddity.s3.amazonaws.com/Xiegu_X6100_Firmware_Upgrade_20221124.zip, unzip it and cut uboot from the image:

```
unzip Xiegu_X6100_Firmware_Upgrade_20221124.zip
dd if=Firmware/sdcard_20221124.img of=uboot_sdcard.bin bs=1024 skip=8 count=512 seek=0
```

## Collect the kernel, device tree and modules
The kpartx tool makes mounting images easy by creating loop based devices. You can also manually set up the loop devices if you don't have kpartx installed.
```
sudo kpartx -av sdcard_20221124.img
sudo mkdir /mnt/x6100
sudo mount /dev/mapper/loop0p1 /mnt/x6100
cp /mnt/x6100/zImage .
cp /mnt/x6100/sun8i-r16-x6100.dtb .
sudo umount /dev/mapper/loop0p1
sudo mount /dev/mapper/loop0p2 /mnt/x6100
cp -r /mnt/x6100/lib/modules .
sudo umount /dev/mapper/loop0p2
sudo kpartx -d sdcard_20221124.img
```

## Put the files into the userpatches directory
```
mkdir -p userpatches/overlay/extracted/
cp zImage userpatches/overlay/extracted/
cp sun8i-r16-x6100.dtb userpatches/overlay/extracted
cp -r modules userpatches/overlay/extracted
```

## Add userpatches to the armbian build
```
cp -r userpatches/ build/
cd build
./compile.sh docker BOARD=lime-a33 BSPFREEZE=yes BRANCH=current RELEASE=sid BUILD_MINIMAL=no BUILD_DESKTOP=yes KERNEL_ONLY=no KERNEL_CONFIGURE=no DESKTOP_ENVIRONMENT=xfce DESKTOP_ENVIRONMENT_CONFIG_NAME=config_base DESKTOP_APPGROUPS_SELECTED="3dsupport browsers" COMPRESS_OUTPUTIMAGE=sha,gpg,img
```
