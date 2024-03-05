# Firmware Reverse Engineering

## Loading bare-metal binaries into IDA

Requirements:

* The **load address** is the address in memory that the binary is being executed from.
* The **entry point** is the location within the binary where the processor starts executing.

⚠️ For ARM Arduino firwmare the entry point is located at **\_RESET** interruption.

> To load it properly in IDA, open the file, select ATMEL AVR and then select ATmega323\_L.

* ESP8266 : [https://github.com/themadinventor/ida-xtensa](https://github.com/themadinventor/ida-xtensa)


## Loading bare-metal binaries into Radare2

Radare2 can disassemble `avr`, `arduino` natively

```powershell
$ radare2 -A -a arm -b 32 ihex://Challenge_v3.hex
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] find and analyze function preludes (aap)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[x] Finding xrefs in noncode section with anal.in=io.maps
[x] Analyze value pointers (aav)
[x] Value from 0x00000000 to 0x10001018 (aav)
[x] 0x00000000-0x10001018 in 0x0-0x10001018 (aav)
[x] Emulate code to find computed references (aae)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.

[0x565e8640]> aaaa
[0xf7723a20]> afl
[0xf7723a20]> e asm.describe = true
[0xf7723a20]> s main
[0x0804873b]> pdf

To perform a case-insensitive search for strings use /i:
[0x0001d62c]> /i Exploding
Searching 9 bytes in [0x0-0x10001018]
hits: 1
0x0003819e hit1_0 .. N# NExploding Firmware ! N.

$ r2 -a avr /tmp/flash
[0x000000c4]> afr
[0x000000c4]> pd 17

$ rasm2 -a avr -d "0c94 751b 0c94 9d1b 0c94 d72c" 
jmp 0x36ea
jmp 0x373a
jmp 0x59ae
```


## Loading bare-metal binaries into Ghidra

SVD-Loader for Ghidra automates the entire generation of peripheral structs and memory maps for over 650 different microcontrollers

* SVD-Loader for Ghidra: Simplifying bare-metal ARM reverse engineering - [svd-loader/](https://leveldown.de/blog/svd-loader/)

**Usage**

* Load a binary file
* Open it in the code-browser, do not analyze it
* Run the SVD-Loader Script
* Select an SVD file
* Analyze the file


## ESPTool

ESP8266 and ESP32 serial bootloader utility : [espressif/esptool](https://github.com/espressif/esptool)

```powershell
josh@ioteeth:/tmp/reversing$ ~/esptool/esptool.py image_info recovered_file
esptool.py v2.4.0-dev
Image version: 1
Entry point: 4010f29c
1 segments
Segment 1: len 0x00568 load 0x4010f000 file_offs 0x00000008
```


## nRF5x Firmware disassembly tools

* [DigitalSecurity/nrf5x-tools](https://github.com/DigitalSecurity/nrf5x-tools)

```powershell
$ python3 nrfident.py bin firmwares/s132.bin
Binary file provided firmwares/s132.bin
Computing signature from binary
Signature:  d082a85351ee18ecfdc9dcb01352f5df3d938a2270bcadec2ec083e9ceeb3b1e
=========================
SDK version:  14.0.0
SoftDevice version: s132
NRF: nrf52832
=========================
SDK version:  14.1.0
SoftDevice version: s132
NRF: nrf52832
SoftDevice  :  s132
Card version :  xxaa
           *****
RAM address  :  0x20001368
RAM length   :  0xec98
ROM address  :  0x23000
ROM length   :  0x5d000
```


## Pure disassemblers

* Vavrdisasm -- vAVRdisasm will auto-recognize Atmel Generic, Intel HEX8, and Motorola S-Record files - [vsergeev/vavrdisasm](https://github.com/vsergeev/vavrdisasm)
* [ODA - The Online Disassembler](https://www.onlinedisassembler.com/odaweb/)
*   avr-objdump – gcc kit standard tool

    ```powershell
    $ avr-objdump -l -t -D -S main.bin > main.bin.dis
    $ avr-objdump -m avr -D main.hex > main.hex.dis
    ```


## Simulating AVR

> Programs compiled for Arduino can be simulated using AVR Studio or the newer Atmel Studio. I have used the former along with hapsim. Hapsim works by hooking into AVR Studio and can simulate peripherals like the UART, LCD etc.

```powershell
$ simulavr -P atmega128 -F 16000000 –f build-crumbuino128/ex1.1.elf
```


## UEFI Firmware

Parse BIOS/Intel ME/UEFI firmware related structures: Volumes, FileSystems, Files, etc - [theopolis/uefi-firmware-parser](https://github.com/theopolis/uefi-firmware-parser)

```ps1
sudo pip install uefi_firmware
$ uefi-firmware-parser --test ~/firmware/*
~/firmware/970E32_1.40: UEFIFirmwareVolume
~/firmware/CO5975P.BIO: EFICapsule
~/firmware/me-03.obj: IntelME
~/firmware/O990-A03.exe: None
~/firmware/O990-A03.exe.hdr: DellPFS
```


## References

* [GreHack22 - SecureDUO - chrisrdlg](https://github.com/chrisrdlg/gh22_SecureDuo)
* [Loader un binaire Arduino dans IDA - Posted on January 26, 2014 by thanatos](https://thanat0s.trollprod.org/2014/01/loader-un-binaire-arduino-dans-ida/)
* [REcon 2014 - Reverse Engineering Flash Memory For Fun and Benefit - Matt Oh](https://youtu.be/nTPfKT61730)
* [Reverse Engineering Flash Memory for Fun and Benefit - Jeong Wook (Matt) Oh](https://www.blackhat.com/docs/us-14/materials/us-14-Oh-Reverse-Engineering-Flash-Memory-For-Fun-And-Benefit-WP.pdf)