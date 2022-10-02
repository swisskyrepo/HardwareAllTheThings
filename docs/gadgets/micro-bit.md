# Micro::bit

### Extract source code from firmware

When the source has been build from [https://makecode.microbit.org/#editor](https://makecode.microbit.org/#editor), the Javascript code is embedded into the firmware.

```python
import bincopy
import lzma
import sys
import subprocess
import json

# split firmware into raw and code
with open(sys.argv[1],'r') as f:
    fwstring = f.read()
    fwsplit = fwstring.split('\n\n')
    
    with open('fw_raw.hex', 'w') as g:
        g.write(fwsplit[0])
    with open('fw_code.hex', 'w') as g:
        g.write(fwsplit[1])

# Convert ihex to bin
f = bincopy.BinFile()
f.add_ihex_file('fw_code.hex')
binary = f.as_binary()
print("[+] ihex converted to binary")

## Extract code firmware, bruteforce offset
for i in range(200):
    with open('firmware.bin', 'w+b') as g:
        g.write(binary[i:])

    try:
        data = subprocess.run(["lzma", "firmware.bin", "-d", "--stdout"], capture_output=True)
        data = data.stdout.decode().split('}',1)
        data = data[1][1:]
        data = json.loads(data)
        print(data)
        print("\n[+] Javascript code")
        print(data['main.ts'])
    except Exception as e:
        continue
```

### Extract firmware using SWD

#### Connection

Solder wires on SWD pins:

![](https://i.ibb.co/FJZ3sr3/upload-c19e00cea2e28464c0fb9d4e8f6a6963.png)

Connect to an ST-LINK v2:

![](https://i.ibb.co/KrSJVKc/upload-2a0191d652b242e1762f75379af2b23c.png)

#### OpenOCD profile

Official datasheet of the nRF51822:

> https://infocenter.nordicsemi.com/pdf/nRF51822\_PS\_v3.1.pdf

Code section size:

![](https://i.ibb.co/Zz2wnry/upload-b2444a535e41dacbf5da5fd22dc66d50.png)

![](https://i.ibb.co/z7hgFqg/upload-feec06938e6d30aa212088c38227086e.png)

> hex(1024\*256) = 0x40000 => 0x00040000

```bash
init
reset init
halt
dump_image image.bin 0x00000000 0x00040000
exit
```

```bash
$ sudo openocd  -f /home/maki/tools/hardware/openocd/tcl/interface/stlink-v2-1.cfg -f /home/maki/tools/hardware/openocd/tcl/target/nrf51.cfg -f dump_fw.cfg
```

#### Python code

Content of `image.dd` file:

```bash
$ strings image.bin
[...]
main.py# Add your Python code here. E.g.
from microbit import *
while True:
    display.scroll('Hello, World!')
    displa
y.show(Image.HEART)
    sleep(1000)
    print("coucou")
    sleep(2000)
```
