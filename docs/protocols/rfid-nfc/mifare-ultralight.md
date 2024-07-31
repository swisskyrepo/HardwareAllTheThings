# HF - Mifare UltraLight

* Ultralight C (3DES authentication)
* Ultralight EV1
* NTAG2

## Chinese backdoor

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

## Simulate

```powershell
hf 14a sim 2 <7-byte tag>
```


## References

* []()