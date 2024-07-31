# HF - Mifare DESFire

## DESFire® Format

* Mifare DESFire [MF3ICD40](https://arstechnica.com/information-technology/2011/10/researchers-hack-crypto-on-rfid-smart-cards-used-for-keyless-entry-and-transit-pass/): uses 3DES encryption, product discontinued.
* **Mifare DESFire EV1** : Secure channel that can work with all the keys: DES, two-key 3DES, three-key 3DES and AES. Limited to 28 applications containing a maximum of 32 files per application.
* **Mifare DESFire EV2** : The newest channel that can work with aes key only
* **Mifare DESFire EV3** : Enhanced transaction speed and even better multi-application support.

Each card has a master application with AID `0x000000` that saves the card's configuration.
The memory organization of DESFire supports up to 28 applications on the card and up to 32 files in each application.

* Master Application (0x000000)
* Applications
    * Files


### Applications

```ps1
hf mfdes lsapp --no-auth # show applications list without authentication
hf mfdes lsapp # show applications list with authentication from default settings
hf mfdes lsapp --files # show applications list with their files
hf mfdes getaids --no-auth # this command can return a simple AID list if it is enabled in the card settings
```

Each application has an individual set of up to 14 application keys (can be AES-128 or DES keys)


### Files

* **Standard** File: used for static data like a employee ID
* **Backup** File: like a Standard File but with a "Commit" feature that allows for secure storage of data, e.g. a changeable user password
* **Value** File: storing changeable value information, e.g. the amount on a canteen payment card
* **Linear Record** File: storing a defined number of records, e.g. collecting of goodies
* **Cyclic Record** File: like a Linear Record file but this file doesn't get "full" but the oldest entry gets overwritten by a new entry, e.g. for a log file

Each file has it’s own **Communication Mode**:

* **Plain**: all data transfer between the NFC tag and the NFC reader is done in plain
* **MACed**: like in Plain mode the communication is is readable but secured by an appended MAC
* **Encrypted**: the communication is not visible be anyone, but only who posses the used key is been able to read the data.

**Dump files**

```ps1
hf mfdes lsfiles --aid 123456 -t aes # file list for application 123456 with aes key
hf mfdes dump --aid 123456 # shows files and their contents from application 123456
```

**Read/Write files**

Read

```ps1
hf mfdes read --aid 123456 --fid 01 # autodetect file type (with hf mfdes getfilesettings) and read its contents
hf mfdes read --aid 123456 --fid 01 --type record --offset 000000 --length 000001 # read one last record from a record file
```

Read via ISO command set

```ps1
hf mfdes read --aid 123456 --fileisoid 1000 --type data -c iso # select application via native command and then read file via ISO
hf mfdes read --appisoid 0102 --fileisoid 1000 --type data -c iso # select all via ISO commands and then read
hf mfdes read --appisoid 0102 --fileisoid 1100 --type record -c iso --offset 000005 --length 000001 # read one record (number 5) from file ID 1100 via ISO command set
hf mfdes read --appisoid 0102 --fileisoid 1100 --type record -c iso --offset 000005 --length 000000 # read all the records (from 5 to 1) from file ID 1100 via ISO command set
```

Write

```ps1
hf mfdes write --aid 123456 --fid 01 -d 01020304 # autodetect file type (with hf mfdes getfilesettings) and write data with offset 0
hf mfdes write --aid 123456 --fid 01 --type data -d 01020304 --commit # write backup data file and commit
hf mfdes write --aid 123456 --fid 01 --type value -d 00000001 # increment value file
hf mfdes write --aid 123456 --fid 01 --type value -d 00000001 --debit # decrement value file
hf mfdes write --aid 123456 --fid 01 --type record -d 01020304 # write data to a record file
hf mfdes write --aid 123456 --fid 01 --type record -d 01020304 --updaterec 0 # update record 0 (latest) in the record file.
```

Write via iso command set

```ps1
hf mfdes write --appisoid 1234 --fileisoid 1000 --type data -c iso -d 01020304 # write data to std/backup file via ISO command set
hf mfdes write --appisoid 1234 --fileisoid 2000 --type record -c iso -d 01020304 # send record to record file via ISO command set
```


## Default Keys

Changing the default keys is a crucial step in the deployment of MIFARE DESFire cards to prevent unauthorized cloning and access.

* Default AES key
    ```
    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
    ```
* Default DES key
    ```
    00 00 00 00 00 00 00 00
    ```

Use a key to get UID

```ps1
hf mfdes getuid # authenticate with default key
hf mfdes getuid -s d40 # via d40 secure channel
hf mfdes getuid -s ev2 -t aes -k 11223344556677889900112233445566 # via ev2 secure channel with specified aes key
```

```ps1
hf mfdes detect # simply detect key for master application (PICC level)
hf mfdes detect --save # detect key and save to defaults. look after to output of hf mfdes default
hf mfdes detect -s d40 # detect via channel d40
hf mfdes detect --dict mfdes_default_keys # detect key with help of dictionary file
hf mfdes detect --aid 123456 -n 2 # detect key 2 from application with AID 123456
```

```ps1
hf mfdes auth -n 0 -t des -k 1122334455667788 --aid 123456 # try application 123456 master key
hf mfdes auth -n 0 -t aes --save # try PICC AES master key and save the configuration to defaults if authentication succeeds
```


## UID check

The UID of the modifiable MIFARE DESFire® Compatible UID tags consists of two parts: the UID itself and the BCC. The BCC is a checksum value calculated from the UID. If the BCC is incorrect, the tag will be rejected by the reader.

```ps1
hf 14a raw -s -c 02 00 ab 00 00 00 07 xx xx xx xx xx xx xx xx xx
```

For MIFARE DESFire cards, Flipper Zero is able to emulate only the UID.

UID rewritable cards:
- [LAB 401 - MODIFIABLE MIFARE DESFIRE® COMPATIBLE UID](https://lab401.com/fr/products/desfire-compatible-emulator-uid-modifiable)
- [LAB 401 - MIFARE DESFIRE® COMPATIBLE MODIFIABLE UID / ATQA / SAK / ATS / APDU](https://lab401.com/products/mifare-desfire-ev2-compatible-modifiable-uid-atqa-sak-ats-apdu)


## References

* [Mifare DESFire EV3 — a beginner tutorial (Android Java) using the DESFire for Android tools - AndroidCrypto - Feb 18, 2024](https://medium.com/@androidcrypto/mifare-desfire-ev3-a-beginner-tutorial-android-java-using-the-desfire-for-android-tools-00aaecb8fa93)
* [Mifare DESFire EVx NFC tag: Change the Master Application Key from DES to AES (Android/Java) - AndroidCrypto - Jun 19, 2024](https://medium.com/@androidcrypto/mifare-desfire-evx-nfc-tag-change-the-master-application-key-from-des-to-aes-android-java-e5dcf02861ff)
* [DESFireChangeMasterAppKey - AndroidCrypto - Jun 19, 2024](https://github.com/AndroidCrypto/DESFireChangeMasterAppKey)
* [Notes on MIFARE DESFire - iceman1001 - 2021](https://github.com/RfidResearchGroup/proxmark3/blob/master/doc/desfire.md)
* [Mifare DESFire - An Introduction - David Coelho - 19 mai 2019](https://www.linkedin.com/pulse/mifare-desfire-introduction-david-coelho/)
* [AN-315 - Understanding MIFARE DESFire Credentials - ICT | Protege - 11-May-22](https://ict.co/media/3zznd34z/an-315_understanding_protege_mifare_desfire_credentials.pdf)
* [MIFARE DESFire gallagher-research - megabug](https://github.com/megabug/gallagher-research/blob/master/formats/card-specific/mifare-desfire.md)