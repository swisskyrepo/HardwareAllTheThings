# ESP32

## Tools

* [espressif/esptool](https://github.com/espressif/esptool) - Espressif SoC serial bootloader utility
* [jmswrnr/esp32knife](https://github.com/jmswrnr/esp32knife) - Tools for ESP32 firmware dissection


## Flashing

The ESP32 microprocessor uses the Xtensa instruction set, use `Tensilica Xtensa 32-bit little-endian` in Ghidra.

* Dump the flash
    ```ps1
    esptool -p COM7 -b 115200 read_flash 0 0x400000 flash.bin
    ```

* Dissect the flash
    ```ps1
    python esp32knife.py --chip=esp32 load_from_file ./flash.bin
    ```

* Flash the new firmware
    ```ps1
    # repair the checksum
    python esp32fix.py --chip=esp32 app_image ./patched.part.3.factory 
    esptool -p COM7 -b 115200 write_flash 0x10000 ./patched.part.3.factory.fixed
    ```

## References

* [ESP32-reversing - BlackVS](https://github.com/BlackVS/ESP32-reversing)

