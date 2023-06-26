# x6100
Documentation for the Xiegu x6100 Radio

There really isn't enough documentation on custom firmware and getting a custom build running on the radio, so let's fix that.

# Useful Projects/Notes
https://github.com/AetherRadio/X6100Buildroot

https://github.com/AetherRadio/X6100Control

https://github.com/AetherRadio/X6100Study


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

# Default Firmware
Connect USB-C to USB-A cable provided by Xiegu to the DEV port on the radio. This interface provides the following interfaces:
* Serial - Serial Console
* Serial
* Audio
## Kernel Boot Log
## Login
Default login/password: root/123
