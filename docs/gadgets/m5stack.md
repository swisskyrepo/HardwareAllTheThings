# Evil M5Core2

> Evil-M5Core2 is an easy Evil portal and rogue app deployement software designed to work on M5Stack Core2.

![Evil-M5Core2](https://raw.githubusercontent.com/7h30th3r0n3/Evil-M5Core2/main/Github-Img/menu-1.jpg)


## Features

* **WiFi Network Scanning**: Identify and display nearby WiFi networks.
* **Network Cloning**: Check information and replicate networks for in-depth analysis.
* **Captive Portal Management**: Create and operate a captive portal to prompt users with a page upon connection.
* **Credential Handling**: Capture and manage portal credentials.
* **Remote Web Server**: Monitor the device remotely via a simple web interface that can provide credentials and upload portal that store file on SD card.
* **Sniffing probes**: Sniff and store on SD near probes.
* **Karma Attack**: Try a simple Karma Attack on a captured probe.
* **Automated Karma Attack**: Try Karma Attack on near probe automatically


## Firmwares

* Firmware: [7h30th3r0n3/Evil-M5Core2](https://github.com/7h30th3r0n3/Evil-M5Core2)

**Requirements**:

* `M5Stack` boards manager
* `M5Unified` library


**Install**:

* Connect your `M5Core2` to your computer.
* Open the `Arduino IDE` and load the provided code.
* Ensure `M5unified` and `adafruit_neopixel` libraries are installed.
* Ensure `esp32` and `M5stack` board are installed. (Error occur with esp32 `3.0.0-alpha3`, please use esp32 `v2.0.14` and below)
* Place SD file content needed on the SD card. (IMG startup and sites folder)
* Upload the script to your `M5Core2` device.
* Restart the device if needed.


## References

* [Evil-M5Core2 v1.1.3 - Serial Command - Github Project](https://github.com/7h30th3r0n3/Evil-M5Core2)
* [Evil Portal Meets Marauder on M5Stack!! Evil-M5Core2 Is the Best of Both Worlds! - Talking Sasquach - 7 jan 2024](https://youtu.be/jcVm4cysmnE)