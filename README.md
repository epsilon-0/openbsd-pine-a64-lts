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
- Put the micro-sd card in your reader and connect it to the laptop
-- If this is the only sd card, it is found on `/dev/mmcblk0` (at least for my ubuntu machine)

## (2) Adding `sun50i-a64-pine64-lts.dtb`

## (3) Flashing `u-boot-sunxi-with-spl.bin`

## (4) Boot and install OpenBSD

## (5) Re-add `u-boot.bin` and `sun50i-a64-pine64-lts.dtb`

## (6) Re-flash `u-boot-sunxi-with-spl.bin`

## Enjoy

TODO: Finish up the write-up
