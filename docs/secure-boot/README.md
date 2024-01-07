# Secure Boot

- Secure Boot is a **security feature implemented in modern computer systems**, primarily in those using the **Unified Extensible Firmware Interface (UEFI) firmware.**
- Its main purpose is to ensure that **only trusted** and **authenticated software** is loaded during the boot process, protecting the system against unauthorized or malicious code that could compromise its integrity and security.
- During boot, UEFI Secure Boot checks the signature of each piece of boot software, including **UEFI firmware drivers** (also known as option ROMs), **Extensible Firmware Interface** (EFI) applications, and the operating system drivers and binaries. If the signatures are valid or trusted by the **Original Equipment Manufacturer** (OEM), the machine boots and the firmware gives control to the operating system.


## References

* [Windows UEFI Bootkit in Rust - memN0ps](https://github.com/memN0ps/bootkit-rs)
* [AzureDocs - Secure Boot - MicrosoftDocs](https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/security/fundamentals/secure-boot.md)
* [Awesome UEFI Security - river-li](https://github.com/river-li/awesome-uefi-security#documentations-book)
