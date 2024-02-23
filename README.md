![Zyxel Multy M1](https://raw.githubusercontent.com/ulp1an/ZyXEL-WSM20/main/image/multy.png)
# Installation

## via Web Interface
The device is cloud-managed, but there is a hidden local firmware upgrade page in the OEM web interface. You should be able to open a new tab and access the firmware upgrade page without registering the device. If this doesn't work, you can always register in the cloud.

The system has a dual firmware design, there is no way to tell which firmware is currently booted. Therefore, an  `-initramfs`  version is flashed first.

1.  Log into the OEM web  GUI  (this step is not required if you are patient and click reload a few times on the upgrade page mentioned below)
    
2.  Access the hidden upgrade page by navigating to  [http://192.168.212.1/gui/#/main/debug/firmwareupgrade](http://192.168.212.1/gui/#/main/debug/firmwareupgrade "http://192.168.212.1/gui/#/main/debug/firmwareupgrade")
    
3.  Upload the  `-initramfs-kernel.bin`  file and flash it
    
4.  Wait for the router to restart and then log into OpenWrt  GUI  (LuCI) at 192.168.1.1. (You may need to momentarily unplug ethernet cable from computer to force it to pick up new  IP  address)
    
5.  Click on the  `Go to firmware upgrade...`  button.
    
6.  Scroll down the page and in the section 'Flash new firmware image', click on  `Flash Image`  button.
    
7.  Select the latest stable OpenWrt Upgrade  `squashfs-sysupgrade.bin`  image and click  `Upload`  button.
    
8.  **Untick**  the  `Keep Setting...`  checkbox.
    
9.  Press  **Continue**  to flash the new sysupgrade image.
    

While the firmware image is being flashed, the LED starts flashing red, at about 1/sec, for about 10 sec. Afterwards the LED will light up steadily in green. Later it starts flashing quickly white, then it flashes slower, still white. OpenWrt is finally installed when the white LED is steady.


##  via UART (serial console)

The UART method is more difficult, as the boot loader does not have a timeout set. A semi-working stock firmware is required to configure it:

1.  Attach UART
    
2.  Boot the stock firmware until the message about failsafe mode appears
    
3.  Enter failsafe mode by pressing “`f`” and “Enter”
    
4.  Type “`mount_root`”
    
5.  Run “`fw_setenv bootmenu_delay 3`”
    
6.  Reboot, U-Boot now presents a menu
    
7.  The `-initramfs-kernel.bin` image can be flashed using the menu
    
8.  Run the regular sysupgrade for a permanent installation
    

Changing the partition to boot is a bit cumbersome in U-Boot, as there is no menu to select it. It can only be checked using mstc_bootnum. To change it, issue the following commands in U-Boot:

   `nand read 1800000 53c0000 800`
   
   `mw.b 1800004 1 1`
   
   `nand erase 53c0000 800`
   
   `nand write 1800000 53c0000 800`

This selects FW1. Replace  `mw.b 1800004 1 1`  by  `mw.b 1800004 2 1`  to change to the second slot.

# Restoring stock firmware

It is possible to flash back to stock, but a OEM firmware upgrade is required. ZyXEL does not provide the link on its website (A link can be acquired from the OEM web  GUI  by analyzing the transferred JSON objects).

-   Download original firmware from this link:  [V1.00(ABZF.4)C0.bin](https://github.com/ulp1an/ZyXEL-WSM20/raw/main/firmware/V1.00(ABZF.4)C0.bin "https://github.com/ulp1an/ZyXEL-WSM20/raw/main/firmware/V1.00(ABZF.4)C0.bin")
    
-   Rename the file to  `zyxel.bin`
    
-   Use SCP (eg. WinSCP for Windows users) and transfer the  `zyxel.bin`  file to the  `/tmp`  folder of the OpenWrt WSM20 router.
    
-   SSH  into the router (eg. PuTTY for Windows users) and it is then a matter of writing the firmware to Kernel2 and setting the boot partition to FW2:
    
`cd /tmp`

`mtd write zyxel.bin Kernel2`

`echo -ne "\x02" | dd of=/dev/mtdblock7 count=1 bs=1 seek=4 conv=notrunc`

If the OpenWrt WSM20 router is connected to the internet, you can alternatively use  `wget`  to download the OEM firmware straight to the router as described in this post by  [Owrt forum post](https://forum.openwrt.org/t/openwrt-for-zyxel-wsm20-multy-m1-development-discussion/154379/766 "https://forum.openwrt.org/t/openwrt-for-zyxel-wsm20-multy-m1-development-discussion/154379/766")

    cd /tmp
    wget "https://d3jal3boi407dg.cloudfront.net/mycloud/wsm20/latest_firmware_info/s3_file/1652350021659/V1.00(ABZF.4)C0.bin"
    mv "V1.00(ABZF.4)C0.bin" "zyxel.bin"
    [ "$(md5sum zyxel.bin | awk '{print $1}')" == "b601f1ee260460e4107345003f82b7f4" ] && echo "Firmware check passed!" || echo "Firmware is corrupted, download it again!"
    mtd write zyxel.bin Kernel2
    echo -ne "\x02" | dd of=/dev/mtdblock7 count=1 bs=1 seek=4 conv=notrunc
    reboot
## Stock firmware
[V1.00(ABZF.4)C0.bin](https://github.com/ulp1an/ZyXEL-WSM20/raw/main/firmware/V1.00%28ABZF.4%29C0.bin)

`Name: V1.00(ABZF.4)C0.bin`

`Size: 22020100 bayt (21 MiB)`

`CRC32: 8EB863CE`

`CRC64: 4EE49AF7A7AE94E3`

`SHA256: 721d1511a1bf29045c14446d0ab0feac05c3d39b3ec4444b8150856889247bf6`

`SHA1: 9d7fc4efef1a3b4c8541bd3910f89a69f0622c74`

`BLAKE2sp: 920882aeccb32c52c83b54ddf8c45c331c9d0a14205d8a9309b5eae5cfcf6b84`
