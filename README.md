![NexMon logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/nexmon.png)


# Channel State Information for Raspberry Pi

This project allows you to extract Channel State Information (CSI) of OFDM-modulated
Wi-Fi frames (802.11a/(g)/n/ac) on a per frame basis with up to 80 MHz bandwidth on
several Broadcomm Wi-Fi chips. For a full list, see the [original Nexmon_CSI repository](https://github.com/seemoo-lab/nexmon_csi).

**This fork and branch are for Raspberry Pi 3B+ and 4 variants.**
|                   |                         |
| ----------------- | ----------------------- |
| Device            | Raspberry Pi 3B+ and 4  |
| Raspbian          | [Raspbian Buster Lite 2020-08-20](https://www.raspberrypi.org/downloads/raspberry-pi-os/) |
| Chip              | BCM43455c0 (built-in)   |
| Nexmon_csi Commit | [ba99ce](https://github.com/seemoo-lab/nexmon_csi/commit/ba99ce12a6a42d7e4ec75e6f8ace8f610ed2eb60) |
| Nexmon Commit     | [050d41](https://github.com/seemoo-lab/nexmon/commit/https://github.com/seemoo-lab/nexmon/commit/050d415d33f1f09223f10cd645483e68d8193497) |
| Date              | August 29, 2020|

## Usage
After following the [getting started](#getting-started) guide, you can begin extracting CSI by doing the following.
1. Use _makecsiparams_ to generate a base64 encoded parameter string which is used to configure the extractor.
   This example collects CSI on **channel 157** with **bandwidth 80 MHz** on first core of the WiFi chip, for the first spatial stream.
   Raspberry Pi has only one core, and a single antenna, so the last two options don't need changing.
    ```
    mcp -c 157/80 -C 1 -N 1
    m+IBEQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==
    ```
    _makecsiparams_ supports several other features like filtering data by Mac IDs or by first byte. Run `mcp -h` to see all available options.
2. `ifconfig wlan0 up`
3. `nexutil -Iwlan0 -s500 -b -l34 -vm+IBEQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==`
4. `iw dev wlan0 interface add mon0 type monitor`
5. `ip link set mon0 up`

Collect CSI by listening on socket 5500 for UDP packets. One way to do this is using tcpdump:
`tcpdump -i wlan0 dst port 5500`. You can store 1000 CSI samples in a pcap file like this:
`tcpdump -i wlan0 dst port 5500 -vv -w output.pcap -c 1000`.

## Analyzing the CSI

The pcap file can be opened in Wireshark or parsed with a script. 
There is an example Matlab script in the utils folder, and a Python script is on the way.
Each UDP packet has 10.10.10.10 as source address, and 255.255.255.255 as destination address on port 5500.
CSI data is embedded inside the UDP packet's payload.

Here is the embedded data structure:

Bytes    | Type       | Name                    | Description
---------| ---------- | ----------------------- | --------------------
4        | `uint32`   | Magic Bytes             | 0x11111111
6        | `uint8[6]` | Address 1               | Address 1 - recv
6        | `uint8[6]` | Address 2               | Address 2 - tran
6        | `uint8[6]` | Address 3               | Address 3
2        | `uint16`   | Frame Control           | Frame's FC field
2        | `uint16`   | Sequence Control        | Frame's SC field
1        | `int8`     | RSSI                    | Instantaneous frame RSSI
18       | `uint16[9]`| Reserved                | Reserved for future use
2        | `uint16`   | Core and Spatial Stream | Lowest 3 bytes indicate the Core, and the next three bits indicate the Spatial Stream number. 
2        | `uint16`   | Chanspec                | Chanspec used during extraction. See `nexutil -k`.
2        | `uint16`   | Chip Version            | Chip Version
variable | `int16[]`  | CSI Data                | Each CSI sample is 4 bytes with interleaved Int16 Real and Int16 Imaginary. There are `bandwidth * 3.2` OFDM subcarriers per channel, and a CSI sample for every subcarrier is present.

# Getting Started
### Prepare Raspberry Pi
* Burn [Raspbian Buster Lite 2020-08-20](https://www.raspberrypi.org/downloads/raspberry-pi-os/) onto an empty SD card. You can use [Etcher](https://www.balena.io/etcher/).
* [Create an empty file called `ssh`](https://www.raspberrypi.org/documentation/remote-access/ssh/), without any extension, on the boot partition of the SD card.
* [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/) into the Pi.
* With `sudo raspi-config`, set WiFi Country, Time Zone, and then Expand File System.
* Reboot when asked to.

### Install dependencies
Install dependencies.

* `sudo apt update`
* `sudo apt upgrade`
* `sudo reboot`
* `sudo apt install raspberrypi-kernel-headers git libgmp3-dev gawk qpdf bison flex make automake texinfo libtool-bin tcpdump tmux`
* `sudo reboot`

### Install Nexmon and Nexmon_CSI
* `sudo su`
* `wget https://raw.githubusercontent.com/zeroby0/nexmon_csi/pi-5.4.51-plus/install.sh -O install.sh`
* `tmux new -c /home/pi -s nexmon 'bash install.sh | tee output.log'`

Your installation will happen in this tmux session. The right bottom corner will show the step running. Use `ctrl-b d` to detach, and `tmux attach-session -t nexmon` to re-attach.


# Extract from our License

Any use of the Software which results in an academic publication or
other publication which includes a bibliography must include
citations to the nexmon project a) and the paper cited under b):

a) "Matthias Schulz, Daniel Wegemer and Matthias Hollick. Nexmon:
       The C-based Firmware Patching Framework. https://nexmon.org"

b) "Francesco Gringoli, Matthias Schulz, Jakob Link, and Matthias
       Hollick. [Free Your CSI: A Channel State Information Extraction
       Platform For Modern Wi-Fi Chipsets](https://doi.org/10.1145/3349623.3355477). In Proceedings of the 13th
       Workshop on Wireless Network Testbeds, Experimental evaluation
       & CHaracterization (WiNTECH 2019), October 2019."

Additionally, I would appreciate it if you would cite this repository.

# References

* Matthias Schulz, Daniel Wegemer and Matthias Hollick. **Nexmon: The C-based Firmware Patching 
  Framework**. https://nexmon.org
* Francesco Gringoli, Matthias Schulz, Jakob Link, and Matthias Hollick. **[Free Your CSI: 
  A Channel State Information Extraction Platform For Modern Wi-Fi Chipsets](https://doi.org/10.1145/3349623.3355477)**.
  In Proceedings of the 13th Workshop on Wireless Network Testbeds, Experimental evaluation & CHaracterization (WiNTECH 2019), October 2019. https://doi.org/10.1145/3349623.3355477

[//]: # "[Get references as bibtex file](https://nexmon.org/bib)"

# Contact

* [Francesco Gringoli](http://netweb.ing.unibs.it/~gringoli/) <francesco.gringoli@unibs.it>
* [Matthias Schulz](https://seemoo.tu-darmstadt.de/mschulz) <mschulz@seemoo.tu-darmstadt.de>
* Jakob Link <jlink@seemoo.tu-darmstadt.de>
* [Aravind Reddy V](https://github.com/zeroby0) <aravind.reddy@iiitb.org>

I'm not affiliated with the Seemoo lab.
This software is useful to me and helped me complete my Thesis, so I'm trying to give back to the community.

# Powered By

## Secure Mobile Networking Lab (SEEMOO)
<a href="https://www.seemoo.tu-darmstadt.de">![SEEMOO logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/seemoo.png)</a>
## Multi-Mechanisms Adaptation for the Future Internet (MAKI)
<a href="http://www.maki.tu-darmstadt.de/">![MAKI logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/maki.png)</a>
## LOEWE centre emergenCITY
<a href="https://www.emergencity.de/">![emergenCITY logo](https://www.emergencity.de/assets/img/logo_emergencity.png)</a>
## Technische Universität Darmstadt
<a href="https://www.tu-darmstadt.de/index.en.jsp">![TU Darmstadt logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/tudarmstadt.png)</a>
## University of Brescia
<a href="http://netweb.ing.unibs.it/">![University of Brescia logo](https://github.com/seemoo-lab/nexmon/raw/master/gfx/brescia.png)</a>

# Disclaimer

You are compiling Nexmon and Nexmon_csi projects and patching your original Broadcomm/Cypress firmware.
This may void your warranty and/or damage your hardware.
This software is provided "as is" and without any warranty, and in no event shall the authors be held liable.