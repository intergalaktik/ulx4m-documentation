<!--
SPDX-FileCopyrightText: 2016 2016 Goran Mahovlic, <goran.mahovlic@gmail.com> et al.

SPDX-License-Identifier: CERN-OHL-S-2.0
-->

# ULX4M-LS (Lattice + SDRAM)

![ulx4m-ls](https://github.com/intergalaktik/ulx4m-documentation/blob/main/pic/ulx4m-ls.png)

## Introduction

ULX4M-LS (Lattice + SDRAM) is a system on a module (SoM) board that can be used on CM4 compatible base board or as a standalone board.

## ULX4M-LS (Lattice + SDRAM)

FPGA: Lattice ECP5 [LFE5U-12F-6BG381C](http://www.latticesemi.com/~/media/LatticeSemi/Documents/DataSheets/ECP5/FPGA-DS-02012.pdf?document_id=50461) (12/25/45/85K LUT)

### Main parts

LFE5U-12F-6BG381C

IS42S16160G-7BL 32 MB SDRAM 

W25Q128JVSIM NOR Flash spiFlash, 3V, 128M-bit, 4Kb Uniform Sector

Ethernet - LAN8720A

### Programming options

External JTAG programming connector

JTAG connected to GPIO (programming with FTDI or ESP32)

USB (bootloader)

### Periferals/ 

- [x] 2 lane CSI camera port  CAM0 and CAM1

- [x] 2 lane DSI display port DISP0 (fake differential)

- [x] DISP0 pins connected to DISP1

- [x] * SerDes pair (TX/RX) connected to 2.0 header (radio experiments)

- [x] True differential GPDI video output

- [x] Fake differential GPDI video output

- [x] 4 bit SD card connection

- [x] * SerDes connected to PCIe 1x 

- [x] * 2x SerDes pairs connected over capacitors to connector

- [x] GPIOs

- [x] 3 Buttons

- [x] 2 DIP SW

- [x] 8 LEDs

* Only available if ECP5 SerDes chip is used


## Electrical and mechanical specifications

![dimensions](https://github.com/intergalaktik/ulx4m-documentation/blob/main/pic/ulx4m-ls-dimensions.png)

## Pinout

## Power 

ULX4M requires at least 500mA to function safely.
5V power supply can be provided over USB port or over GPIO.
3.3V OUTPUT from ULX3S can handle up to 1A if apropriate input power is used.

## ULX4M-LS description and instructions

Missing

## ULX4M-LS description and instructions

NEEDS UPDATE!!!
![LD short desc](https://github.com/intergalaktik/ulx4m-documentation/blob/main/pic/ulx4m-ls-short-desc.png)

NEEDS UPDATE!!!
![Waveshare short desc](https://github.com/intergalaktik/ulx4m-documentation/blob/main/pic/Waveshare-ulx4m-ls-v2-explain.png)

![Rawspberry IO](https://github.com/intergalaktik/ulx4m-documentation/blob/main/pic/ulx4m-ls-v003-rpi-io_03_crop.jpg)

NEEDS UPDATE!!!
![GPIO](https://github.com/intergalaktik/ulx4m-documentation/blob/main/pic/ls-gpio.png)

On this version SD_CARD is aslo connected to GPIO pins, so we can share it with ESP32 as on ULX3S.

On this version FPGA JTAG pins are connected to some GPIO pins so you will not be able to use them as GPIOs, but you can acces JTAG over GPIO.

If Waveshare IO board is used it can be powered only from USB-C.

If you are using Raspberry IO board then you will need 12V DC input as micro USB does not have 5V connected.
There is one wire hack that can provide 5V from USB to the board.
If this hack is used USB HOST will not work as it will disable HUB chip.

### ULX4M-LS schematics

https://github.com/intergalaktik/ulx4m/blob/ulx4m-ls/doc/schematics.pdf

## ULX4M-LS design files

https://github.com/intergalaktik/ulx4m/tree/ulx4m-ls

### ULX4M-LS LPF

https://github.com/intergalaktik/ulx4m/blob/ulx4m-ls/doc/constraints/prototype/ulx4m_ls_v003.lpf


### JTAG over GPIO

### DFU bootloader for ULX4M-LS

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
    Bus 001 Device 117: ID 1d50:614b OpenMoko, Inc. ULX4M-LS(DFU)

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

## Chat and support

Discord chanel

  - https://discord.gg/qwMUk6W (problems/question/general chat)

## INSTRUCTIONS FOR SAFE USE

This product shall be connected to an computer/laptop USB port or external power supply rated at 5V DC with maximum current of 2A.
Any external power supply used with ULX4M shall comply with relevant regulations and standards applicable in the country of intended use.
This product should be operated in a well ventilated environment and should not be covered.
This product should be placed on a stable, flat, non-conductive surface in use and should not be contacted by conductive items. 
The connection of unapproved devices to any receptacle may affect compliance or results in damage to the unit and invalidate the warranty. Do not expose it to water, moisture or place on a conductive surface whilst in operation
Do not expose it to heat from any source; ULX4M is designed for reliable operation at normal ambient room temperatures.
Take care whilst handling to avoid mechanical or electrical damage to the printed circuit board and connectors.
Avoid handling ULX4M while it is powered. Only handle by the edges to minimize the risk of electrostatic discharge damage.
All peripherals used with ULX4M should comply with relevant standards for the country of use and be marked accordingly to ensure that safety and performance requirements are met.
ULX4M requires at least 500mA to function safely.