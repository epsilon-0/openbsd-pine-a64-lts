# Installing OpenBSD 6.5 on the Pine A64-LTS model

This is a short and VERY specific guide for installing OpenBSD 6.5 on the [Pine A64-LTS](https://www.pine64.org/?product=pine-a64-lts) SoC board. The official installation instructions are present [here](https://ftp.openbsd.org/pub/OpenBSD/6.5/arm64/INSTALL.arm64), but they are unforunately not enough for our current board. There are a lot of nuances in this installation, from the standard flashing and booting.

## A brief outline of the installation:
- Flashing miniroot onto the micro-sd card
- Adding the appropriate `*.dtb` file (the device tree file) to the micro-sd card and flashing.
- Booting and installing OpenBSD
- Readding the files and flashing the BOOT partition

## Requirements
I am going to assume that this is the first ever SoC computer that you have ever seen in life and have no freaking clue how to even connect to this device. I am doing this because this was my exact state a few weeks before and there were no comprehensive tutorials around. I am aiming this guide to be a good introduction to working with SoC boards, along with what components are needed and how to connect (via a serial console)
- [Pine A64-LTS](https://www.pine64.org/?product=pine-a64-lts) (also maybe get a [power supply](https://www.pine64.org/?product=sopine-baseboard-us-power-supply))
- [USB to TTL converter](https://www.amazon.com/gp/product/B008AGDTA4/) (cp2102)(needed for serial console access)
- micro-sd card (+ card reader, for being able to read the micro-sd card on your laptop/desktop)
- Access to another (unix/linux) computer for formatting/writing to the micro-sd card (I am going to install OpenBSD on the pine using an Ubuntu laptop, so the tty/mmcblk devices will be reflecting the linux nomenclature).

## (1) Flashing miniroot65.fs
- Download [`miniroot65.fs`](https://cdn.openbsd.org/pub/OpenBSD/6.5/arm64/miniroot65.fs) for the `arm64` machines (Pine A64 is one of these).
- Put the micro-sd card in your reader and connect it to the laptop. Make sure the the partitions are **unmounted**.
  - If this is the only mirco-sd card and in a dedicated card reader, it is found on `/dev/mmcblk0`
- Flash the miniroot65.fs:
    ```
    dd if=miniroot65.fs of=/dev/mmcblk0 bs=1M status=progress
    ```
- After flashing you will get a BOOT partition of 4MB in size and a `ufs` partition of 14MB in size
- Download https://ftp.openbsd.org/pub/OpenBSD/6.5/packages/amd64/u-boot-aarch64-2019.01p5.tgz and extract `.../share/u-boot/pine64-lts/u-boot-sunxi-with-spl.bin`
- Flash the file `u-boot-sunxi-with-spl.bin` with the correct batch size and seek:
    ```
    dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
    ```
  - The seek is to format the starting of the `ufs` partition so that the machine can boot correctly.

## (2) Adding `sun50i-a64-pine64-lts.dtb`
- Download https://ftp.openbsd.org/pub/OpenBSD/6.5/packages/amd64/dtb-4.20.tgz and extract `.../share/dtb/arm64/allwinner/sun50i-a64-pine64-lts.dtb`
- Mount the BOOT partition of the micro-sd card and copy the `*.dtb` file into a folder called `allwinner`, and make a copy of ALL FILES in the boot partition (we are going to need them in step 4)
    ```
    mkdir -p /tmp/tmpmnt
    mount /dev/mmcblk0p1 /tmp/tmpmnt
    mkdir -p /tmp/tmpmnt/allwinner
    cp sun50i-a64-pine64-lts.dtb /tmp/tmpmnt/allwinner
    cp -r /tmp/tmpmnt ~/
    umount /tmp/tmpmnt
    ```

## (3) Boot and install OpenBSD
- Insert the micro-sd card into the pine.
- OpenBSD does not support HDMI I/O for the Pine A64 LTS (or any other SoC for that matter) so you need to connect using a serial console (the usb to ttl conenctor)
- This is a [UART connector](http://linux-sunxi.org/File:Pine64_UART0.jpg) for the pine, connect it to the RIGHT side pins, as shown in the picture
  - THE TXD connects to the RXD pin and the RXD connects to the TXD pin [RXD - receiver, TXD - transmitter]
  - Connecting to the left hand side can mess up voltages
  - Information from https://chown.me/blog/playing-with-the-pine64.html

- Connect the usb to your machine and to access the console you can use two commands `cu` or `screen`
- I will be using `screen` as that comes default in most linux distributions
- The serial console attaches itself to `ttyUSB0` in most cases and can be connect via
    ```
    screen /dev/ttyUSB0 115200
    ```
  - Don't forget to hard-boot the machine after connecting the serial console so that you can begin the boot process

## (4) Re-add `u-boot.bin` and `sun50i-a64-pine64-lts.dtb` and flash `u-boot-sunxi-with-spl.bin`
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
