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
