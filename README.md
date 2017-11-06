YI M1 Mirrorless Camera Firmware Unpacker
=========================================

A firmware unpacker for YI M1 firmware files. Currently does not much more than parsing the section headers of a firmware file and extracting the sections into separate files. Works with all firmware versions.

Requirements: Node.js & npm
 
Usage: 
 1. `npm install`
 1. Download [firmware](https://www.yitechnology.com/yi-m1-mirrorless-camera-firmware)
 1. `npm run unpack /path/to/firmware.bin`
 1. `npm run decompress /path/to/firmware.bin.0` (output from `unpack`)

The output will be a number of files (usually 4) named `firmware.bin.{sectionNumber}[.{sectionId}]`.

Firmware analysis
-----------------

Firmware files consist of 4 sections.

| Section Number | Section Id | Size | Description |
| -------------- | ---------- | ---- | ----------- |
| 0              | *none*     | variable, ~7 MB | Most probably the actual firmware code. Contains two sections with 0x1000 byte length each, followed by compressed data (compressed with some kind of LZSS algorithm). |
| 1              | ND1        | variable, ~4 MB | Offset 0x1600000. Memory image that contains resources like bitmaps, fonts, and texts in different languages |
| 2              | IPL        | 128 kB, 0x20000 byte | Bootloader (Initial Program Loader) |
| 3              | PTBL       | 4 kB, 0x1000 byte | Partition table, unknown format |

### Section headers

The section headers inside the firmware file are simple strings with a length of 256 byte (0x100). Examples from FW 2.9.1-int:

```
LENGTH=7366656 C59Y1 VER=M1INT DVR=Ver1.37 SUM=937214718 ND1 IPL PTBL
ND1 LENGTH=4197888 C59Y1 VER=M1INT DVR=Ver1.37 SUM=299791776 OFFSET=23068672
IPL LENGTH=131072 C59Y1 VER=M1INT DVR=Ver1.37 SUM=5714438
PTBL LENGTH=4096  C59Y1 VER=M1INT SUM=5181
```

The `LENGTH` is the length in bytes of the following section body. `SUM` is a simple checksum calculated by summing all bytes. `OFFSET` seems to be an offset in the camera's memory space to which the section is written. The first header has the IDs of the following headers appended. All following headers start with their ID.

### Hardware & Software Identification

Tons of interesting strings regarding the system can be found by runnings the Unix `strings` utility against the decompressed firmware. Some interesting strings regarding the hardware:

Section 0 / System

 * C:/XC_ODM/sdk/SDK_selfcheck/src/EV9x_DevEnv/btstack/bluesdk
 * BCM4343A1_00_1.002 -> Wifi radio?
 * WA1 37.4MHz Murata Type-1FJ BT4.1 OTP-BD -> Bluetooth radio?
 * Copyright (c) 2009-2010 Tokyo Electron Device Ltd
 * Broadcom BCM.%s 802.11 Wireless Controller %s
 * macaddr=00:90:4c:c5:12:38 -> Epigram MAC (Broadcom)
 * xtalfreq=374
 * bcm9
 * Copyright 2009 Murata Manufacturing Co.,Ltd

Section 2 / IPL

 * PureNAND IPL ev9x-v1.8t.r1864 (Mimasaka) [DEBUG BUILD] (Feb 10 2016 10:30:54)
 * EV9XES1.0
 * EV9XES2.0
 * Warning: EV9X ES1.0 does not use 513MHz, it was changed to 400MHz
 * ARM926_1 -> armv5te architecture
 * ARM926_2
 * BCH2K124

### Next steps

 * Identify the exact format of the first section / fix decompression
 * Identify partition table format and decode
 * Disassemble first section
 * Change something simple (e.g. the 500 shot limit in the beta firmware), repack FW file and upload to camera

Other Cameras
-------------

The Fujifilm X-A10 camera uses the same [firmware](http://www.fujifilm.com/support/digital_cameras/software/firmware/x/xa10/download.html) format and can be unpacked with this tool as well. The compression scheme of the first section is also similar. In comparison to the M1 it contains an additional fifth section called "A9262" that contains additional code. Also interesting is that the model ID is very similar (M1: C59Y1, X-A10: C5932).

Other Fujifilm cameras, e.g. Finepix S800, seem to use a similar firmware format, but with shorter headers, so these cannot be unpacked with this tool.

FAQ
---

> What's the purpose of this tool?

To lay the roots for a firmware hack.

> What can a potential firmware hack do?

 * Increase video bitrates
 * Add 24p video modes
 * Decrease JPEG compression
 * Change focus peaking color
 * Fix UX issues
   * Disable full shutter button press during video recording which stops the recording while a half press triggers focus
   
> How can I contribute?

Please open an issue, pull request, or drop me a mail at mg@protyposis.net

> How dare you write this in JavaScript?

Times are changing, and messing with the string headers in C seemed too much of a hassle. JS with Node also runs on virtually every platform.