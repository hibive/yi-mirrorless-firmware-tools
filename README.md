YI M1 Mirrorless Camera Firmware Unpacker
=========================================

A firmware unpacker for YI M1 firmware files. Currently does not much more than parsing the section headers of a firmware file and extracting the sections into separate files. Works with all firmware versions.

Requirements: Node.js & npm
 
Usage: `npm run unpack /path/to/firmware.bin`

The output will be a number of files (usually 4) named `firmware.bin.{sectionNumber}.{sectionId}`.

Firmware analysis
-----------------

Firmware files consist of 4 sections.

| Section Number | Section Id | Size | Description |
| -------------- | ---------- | ---- | ----------- |
| 0              | *none*     | variable, ~7 MB | Most probably the actual firmware code. Seems to be lightly scrambled or infused with some kind of checksum every 8 bytes. Could not make much sense of it yet. Can easily be seen in string values, e.g. `Copyr.ight 200.9 Murata. Manufac.turing C.o.,Ltd` or `text./html.Spl.ain.Sx.Ucs.s.applic.ation/js.on([avasc.ript.ima.ge/jpeg` (this intermediate byte is always 0xFF when 8 more ASCII chars follow). |
| 1              | ND1        | variable, ~4 MB | Offset 0x1600000. Memory image that contains resources like bitmaps, fonts, and texts in different languages |
| 2              | IPL        | 128 kB, 0x20000 byte | unknown, second half empty |
| 3              | PTBL       | 4 kB, 0x1000 byte | unknown, mostly empty |

### Section headers

The section headers inside the firmware file are simple strings with a length of 256 byte (0x100). Examples from FW 2.9.1-int:

```
LENGTH=7366656 C59Y1 VER=M1INT DVR=Ver1.37 SUM=937214718 ND1 IPL PTBL
ND1 LENGTH=4197888 C59Y1 VER=M1INT DVR=Ver1.37 SUM=299791776 OFFSET=23068672
IPL LENGTH=131072 C59Y1 VER=M1INT DVR=Ver1.37 SUM=5714438
PTBL LENGTH=4096  C59Y1 VER=M1INT SUM=5181
```

The `LENGTH` is the length in bytes of the following section body. `SUM` is a simple checksum calculated by summing all bytes. `OFFSET` seems to be an offset in the camera's memory space to which the section is written. The first header has the IDs of the following headers appended. All following headers start with their ID.

A similar header format can be found in the firmware of an unknown device C5932 / C5932-v84. A similar format, but with shorter length, can also be found on some Fujifilm cameras (e.g. Finepix S800).

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