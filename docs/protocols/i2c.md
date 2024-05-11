# I2C

I2C (Inter-Integrated Circuit), pronounced "I-squared-C" or "I-two-C", is a popular communication protocol mainly used for low-speed, short-distance communication in embedded systems.


## Analysis

:warning: Enable I2C on the Raspberry Pi via `raspi-config`

* i2c-tools
    ```ps1
    sudo apt-get install i2c-tools
    i2cdetect -y 1
    ```

* eeprog
    ```ps1
    wget http://darkswarm.org/eeprog-0.7.6-tear5.tar.gz
    tar -xvf eeprog-0.7.6-tear5.tar.gz eeprog-0.7.6-tear12/
    cd eeprog-0.7.6-tear12/
    make
    sudo make install
    ```

* HydraBus
    ```ps1
    i2c1> show pins
    i2c1> scan
    ```


## Read / Write

* Read: `./eeprog  -x /dev/i2c-1 0x50 -16  -r 0x00:0x10`
* Write: `echo "hello" | ./eeprog -f -16 -w 0 -t 5 /dev/i2c-1 0x50`


## References

* [How to Scan and Detect I2C Addresses - Carter Nelson](https://learn.adafruit.com/scanning-i2c-addresses/raspberry-pi)
