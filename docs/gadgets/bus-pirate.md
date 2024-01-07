# Bus Pirate

![MOSI-MISO](https://iotmyway.files.wordpress.com/2018/05/mode-guide.png)


## Update Bus Pirate

```powershell
git clone https://github.com/BusPirate/Bus_Pirate.git
cd Bus_Pirate/package/BPv4-firmware/pirate-loader-v4-source/pirate-loader_lnx
sudo ./pirate-loader_lnx --dev=/dev/ttyACM0 --hex=../BPv4-firmware-v6.3-r2151.hex
```

```powershell
# Identify EEPROM chip
sudo flashrom -p buspirate_spi:dev=/dev/ttyUSB0

# Dump firmware using a bus pirate (SPI)
sudo flashrom -p Buspirate_spi:dev=/dev/ttyUSB0,spispeed=1M -c (Chip name)  -r (Name.bin)
```


## References

* [Bus Pirate Unboxing - Toolkit - Hacker Warehouse - 4 juin 2018](https://youtu.be/lP8vMvBu3Bg)