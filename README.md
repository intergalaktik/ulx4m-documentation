<!--
SPDX-FileCopyrightText: 2016 2016 Goran Mahovlic, <goran.mahovlic@gmail.com> et al.

SPDX-License-Identifier: CERN-OHL-S-2.0
-->

# ULX4M series PCBs

[ULX4M] = University digital Logic Learning Xtensible board release 4 Moduler successor of [ULX3S](http://radiona.org/ulx3s) = [University digital Logic](https://www.fer.unizg.hr/en/course/diglog) Learning
Xtensible board release 3 with SDRAM,
Successor of [ULX2S](http://github.com/emard/ulx2s).

## ULX4M-LS (Lattice + SDRAM version)

## ULX4M-LD (Lattice + DDR3 version)

![LD short desc](/pic/ulx4m-ld-short-desc.png)

![Waveshare short desc](/pic/Waveshare-ulx4m-ld-v2-explain.png)

![Rawspberry IO](/pic/ULX4M-LD-V2-raspberry_IO.jpg)

### ULX4M-LD v2 schematics

https://github.com/intergalaktik/ulx4m/blob/ulx4m-ld/doc/schematics.pdf

### ULX4M-LD v2 LPF

https://github.com/intergalaktik/ulx4m/blob/ulx4m-ls/doc/constraints/prototype/ulx4m-ld_v002.lpf

### ULX4M repositories

https://github.com/lawrie/ulx4m_examples

https://github.com/lawrie/ulx4m_amaranth_examples

https://github.com/emard/ulx3s-misc

https://github.com/emard/had2019-playground/tree/master/projects/bootloader

https://github.com/goran-mahovlic/mipi-csi-2

https://github.com/hdl4fpga/hdl4fpga

### ULX4M-LS v1 bitstreams

https://github.com/lawrie/ulx4m_bitstreams

### DFU bootloader for ULX4M

USB DFU at US1 port for FPGA flashing.
DFU is very fast.

Source is based on [HAD2019 badge bootloader](https://github.com/smunaut/had2019-playground)
modified to [ULX3S bootloader](https://github.com/emard/had2019-playground)

Bootloader by default skips to user's bitstream.
If FLASH doesn't contain valid user's bitstream, LEDs will blink
because bootloader is constantly restaring. This is normal.

To enter bootloader, hold BTN3 or set DIP SW1=ON and plug US1
In bootloader mode, LEDs 0-2 should be ON, other LEDs 3-7 OFF:

|   D7   |   D6   |   D5   |   D4   |   D3   |    D2   |    D1   |    D0   |
|--------|--------|--------|--------|--------|---------|---------|---------|
|&#x2b1b;|&#x2b1b;|&#x2b1b;|&#x2b1b;|&#x2b1b;|&#x1f7e9;|&#x1f7e7;|&#x1f7e5;|

    lsusb
    Bus 001 Device 117: ID 1d50:614b OpenMoko, Inc. ULX4M-LD(DFU)

On linux, it's practical to add a udev rule which allows normal users
members of "dialout" group to also run dfu-util,
otherwise it should be run as root:

    # file: /etc/udev/rules.d/80-fpga-dfu.rules
    # this is for DFU 1d50:614b libusb access
    ATTRS{idVendor}=="1d50", ATTRS{idProduct}=="614b", \
    GROUP="dialout", MODE="666"

DFU is very fast, writes bitstream in few seconds.

# Usage

To upload user bitstream, hold BTN1 or turn on DIP SW1 and plug
USB. Here are some commandline examples:

    dfu-util -a 0 -D blink.bit
    openFPGALoader --dfu --vid 0x1d50 --pid 0x614b --altsetting 0 blink.bit

To exit bootloader and execute user's bistream, use DFU command:

    dfu-util -a 0 -e

To upgrade bootloader, hold BTN1 and BTN2 and plug USB:

# Write Protecting Bootloader

To prevent accidental overwrite,
flash chips have options to
protect range of address space.

    16MB ISSI and Winbond FLASH chips are supported

Different FLASH chips have standard/compatible
commands for reading and writing but different
commands for write protection. On ULX3S usual
16MB chips are ISSI IS25LP128 or Winbond W25Q128,
both are supported by esp32ecp5 and work.

    4MB ISSI FLASH chip is not supported

ISSI IS25LP032 4MB is actually "supported",
it has same datasheet and commands as IS25LP128
but onboard chip probably has some bug, it can't
protect address range where bootloader is.

    Protected range: first 2MB

For this bootloader, first 2MB (address
0 - 0x1FFFFF) should be write protected.
It contains bootloader bitstream and some
data structure that can jump to user's
bitstream which starts from 0x200000.

    From what it protects

FLASH protection applied here is based on non-OTP
bits so it's reversible and protects from accidental
overwrite with "fujprog", "esp32ecp5" and
DFU bootloader itself. "fujprog" will early
segfault, "esp32ecp5" will stop after first
block verify, bootloader will upload data without
error but content will not be written.

Current version of "openFPGALoader" will silently
remove non-OTP write protection, overwrite bootloader and
leave FLASH chip unprotected. 

ISSI FLASH should be always safe and reversible.
Winbond FLASH protection can set non-reversible
OTP status register lock bit and in that case,
there is no known way to remove protection.