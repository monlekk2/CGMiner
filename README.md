
# CGMiner: Multi-Threaded, Multi-Pool Bitcoin Miner

---

## Overview

**CGMiner** is a multi-threaded, multi-pool miner designed for Bitcoin using FPGA and ASIC hardware. This powerful tool supports a broad range of mining devices and protocols—including standard getwork and the more efficient stratum protocol. The code is provided completely free of charge by its developer in his spare time under the **GPLv3** license, and donations are highly appreciated!

---

## Table of Contents

- [Features](#features)
- [Usage Summary](#usage-summary)
- [Building CGMiner](#building-cgminer)
- [Usage Instructions](#usage-instructions)
- [Advanced Options](#advanced-options)
- [USB Device Setup](#usb-device-setup)
  - [Windows](#windows)
  - [Linux](#linux)
  - [OSX](#osx)
- [Multipool Strategies and Quotas](#multipool-strategies-and-quotas)
- [Logging and Benchmarking](#logging-and-benchmarking)
- [RPC API](#rpc-api)
- [FAQ](#faq)
- [License](#license)
- [Contact](#contact)

---

## Features

- **Multi-threaded Mining:** Optimized for high-performance mining across multiple threads.
- **Multi-Pool Support:** Connect to several pools simultaneously with strategies like failover, round-robin, rotate, and load-balance.
- **Broad Hardware Compatibility:** Supports various ASIC and FPGA miners (e.g., Avalon, Antminer, BitFury, and many others).
- **Proxy Support:** Full support for http, socks4, socks5 (and variants) proxies.
- **Dynamic Configuration:** Save configurations from the interactive menu and load them on startup.
- **Detailed Logging & Monitoring:** Extensive logging, per-device statistics, and real-time status displays.
- **Benchmark Mode:** Test and optimize your mining performance without submitting shares.

---

## Usage Summary

### Single Pool Mining

```bash
cgminer -o http://pool:port -u username -p password
```

### Multiple Pools

```bash
cgminer -o http://pool1:port -u pool1username -p pool1password         -o http://pool2:port -u pool2username -p pool2password
```

### Using a Standard HTTP Proxy

```bash
cgminer -o "http:proxy:port|http://pool:port" -u username -p password
```

### Using a SOCKS5 Proxy

```bash
cgminer -o "socks5:proxy:port|http://pool:port" -u username -p password
```

### Stratum Protocol Mining

```bash
cgminer -o stratum+tcp://pool:port -u username -p password
```

### Solo Mining (with local bitcoind)

```bash
cgminer -o http://localhost:8332 -u username -p password --btc-address 15qSxP1SQcUX3o4nhkfdbgyoWEFMomJ4rZ
```

---

## Building CGMiner

### Dependencies

- **Mandatory:**
  - `pkg-config` ([pkg-config](http://www.freedesktop.org/wiki/Software/pkg-config))
  - `libtool` ([GNU libtool](http://www.gnu.org/software/libtool/))
- **Optional:**
  - **libcurl** development library ([libcurl](http://curl.haxx.se/libcurl/))  
    *(If using a libcurl version older than 7.19.4, some options may be unavailable.)*
  - **Curses** development library (e.g., `libncurses5-dev` or `libpdcurses` for Windows)
  - **libusb-1.0** (e.g., `libusb-1.0-0-dev`)
  - **libudev** (e.g., `libudev-dev` on Linux)
  - **uthash** and **libjansson** (included if not available)

If building from the Git repository, install also:

- `autoconf`
- `automake`

On Ubuntu, you might use:

```bash
sudo apt-get install build-essential autoconf automake libtool pkg-config libcurl4-openssl-dev libudev-dev libusb-1.0-0-dev libncurses5-dev
```

### CGMiner-Specific Configuration Options

You can customize your build with options such as:

- `--enable-ants1`   *Antminer S1 (default disabled)*
- `--enable-ants2`   *Antminer S2 (default disabled)*
- `--enable-avalon`   *Avalon (default disabled)*
- `--enable-avalon2`  *Avalon2/3 (default disabled)*
- `--enable-avalon4`  *Avalon4/4.1/6 (default disabled)*
- `--enable-avalon7`  *Avalon7 (default disabled)*
- `--enable-avalon8`  *Avalon8 (default disabled)*
- `--enable-bab`    *BlackArrow Bitfury (default disabled)*
- …and many more (see full list in the source documentation).

### Basic *nix Build Instructions

```bash
./autogen.sh      # Only if building from the git repository
CFLAGS="-O2 -Wall -march=native" ./configure [your options]
make
```

*No installation is necessary, but you can run `make install` to install CGMiner into your system.*

### Building for Windows

For Windows, cross-compiling with [mxe](http://mxe.cc/) is recommended (use the 32-bit toolchain):

```bash
export PATH=(path/to/mxe)/usr/bin/:$PATH
CFLAGS="-O2 -Wall -W -march=i686" ./configure --host=i686-pc-mingw32 [your options]
make
```

---

## Advanced Options

CGMiner offers many advanced features:

- **Proxy Configuration:** Configure individual proxies for each pool.
- **USB Management:** Options to restrict scanning (`--usb`), display detailed USB device info (`--usb-dump`), and manage hotplug behavior.
- **Multipool Strategies:** Select from failover (default), round robin, rotate, or load balance strategies.
- **Quotas:** Use the `--quota` option to assign quotas to pools for load balancing.
- **Logging:** Enable detailed logging to stderr or to a file, and use the `--sharelog` option to record share data in CSV format.
- **Benchmarking:** Run benchmark mode with `--benchmark` or use a custom work file with `--benchfile`.

---

## USB Device Setup

### Windows

1. **Driver Installation:**  
   Install the **WinUSB** driver (do **not** use the ftdi_sio driver).
2. **Using Zadig:**  
   Run the [zadig utility](https://zadig.akeo.ie/) as administrator. Once your device (e.g., "BitFORCE SHA256 SC") appears, install or replace the driver with WinUSB.
3. **Troubleshooting:**  
   If you still see permission errors, try unplugging and replugging the device or reboot your system.

### Linux

1. **Udev Rules:**  
   Copy the `01-cgminer.rules` file to the udev rules directory:
   ```bash
   sudo cp 01-cgminer.rules /etc/udev/rules.d/
   ```
2. **User Permissions:**  
   Add your user to the `plugdev` group:
   ```bash
   sudo usermod -a -G plugdev $(whoami)
   ```
   If the `plugdev` group does not exist, create it:
   ```bash
   sudo groupadd plugdev
   ```
3. **Restart:**  
   Restart udev or simply reboot your system.

### OSX

1. **Driver Unloading:**  
   Unload conflicting drivers (especially for BitFury USB sticks):
   ```bash
   sudo kextunload -b com.apple.driver.AppleUSBCDC
   sudo kextunload -b com.apple.driver.AppleUSBCDCACMData
   ```
2. **USB Limits:**  
   If necessary, increase USB device limits by editing `/etc/sysctl.conf` and rebooting.
3. **Permissions:**  
   Some devices may require superuser access; run CGMiner with `sudo` if needed.

---

## Multipool Strategies and Quotas

CGMiner provides several strategies for managing multiple pools:

- **Failover (Default):** Prioritizes pools in the order listed; moves to backup pools if the primary lags.
- **Round Robin:** Switches pools when the current one becomes idle.
- **Rotate:** Cycles through active pools at user-defined intervals.
- **Load Balance:** Distributes work across pools based on quotas.

### Setting Quotas

Assign quotas using the `--quota` option on the command line or in configuration files. For example:

```bash
--url poola:porta -u usernamea -p passa --quota "2;poolb:portb" -u usernameb -p passb
```

This configuration assigns approximately 1/3 of the work to pool A and 2/3 to pool B.

---

## Logging and Benchmarking

- **Logging to File:**  
  Redirect stderr to a file to capture logs:
  ```bash
  ./cgminer -o pool -u user -p pass 2>logfile.txt
  ```
- **Share Logging:**  
  Use the `--sharelog` option to log each share found in CSV format.
- **Benchmark Mode:**  
  Test performance with:
  ```bash
  cgminer --benchmark
  ```
  or specify a custom work file with `--benchfile`.

---

## RPC API

For details on the RPC API that allows remote monitoring and management, please refer to the `API-README.md`.

---

## FAQ

**Q: Why does my miner show all zero values?**  
A: Bitcoin mining with a CPU or GPU is not feasible today. Dedicated ASIC hardware is required.

**Q: My USB devices aren’t all detected.**  
A: Not all USB hubs supply sufficient power. Use a powered hub and ensure proper driver installation/configuration.

**Q: Can I use multiple pools?**  
A: Yes, you can configure multiple pools either via the command line or by editing the configuration file. See the usage examples above.

**Q: The build fails with gcc errors.**  
A: Remove the `-march=native` flag from your CFLAGS if your gcc version does not support it.

---

## License

CGMiner is released under the **GPLv3** license. Any modifications you distribute must also include the source code under the same license. See the `COPYING` file for full details.

---

## Contact

For support or to suggest features, please refer to the forum thread or contact:

---

*Happy Mining!*

