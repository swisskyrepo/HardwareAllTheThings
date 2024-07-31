# iCopy-X

iCopy-X is a "super" automated handheld RFID copier based on Proxmark3. iCopy-X can read, crack, duplicate, sniff and simulate without the use of a PC.

* [iCopy-X-Community](https://github.com/iCopy-X-Community)
    * [iCopy-X-Community/icopyx-upstream](https://github.com/iCopy-X-Community/icopyx-upstream) - Collecting elements provided by the manufacturer
    * [iCopy-X-Community/icopyx-community-pm3](https://github.com/iCopy-X-Community/icopyx-community-pm3) - Scrap repo for various tests
    * [iCopy-X-Community/icopyx-teardown](https://github.com/iCopy-X-Community/icopyx-teardown)


## Update

Latest firmware: `1.0.90 2022-08-16`

* [icopy-x.com/otasys](https://icopy-x.com/otasys/index.php)


**Step 1**: Enter the device S/N (found under the “About” menu) on the website and download the upgrade package to your PC.

**Step 2**: Connect the iCopy-X to your computer using the supplied USB TYPE C cable and delete any files that end in “.ipk” from the root directory.

**Step 3**: Copy the newly downloaded upgrade package to the root directory.

**Step 4**: Press "Ok" on the second page of the "About" menu on the iCopy-X to start the automatic upgrade.

**TIP**: Ensure that the serial number has been entered correctly before starting as this could cause the upgrade to fail.


## PC Mode

In PC-Mode, after connecting to the computer, open the client in the built-in U disk, you can directly use the Proxmark3 universal CMD to operate.

```ps1
COM Port (Check Device Manager, numbers only):  4
[=] Session log E:/CLIENT_X86/.proxmark3/logs/log_20240730.txt
[+] loaded from JSON file E:/CLIENT_X86/.proxmark3/preferences.json
[=] Using UART port /com4
[=] Communicating with PM3 over USB-CDC
[usb] pm3 -->
```


## References

* [iCopy-X - Kickstarter - iCopy-X: Handheld Smart RFID Multi-Tool - Nikola T. Lab](https://www.kickstarter.com/projects/nikola-lab/icopy-x-0)
* [icopy-x Official Website](https://icopy-x.com/)