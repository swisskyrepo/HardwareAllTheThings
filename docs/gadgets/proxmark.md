# Proxmark

Proxmark3 is a powerful tool for RFID research, allowing you to read, write, and clone various types of RFID tags. This cheatsheet provides a quick reference for common Proxmark3 commands and usage.

## Table of Contents

1. [Setup and Configuration](#setup-and-configuration)
2. [Basic Commands](#basic-commands)
3. [Low Frequency (LF) Commands](#low-frequency-lf-commands)
4. [High Frequency (HF) Commands](#high-frequency-hf-commands)
5. [Mifare Classic Commands](#mifare-classic-commands)
6. [Mifare Ultralight Commands](#mifare-ultralight-commands)
7. [iClass Commands](#iclass-commands)
8. [Troubleshooting](#troubleshooting)

## Setup and Configuration

### Installation

1. **Clone the Proxmark3 repository:**

   ```sh
   git clone https://github.com/RfidResearchGroup/proxmark3.git
   ```

2. **Navigate to the Proxmark3 directory:**

   ```sh
   cd proxmark3
   ```

3. **Compile the firmware and software:**

   ```sh
   make clean && make all
   ```

4. **Flash the Proxmark3 device:**

   ```sh
   ./pm3-flash-all
   ```

### Connecting to Proxmark3

1. **Connect the Proxmark3 device to your computer via USB.**
2. **Open a terminal and run the Proxmark3 client:**

   ```sh
   ./pm3
   ```

## Basic Commands

### Hardware Commands

- `hw status`: Check the hardware status.
- `hw tune`: Tune the antenna.
- `hw version`: Display the hardware version.

### General Commands

- `help`: Display the help menu.
- `exit`: Exit the Proxmark3 client.

## Low Frequency (LF) Commands

### Reading LF Tags

- `lf search`: Search for LF tags.
- `lf read`: Read data from an LF tag.
- `lf em 410x`: Emulate an EM410x tag.

### Cloning LF Tags

- `lf clone`: Clone an LF tag.
- `lf sim`: Simulate an LF tag.

## High Frequency (HF) Commands

### Reading HF Tags

- `hf search`: Search for HF tags.
- `hf mifare r`: Read data from a Mifare Classic tag.
- `hf mifare c`: Clone a Mifare Classic tag.

### Writing HF Tags

- `hf mifare w`: Write data to a Mifare Classic tag.
- `hf mifare p`: Print data from a Mifare Classic tag.

## Mifare Classic Commands

### Authentication

- `hf mifare a`: Authenticate with key A.
- `hf mifare b`: Authenticate with key B.

### Reading and Writing

- `hf mifare rdbl`: Read a block from a Mifare Classic tag.
- `hf mifare wrbl`: Write a block to a Mifare Classic tag.

### Key Recovery

- `hf mifare nest`: Nested attack for key recovery.
- `hf mifare chk`: Check keys for a Mifare Classic tag.

## Mifare Ultralight Commands

### Reading and Writing Mifare

- `hf mful rd`: Read data from a Mifare Ultralight tag.
- `hf mful wr`: Write data to a Mifare Ultralight tag.

### Cloning Mifare

- `hf mful clone`: Clone a Mifare Ultralight tag.

## iClass Commands

### Reading and Writing iClass

- `hf iclass rd`: Read data from an iClass tag.
- `hf iclass wr`: Write data to an iClass tag.

### Cloning iClass

- `hf iclass clone`: Clone an iClass tag.

## Troubleshooting

### Common Issues

- **Device not recognized**: Ensure the Proxmark3 device is properly connected and drivers are installed.
- **Antenna tuning**: Use the `hw tune` command to tune the antenna for better performance.
- **Firmware issues**: Re-flash the Proxmark3 device using the `pm3-flash-all` command.

### Debugging

- `hw status`: Check the hardware status for any issues.
- `hw version`: Verify the firmware version.
