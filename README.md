# Installing OpenBSD (-current) on the Pine A64-LTS model

This is a short guide for installing the OpenBSD latest snapshot on the [Pine A64-LTS](https://pine64.com/product/pine-a64-lts/?v=0446c16e2e66) SoC board.<br/>
The official installation instructions are present [here](https://ftp.openbsd.org/pub/OpenBSD/snapshots/arm64/INSTALL.arm64).

<table>
<tr>
<td width="27%" style="border-style:solid; border-radius:10px;">

## Contents

0. [Requirements](#requirements)
1. [Flash `minirootXX.img`](#flash)
2. [Add `*.dtb` file](#dtb)
3. [Boot and install OpenBSD](#boot)
4. [Readd and reflash BOOT partition](#reflash)

</td>
</tr>
</table>

## Requirements <a name="requirements"></a>
This guide assumes that this is the first ever SoC computer that the reader has ever seen in life and has no clue how to even connect to this device.<br/>
This guide is designed for a basic introduction to working with SoC boards, along with the components needed for connecting (via a serial console):
- [Pine A64-LTS](https://pine64.com/product/pine-a64-lts/?v=0446c16e2e66) (along with a [power supply](https://pine64.com/?product=pine-a64-usa-power-supply&v=0446c16e2e66))
- [USB to TTL converter](https://store.pine64.org/?product=pinebook-serial-console) (cp2102)(needed for serial console access, also available on [amazon](https://www.amazon.com/gp/product/B008AGDTA4/))
- micro-sd card (+ card reader, to read the micro-sd card from laptop/desktop)
- Access to another (unix/linux) computer for writing to the micro-sd card (this guide uses a linux computer to install OpenBSD on the pine, so the tty/mmcblk devices will be reflecting the linux nomenclature).

## Flash `minirootXX.img` <a name="flash"></a>
- Download [`minirootXX.img`](https://ftp.openbsd.org/pub/OpenBSD/snapshots/arm64/) for the `arm64` machines (Pine A64 is an armv8/aarch64 machine).
- Put the micro-sd card in the card reader and connect it to the laptop. Make sure the the partitions are **unmounted**.
  - If this is the only mirco-sd card and in a dedicated card reader, it will most likely be found on `/dev/mmcblk0`
- Flash the `minirootXX.img`:
    ```
    dd if=minirootXX.img of=/dev/mmcblk0 bs=1M status=progress
    ```
- After flashing there will be BOOT partition of size 4MB i and a `ufs` partition of size 14MB.
- Download `u-boot-aarch64-YYYY.MMpX.tgz` from https://ftp.openbsd.org/pub/OpenBSD/snapshots/packages/amd64/ and extract the file `.../share/u-boot/pine64-lts/u-boot-sunxi-with-spl.bin`
- Flash the file `u-boot-sunxi-with-spl.bin` with the correct batch size and seek:
    ```
    dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
    ```
  - The seek is to format the starting of the disk so that the machine can boot correctly.
  
NOTE: The download links are for `amd64`, even though the machine is `arm64`, as the package is actually an `arm64` package. The `amd64` links are used as they are updated and compiled more frequently.

## Add `sun50i-a64-pine64-lts.dtb` <a name="dtb"></a>
- Download the file `dtb-X.YYpZ.tgz` from https://ftp.openbsd.org/pub/OpenBSD/snapshots/packages/amd64/ and extract the file `.../share/dtb/arm64/allwinner/sun50i-a64-pine64-lts.dtb`
- Mount the BOOT partition of the micro-sd card and copy the `*.dtb` file into a folder called `allwinner`, and make a copy of **ALL FILES** in the boot partition (needed again in step 4, so this creates a backup):
    ```
    mkdir -p /tmp/tmpmnt
    mount /dev/mmcblk0p1 /tmp/tmpmnt
    mkdir -p /tmp/tmpmnt/allwinner
    cp sun50i-a64-pine64-lts.dtb /tmp/tmpmnt/allwinner
    cp -r /tmp/tmpmnt ~/
    umount /tmp/tmpmnt
    ```

## Boot and install OpenBSD <a name="boot"></a>
- Insert the micro-sd card into the pine.
- OpenBSD does not support HDMI I/O for the Pine A64 LTS so a serial console (the usb to ttl conenctor) is needed to connect to the board.
- This is the [UART connector](http://linux-sunxi.org/File:Pine64_UART0.jpg) for the pine, connect it to the RIGHT side pins, as shown in the picture (connecting to the left side can mess up voltages).
  - THE TXD pin of the connector connects to the RXD pin of the board and the RXD connects to the TXD pin [RXD - receiver, TXD - transmitter].
  - Remove the RXD pin of the board before booting. This means no data can be sent while booting. Connecting the wire during the boot phase can mess up voltages. 
  - Reconnect the RXD pin of the board after booting has finished.

- Connect the usb to your machine and to access the console you can use two commands `cu` or `screen`
- I will be using `screen` as that comes default in most linux distributions
- The serial console attaches itself to `ttyUSB0` in most cases and can be connected via
    ```
    screen /dev/ttyUSB0 115200
    ```
  - Don't forget to hard-boot the machine after connecting the serial console so that you can begin the boot process

## Re-add `u-boot.bin` and `sun50i-a64-pine64-lts.dtb` and re-flash `u-boot-sunxi-with-spl.bin` <a name="reflash"></a>
- Now that you have finished the installation on `sd0`, if you try to reboot the machine you will end up with the following error
    ```
    DRAM: 0 MiB
    ### ERROR ### Please RESET the board ###
    ```
- The reason is that the installation removes the `u-boot.bin` and the `sun50i-a64-pine64-lts.dtb` files from the BOOT partition
- Remove the micro-sd card from the pine and connect it to your laptop (you can safely remove the card when the above error is showing)
- Now we need to copy all the files from step 2 back into the boot directory
    ```
    mkdir -p tmp/tmpmnt
    mount /dev/mmcblk0p1 /tmp/tmpmnt
    cp -r ~/tmpmnt/* /tmp/tmpmnt
    umount /tmp/tmpmnt
    dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
    ```

## Enjoy
