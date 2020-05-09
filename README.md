[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

# BlackParrot in LiteX

This repository presents necessary steps to run Linux on FPGA and simulation level using BP core integrated into LiteX.
> **Note:** Tested on Ubuntu 18.04

## FPGA Demo

https://www.youtube.com/watch?v=npeDkfEMsoI&feature=youtu.be


## Prerequisites

```
$ sudo apt install build-essential device-tree-compiler wget git python3-setuptools libevent-dev libjson-c-dev
$ sudo apt install verilator # for simulation
$ git clone https://github.com/enjoy-digital/linux-on-litex-blackparrot

```
## Installing LiteX

```
$ wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
$ chmod +x litex_setup.py
$ ./litex_setup.py init
$ sudo ./litex_setup.py install
```

## Installing RISC-V toolchain
```
$ sudo ./litex_setup.py gcc 
```
Do not forget to add RISC-V toolchain binary path to your PATH.

 
## Set necessary environment variables for BlackParrot

Add the following lines to your bashrc to set up BlackParrot environment variables

```
pushd .
cd  PATH/TO/LITEX/litex/soc/cores/cpu/blackparrot
source ./setEnvironment.sh
popd
```

## Pre-built Bitstream and BBL
Pre-built bistream for Genesys Kintex 2 and pre-built Berkeley boot loader (bbl) can be found in the prebuilt folder.

## Running Linux 


### Simulation
First modify $LITEX/litex/litex_sim.py by replacing soc.add_constant("ROM_BOOT_ADDRESS", 0x40000000) with soc.add_constant("ROM_BOOT_ADDRESS", 0x80000000)

Next, launch simulation.
```
$ cd linux-on-litex-blackparrot
$ $LITEX/litex/litex_sim.py --cpu-type blackparrot --cpu-variant standard --integrated-rom-size 40960 --output-dir $PWD/build/BP_linux_simu/ --ram-init prebuilt/simulation/Genesys2/bbl

```

### FPGA
Generate the bitstream 'top.bit' under build/BP_trial/gateware folder
```
$LITEX/litex/boards/genesys2.py --cpu-type blackparrot --cpu-variant standard --output-dir $PWD/build/BP_Trial --integrated-rom-size 51200 --build  
```
In another terminal, launch LiteX terminal.
```
cd linux-on-litex-blackparrot
sudo $LITEX/litex/tools/litex_term.py /dev/ttyUSBX --images images.json --no-crc
```
Load the FPGA bitstream top.bit to your FPGA (you can use vivado hardware manager)

This step will boot up LinuX after copying bbl to DRAM through UART. The whole process will take roughly 15 minutes. 

## Generating the BBL (optional TODO) 

```sh
$ git clone freedom-sdk
$ cd freedom-sdk
$ make bbl
```
The BBL is located in *work/riscv-pk/*. #todo double check


