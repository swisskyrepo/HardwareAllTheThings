# NFC - RFID

> Radio Frequency Identification (RFID) & Near Field Communication (NFC)

## Notes about card types

### High Frequency

Around 13.56 MHz.

* **MIFARE Classic 1K/4K**: basically just a memory storage device. This memory, either 1024 or 4096 bytes, is divided into sectors and blocks. Most of the time used for regular access badges and has really simple security mechanisms for access control
* **MIFARE Ultralight**: a 64 bytes version of MIFARE Classic. It’s low costs make it widely used as disposable tickets for events or transportation.
* **MIFARE Plus**: announced as a replacement of MIFARE Classic. The Plus subfamily brings the new level of security up to 128-bit AES encryption.
* **MIFARE DESFire**: those tags come pre-programmed with a general purpose DESFire operating system which offers a simple directory structure and files, and are the type of MIFARE offering the highest security levels.


### Low Frequency

Usually around 125 kHz.

* HID
* EM410X
* Indala


## Replay Attacks

Replay attack is a technique where a malicious user could implement a device to intercept a NFC transaction and redeem it later, using other device or even in different location. 


## Relay Attack

The relay attack is a technique where a malicious user implements a man in the middle attack. The attacker(APDUer) is capable to intercept, manipulate and change the transaction in real time to take advantage of it. [https://en.wikipedia.org/wiki/Relay_attack](https://en.wikipedia.org/wiki/Relay_attack)

* NFC Payment Relay Attacks - [intro-to-nfc-payment-relay-attacks/](https://salmg.net/2018/12/01/intro-to-nfc-payment-relay-attacks/)
* NFCopy85 is a 10 dollars device to make replay attacks against NFC payment systems - [nfcopy85](https://salmg.net/2019/06/16/nfcopy85/)


## References

* [RFID Hacking with The Proxmark 3 - Kevin Chung - May 29, 2017](https://blog.kchung.co/rfid-hacking-with-the-proxmark-3/)
* [RFID – Le clone parfait - Alex - 12 juillet 2017](https://www.latelierdugeek.fr/2017/07/12/rfid-le-clone-parfait/)
* [Proxmark 3, le couteau suisse RFID - Bourdin Pierre - octobre 2017](https://connect.ed-diamond.com/MISC/MISCHS-016/Proxmark-3-le-couteau-suisse-RFID)
* [A 2018 practical guide to hacking NFC/RFID - Sławomir Jasek - 4.06.2018](https://smartlockpicking.com/slides/Confidence_A_2018_Practical_Guide_To_Hacking_RFID_NFC.pdf)
* [Infosec - NFC Mifare - @SecurityGuill](https://securityguill.com/nfc_mifare.html)