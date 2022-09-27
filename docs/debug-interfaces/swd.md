# SWD

### Summary

* SWD Pins

### SWD pins

* SWCLK: Clock into the core
* SWDIO: Data in / out

JTAG and SWD are similar and can be interfaced with each other:

| JTAG Mode | SWD Mode | Signal                                        |
| --------- | -------- | --------------------------------------------- |
| TCK       | SWCLK    | Clock into the core                           |
| TDI       | -        | JTAG test data input                          |
| TDO       | SWV      | JTAG Test data output / SWV trace data output |
| TMS       | SWDIO    | JTAG test mode select / SWD data in and out   |
| GND       | GND      | -                                             |
