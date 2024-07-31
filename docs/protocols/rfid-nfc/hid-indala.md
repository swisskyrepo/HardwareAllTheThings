# LF - HID & Indala

## HID & Indala

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


## HID : Examples - Card

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

### Write to an HID card

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


## References

* [Proxmark 3, le couteau suisse RFID - Bourdin Pierre - octobre 2017](https://connect.ed-diamond.com/MISC/MISCHS-016/Proxmark-3-le-couteau-suisse-RFID)
* [Badge Cloning: Clone HID Prox with Proxmark3 RDV4 - Standalone Mode - Tinker - October 22, 2018](https://web.archive.org/web/20210725163656/https://www.tinker.sh/badge-cloning-clone-hid-prox-with-proxmark3-rvd4/)