# ZigBee

> Zigbee is a specification for a suite of high-level communication protocols using low-power digital radios. It's designed for use in areas like home automation, medical data collection, industrial control systems, and other applications that require secure and reliable wireless communication. 

Zigbee is an IEEE 802.15.4-based, wireless networking standard, which is basically used for two-way communication between sensors and control systems. Zigbee is a short-range wireless communication standard like Bluetooth and Wi-Fi while covering a range of 10 to 100 meters.


## Tools

* [riverloopsec/killerbee](https://github.com/riverloopsec/killerbee) - IEEE 802.15.4/ZigBee Security Research Toolkit
* [APIMote](https://www.attify-store.com/products/apimote) - The ApiMote is a ZigBee security research hardware intended for researchers, students, utility companies, etc. to use for learning about and evaluating the security of IEEE 802.15.4/ZigBee systems as authorized.
* [Cognosec/SecBee](https://github.com/Cognosec/SecBee) (archived) - SecBee is a ZigBee security testing tool developed by Cognosec to test ZigBee implementations for security issues


## Default Trust Center Link Key

Zigbee includes several layers of security, including AES-128 encryption, to ensure that data is transmitted securely across the network.

The Zigbee Default Trust Center Link Key is a predefined cryptographic key used in Zigbee networks to secure the initial joining process of a new device to the network. It's part of the security measures implemented within the Zigbee protocol to ensure that only authorized devices can join a particular network.

When a new device wants to join a Zigbee network, it must first establish a secure connection with the Trust Center. To do this, the device and the Trust Center use the Default Trust Center Link Key to encrypt their communication.

For the profile "Home Automation" the default Trust Center Link Key is : `ZigBeeAlliance09` (`"5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39"`).

You can use it in Wireshark: Edit > Preferences > Protocols > Zigbee NWK, then "New" and write the key in hex format.

Example: [CVE-2020-28952 - Athom Homey Static and Well-known Keys](https://yougottahackthat.com/blog/1260/athom-homey-security-static-and-well-known-keys-cve-2020-28952)


## References

* [AN1233: Zigbee Security - Silabs](https://www.silabs.com/documents/public/application-notes/an1233-zigbee-security.pdf)
* [Zigbee Security 101 (Architecture And Security Issues) - February 11, 2023 - dattatray](https://payatu.com/blog/zigbee-security-101/)
* [Tout, tout, tout vous saurez tout sur le ZigBee / MISC n°86 - July 2016 - Kovacs Nicolas](https://connect.ed-diamond.com/MISC/misc-086/tout-tout-tout-vous-saurez-tout-sur-le-zigbee)
* [ZIGBEE SECURITY: BASICS (PART 2) - Vishruta Rudresh - November 8, 2017](https://research.kudelskisecurity.com/2017/11/08/zigbee-security-basics-part-2/)
* [ZIGBEE SECURITY: BASICS (PART 3) - Vishruta Rudresh - November 21, 2017](https://research.kudelskisecurity.com/2017/11/21/zigbee-security-basics-part-3/)
* [Understanding Zigbee and Wireless Mesh Networking - Ray Felch - 27 Aug 2021](https://www.blackhillsinfosec.com/understanding-zigbee-and-wireless-mesh-networking/)
* [ZigBee Penetration Testing: Strengthen Your IoT Security - RF Security - 12/08/2022](https://www.offensive-wireless.com/zigbee-penetration-testing/)
* [pCTF 2011 #32 That's no bluetooth - STALKR - TUESDAY, APRIL 26, 2011](https://blog.stalkr.net/2011/04/pctf-2011-32-thats-no-bluetooth.html)