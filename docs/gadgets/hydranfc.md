# HydraNFC

## Features

* Support of microSD (FAT16/FAT32) card up to 32GB
* Virtual Serial Port access through micro USB with VT100 terminal/shell
* Basic UID read for Vicinity/ISO15693
* Basic UID read for ISO14443-A/MIFARE ® card 4 or 7bytes UID
* Read MIFARE Ultralight® tag content (full dump)
* Tag Emulation UID ISO14443A & MIFARE Classic® 1K
* Sniffer mode in an autonomous/stand-alone mode
* Real-time ISO14443A sniffer mode

 
## Firmware

* [hydrabus/hydrafw_hydranfc_shield_v2](https://github.com/hydrabus/hydrafw_hydranfc_shield_v2) - HydraFW dedicated to HydraBus v1 / HydraNFC Shield v2

Using console, type `nfc` + `Enter` to enter NFC mode dedicated to HydraNFC Shield v2.

```ps1
> nfc
NFCv2> nfc-all
NFCv2> show
NFCv2> nfc-all scan
```


## References

* [HydraFW HydraNFC v2 guide - Benjamin Vernoux - Jul 4, 2021](https://github.com/hydrabus/hydrafw_hydranfc_shield_v2/wiki/HydraFW-HydraNFC-v2-guide)
* [HydraNFC Getting Started - Lab401 - 30 mai 2017](https://youtu.be/-bYXXqPaB4s)
* [HydraBus / HydraNFC unboxing & Assembly - Lab401 - 30 mai 2017](https://youtu.be/D-alGCsmqPU)
* [HydraNFC - LAB401 product presentation - 17 mai 2018](https://youtu.be/MCmCK9y7Ojk)