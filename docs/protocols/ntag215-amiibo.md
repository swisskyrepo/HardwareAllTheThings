# NFC - Amiibo

> Amiibo are small figurines or cards produced by Nintendo that contain Near Field Communication (NFC) chips. These chips allow the Amiibo to interact with various Nintendo gaming systems, such as the Wii U, Nintendo 3DS, and Nintendo Switch.


## Tools

* [socram8888/amiitool](https://github.com/socram8888/amiitool) - Reverse-engineered amiibo cryptography


## Amiibo Encryption

Nintendo added their own layer of encryption and digital signing to increase security. The digital signing prevents you from blindly altering the game data bytes because then the signature will no longer match. Additionally, the signature is also based on the **UID** of the tag, so you can't simply copy the bytes from an Amiibo to a blank NTAG215 to clone it.


## Password Reverse Engineering

The password is derived from the 7-byte tag UID (Unique Identifier) of the Amiibo. The algorithm used to generate the password is as follows:

```ps1
password[0] = 0xAA ^ (uid[1] ^ uid[3])
password[1] = 0x55 ^ (uid[2] ^ uid[4])
password[2] = 0xAA ^ (uid[3] ^ uid[5])
password[3] = 0x55 ^ (uid[4] ^ uid[6])
```

The algorithm takes specific bytes of the UID, performs XOR operations with constant values (0xAA and 0x55), and combines them to form the 32-bit password.


## References

* [Reverse Engineering Nintendo Amiibo (NFC Toy) - Apr 27, 2020 - Kevin Brewster](https://kevinbrewster.github.io/Amiibo-Reverse-Engineering/)
* [Amiibo encryption reverse-engineering - Apr 11, 2015 - Marcos Del Sol Vives](https://www.reddit.com/r/amiibros/comments/328hqz/amiibo_encryption_reverseengineering/)