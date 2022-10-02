# SPI

### Dump Firmware via SPI

```powershell
sudo raspi-confi > Interface > SPI(P4)
NOTE: might need a press/hold the reset button
# check
sudo flashrom -p linux spi:dev=/dev/spidev0.0,spispeed=1000
# dump
sudo flashrom -p linux spi:dev=/dev/spidev0.0,spispeed=1000 -r dump.bin
```

### SPIFFS

```powershell
$ cd ~/.arduino15/packages/esp32/tools/esptool/2.3.1
$ python ./esptool.py -p /dev/ttyUSB0 -b 460800 read_flash 0x300000 0x0fb000 /tmp/spiffs.bin

$ cd ~/.arduino15/packages/esp32/tools/mkspiffs/0.2.3
$ ./mkspiffs -u /tmp/data -p 256 -b 8192 -s 1028096 /tmp/spiffs/bin
```

### References

* https://www.youtube.com/watch?v=Bn5zajZ4I5E
