---
title: Quick start
image: /assets/images/arty.png
actions:
- label: "Download"
  icon: download
  url: https://github.com/bonfireprocessor/bonfire/releases
---

## Prerequisites
### Install Git
```
sudo apt install git
```
### Install Python 3.7
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.7
sudo apt install python3-pip
sudo apt install python3-setuptools
```

### Install FuseSoC
```
sudo pip3 install --upgrade fusesoc ``
fusesoc --version
fusesoc init
fusesoc list-cores
```
### Install RISCV GCC Toolchain
```
cd ~/
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
cd riscv-gnu-toolchain
mkdir build
cd build
../configure --prefix=/opt/riscv32 --with-arch=rv32im --with-abi=ilp32
sudo make
export TARGET_PREFIX=/opt/riscv32/bin/riscv32-unknown-elf
```
### Install Xilinx Vivado
```
chmod +x ./Xilinx_Unified_2019.2_1106_2127_Lin64.bin
./Xilinx_Unified_2019.2_1106_2127_Lin64.bin
/tools/Xilinx/Downloads/2019.2/xsetup
```
Install the Vivado HL WebPACK. For a more detailed guide follow Digilent's [Installing Vivado, Xilinx SDK, and Digilent Board Files](https://reference.digilentinc.com/vivado/installing-vivado/start).
### Install Serial Drivers
```
sudo /tools/Xilinx/Vivado/2019.2/data/xicom/cable_drivers/lin64/install_script/install_drivers/install_drivers
sudo adduser $USER dialout
```
### Install Digilent Board Files
```
mkdir ~/vivado
cd ~/vivado
wget https://github.com/Digilent/vivado-boards/archive/master.zip
sudo cp -ar ~/vivado/vivado-boards-master/new/board_files/* /tools/Xilinx/Vivado/2019.2/data/boards/board_files/
```
### Build Bonfire Arty A7 Bitstream
```
mkdir ~/vivado
cd ~/vivado
git clone https://github.com/bonfireprocessor/bonfire.git
cd bonfire
git submodule update --init --recursive
cd bonfire_arty_a7_full
git checkout vivado2019.2
cd ..
fusesoc --config fusesoc.conf build --no-export --setup bonfire-arty-a7
cd ~/vivado/fusesoc_build/vivado2018.2.3/bonfire-arty-a7_0/bld-vivado/
source /tools/Xilinx/Vivado/2019.2/settings64.sh
make all
```
You can also build in the Vivado GUI by running `make build-gui` after `make all`.

### Pre-built Bitstream Quick Start Guide

If you own a Digilent [Arty A35 Board](https://store.digilentinc.com/arty-a7-artix-7-fpga-development-board-for-makers-and-hobbyists/) a prebuild mcs File is available, which contains the Bonfire Implementation for Arty Board including a simple Boot Monitor and a ready to run eLua image.

#### Installation Instructions
1.  Download the mcs.zip file from [here](https://github.com/bonfireprocessor/bonfire/releases)
1. Unzip it
1. Connect you Arty Board to a USB connector (make sure that Vivado and the Xilinx cable drives are correctly installed)
1. Start Vivado (or the Vivado Labtools)
1. Open the Vivado Hardware Manager (it is available in the qick start pane)
  ![install01](/assets/images/install01.png)

1. Connect to the target board (the Arty, it is named xc7a35t_0 in the HW Manager screen)
  ![install02](/assets/images/install02.png)
1.  In the Hardware Manager window, under hardware right click your device and click Add Configuration Memory Devices
1. A window will pop up. Search for "Micron" and select **mt25ql128-spi-x1_x2_x4** (**NOTE**: Newer Arty boards may use Spansion **s25fl128sxxxxxx0-spi-x1_x2_x4** instead). Click OK on the next window asking if you want to program the configuration memory device.
  ![install03](/assets/images/install03.png)
2. In the following dialog box select the unpzipped .mcs file from step 1 as "Confiugration file".
  ![install04](/assets/images/install04.png)
1. Start programming

If you get an error message "Flash Programming Unsuccessful. Part selected mt25ql128, but part s25fl128sxxxxxx0 detected." It means your Arty board uses a Spansion **s25fl128sxxxxxx0-spi-x1_x2_x4 instead** instead.

#### Connect with a terminal program to the Board

The prebuild mcs image communicates with the usb-uart of the Arty Board with 500.000 Baud 8N1, without any handshaking.

The recommended tool for communication is minicom. In most cases the Arty UART is available under /dev/ttyUSB1, but it can be different.
#### Start minicom

`$ sudo minicom -D /dev/ttyUSB1 -b 500000`

#### Turn off Flow Control

1. Press CTRL-A Z
2. Select O (cOnfigure Minicom)
3. Select "Serial port setup"
4. Use F and G to set:
   * F - Hardware Flow Control : No
   * G - Software Flow Control : No
5. Press <escape> until you are back to the main prompt

When pressing the 'Prog' button on Arty the monitor boot message should appear. The is a prompt, when you enter 'r' followed by <return> the eLua Image is loaded from Flash into memory and started.
The screen should look like as follows:

```
Bonfire Boot Monitor 0.3d (GCC 7.2.0)
MIMPID: 00010014
MISA: 40001100
UART Divisor: 165
Uptime 0 sec
DRAM Size 268435456 bytes
Cache size: 8192 bytes
Line size: 32 bytes
Cache lines: 256
Framing errors: 0
SPI Flash JEDEC ID: 0018ba20

>r
Reading Header
...OK
Boot Image found, length 462848 Bytes, Break Address: 00081f44
...OK
Cache 32768 bytes Flush read from 0fff8000
Heap: 000830b0 .. 0ffef7ff
eLua for Bonfire SoC 1.1
Build with GCC 7.2.0
Initalizing Ethernet core
__virt_timer_period 1666666

eLua v0.9-388-g7e7db3b  Copyright (C) 2007-2013 www.eluaproject.net
eLua#

```
Enter `help` to get a list of commands
```
eLua# help
Shell commands:
  help   - shell help
  lua    - start a Lua session
  ls     - lists files and directories
  dir    - lists files and directories
  cat    - list the contents of a file
  type   - list the contents of a file
  recv   - receive files via XMODEM
  cp     - copy files
  mv     - move/rename files
  rm     - remove files
  ver    - show version information
  mkdir  - create directories
  exit   - exit the shell
  edit   - edit a file
For more information use 'help <command>'.
eLua#
```
Enter
`lua /rom/life.lua`

to run the Game of Life demo programs
![Life screenshot](/assets/images/life_demo.png)

#### Integrated Text Editor

````
edit /rom/life_server.lua
````
![Editor screenshot](/assets/images/kilo.png)

The editor supports syntax highlighting for Lua code, you need to enable ANSI colors in your terminal program to see it. For minicom use the option `` --color=on`` to enable it.

Of course on the /rom filesystem saving the file is not possible. So the editor is more useful if a Micro SD card is connected to the system. 

For more information about eLua and the Lua Language go to the [eLua Project Page](http://www.eluaproject.net/) and to the [Lua Language Page](https://www.lua.org/)
## Uploading eLua
### Build eLua
```
sudo apt-get install luarocks
sudo luarocks install luafilesystem md5 lpack bit32
cd ~/
mkdir ~/upload
git clone https://github.com/ThomasHornschuh/elua
cd elua
make all BOARD=bonfire_arty
```
### Run minicom

`sudo minicom -D /dev/ttyUSB1 -b 500000`

`>x`

The `x` command will download the binary to location 0x10000 in memory.
### Upload the eLua binary
1. Press CTRL-A Z
2. Select S (Send files)
3. Select **xmodem**
4. Browse to /home/*username*/upload:
   * Double press <space> to enter a folder
   * Press <space> to select **elua_lua_bonfire_arty.bin**
5. Press <return> to send the file

Once the file is uploaded use the following command to run the image:

`>g`

You can also use the following command to write the binary to flash memory:

`>w`

**NOTE**: Never use the `w` after the `g` command without performing a new xmodem upload.
## Uploading a custom bootloader
### Build monitor
```
cd ~/vivado/bonfire/bonfire-software/monitor
make all PLATFORM=ARTY_AXI
cp ./ARTY_AXI_monitor.hex ~/vivado/bonfire/bonfire_arty_a7_full/compiled_code/
cp ./ARTY_AXI_monitor.bin ~/upload/
```
### Run minicom

`sudo minicom -D /dev/ttyUSB1 -b 500000`

`>x0`

The `x0` command will download the binary to location 0x00000 in memory.
### Upload the bootloader binary
1. Press CTRL-A Z
2. Select S (Send files)
3. Select **xmodem**
4. Browse to /home/*username*/upload:
   * Double press <space> to enter a folder
   * Press <space> to select **ARTY_AXI_montior.bin**
5. Press <return> to send the file

Once the file is uploaded use the following command to run the image.

`>g0`

## Create a Custom MCS File
1. Run Vivado
2. Select Tools->Generate Memory Configuration File...
3. Set the following properties:
* Memory Part: **mt25ql128-spi-x1_x2_x4** or **s25fl128sxxxxxx0-spi-x1_x2_x4**
* Format: **MCS**
* Interface: **SPIx4**
* Load bitstream files: **true**
* Start address: **00000000**
  * Direction: **up**
  * Bitfile: ~/vivado/fusesoc_build/vivado2018.2.3/bonfire-arty-a7_0/bld-vivado/bonfire-arty-a7_0.bit
* Load data files: **true**
  * Start address: **00300000**
  * Direction: **up**
  * Datafile: ~/elua/elua_lua_bonfire_arty.img
* Write checksum: **false**
* Disable bit swapping: **false**
* Overwrite: **false**
