[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

# BlackParrot in LiteX

This repository presents necessary steps to run Linux on FPGA and simulation level using BP core integrated into LiteX.

> **Note:** Tested on Ubuntu 20.04

## FPGA Demo

[![https://www.youtube.com/watch?v=npeDkfEMsoI](https://img.youtube.com/vi/npeDkfEMsoI/0.jpg)](https://www.youtube.com/watch?v=npeDkfEMsoI "BP LiteX Linux FPGA")


## Prerequisites

```
$ sudo apt install build-essential device-tree-compiler wget git python3-setuptools libevent-dev libjson-c-dev
$ sudo apt install verilator # for simulation
$ git clone https://github.com/scanakci/linux-on-litex-blackparrot

```
## Installing LiteX

```
$ wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
$ chmod +x litex_setup.py
$ ./litex_setup.py init install --user
```

## Installing RISC-V toolchain
```
$ ./litex_setup.py gcc 
```
Do not forget to add RISC-V toolchain binary path to your PATH.


## Pre-built Bitstream and BBL
Pre-built bistream for the Arty and pre-built Berkeley boot loader (bbl) can be found in the prebuilt folder.

## Running Linux 


### Simulation

Next, launch simulation.
```
$ cd linux-on-litex-blackparrot
$ lxsim --cpu-type blackparrot --cpu-variant standard --with-sdram --sdram-init prebuilt/simulation/boot_simulation.bin

```

### FPGA
Generate the bitstream for the Arty:
```
$ cd litex
$ litex-boards/litex_boards/targets/digilent_arty.py --build --sys-clk-freq 20e6 --cpu-type blackparrot --cpu-variant standard --variant=a7-100 --csr-csv "csr-arty.csv"
```

Load the FPGA bitstream to the Arty:
```
$ cd litex
$ litex-boards/litex_boards/targets/digilent_arty.py --load --sys-clk-freq 20e6 --cpu-type blackparrot --cpu-variant standard --variant=a7-100 --csr-csv "csr-arty.csv"
```
Alternatively you can can find bitfile `digilent_arty.bit` in `build/gateware` and upload it using vivado hardware manager.

In another terminal, launch LiteX terminal.
```
$ cd linux-on-litex-blackparrot
$ lxterm /dev/ttyUSBX --kernel prebuilt/fpga/Arty/boot_digilent_arty.bin --kernel-adr 0x80000000 --speed=115200
```

If the memory test fails you might need to adjust the [DRAM CL](https://github.com/enjoy-digital/litex/issues/933#issuecomment-873638621).

This step will boot up LinuX after copying bbl to DRAM through UART. The whole process will take roughly 20 minutes. You can login with username `root` and password `blackparrot`.



## Generating the BBL manually 
If you need to generate a BBL from scratch, please follow these steps.

Make sure to adjust the memory capacity in the [device_litex.dts](https://github.com/developandplay/riscv-pk/blob/f18ec2bcccb4273b06f22b2813912933b959ae1d/device_litex.dts#L29) file.
After initial generation if you want to adjust the dts make sure to `rm riscv-pk/machine/device.dtb` first.

Additionally adjust the location of the [UART CSR](https://github.com/developandplay/riscv-pk/blob/f18ec2bcccb4273b06f22b2813912933b959ae1d/machine/uart_lr.c#L9) to match the output of `csr-arty.csv`.

```sh
$ git clone https://github.com/developandplay/freedom-u-sdk.git
$ cd freedom-sdk
$ git checkout blackparrot_mods
$ git submodule update --init --recursive
$ make bbl LITEX_MODE=-DLITEX_MODE //The BBL is located in work/riscv-pk/
$ riscv64-unknown-elf-objcopy -O binary work/riscv-pk/bbl boot.bin // final bbl that needs to be loaded in DRAM
```


