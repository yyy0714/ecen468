---
layout: manual
title: 'Lab 9: Design of System Bus (Verilog)'
session: 'Week 10 (Oct 26 – Oct 30)'
report_due: 'Week 11 (Nov 2 – Nov 6)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab09/lab09_code.tar.gz
---

# 1. Objectives
- Complete RTL design of a system bus in Verilog.
- Simulate the RTL design.
- Synthesize the RTL design and generate gate-level design.
- Simulate the gate-level design.


# 2. Introduction of System Bus

## 2.1 Overview

## 2.2 System Bus Arbiter

## 2.3 System Bus SRAM Wrapper and UART Transmitter Wrapper

## 2.4 Implementation Requirements
- Implement system bus arbiter in `system_bus_arbiter.v`.
- Implement system bus SRAM wrapper in `system_bus_sram_wrapper.v`.
- Implement system bus UART transmitter wrapper in `system_bus_uart_tx_wrapper.v`.
- Implement system bus top module in `system_bus_top.v`.

# 3. Lab Procedures

## 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `mkdir $HOME/ecen468/lab09/`
  - `cd $HOME/ecen468/lab09/`

2. Download `lab09_code.tar.gz` from the lab website and put it the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab09_code.tar.gz`
  - `rm lab09_code.tar.gz`

4. Confirm that the following directories and files exist in the working directory.
    - `lib` (directory for technology libraries)
        - `osu018_stdcells.db`
        - `osu018_stdcells.v`
        - `generic.sdb`
    - `rtl` (directory for RTL verilog code)
        - `_sram.v`
        - `arbiter.v`
        - `sram_wrapper.v`
        - `top.v`
        - `uart_tx_wrapper.v`
    - `sim` (directory where Synopsys VCS will be run)
    - `syn` (directory where Synopsys Design Vision will be run)
        - `netlist` (directory for gate-level verilog code)
        - `sdf` (directory for SDF files)
    - `tb` (directory for testbenches)
        - `top_netlist_tb.v`
        - `top_tb.v`

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`


## 3.1 Simulating UART Transmitter RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab09/sim`
    - `vcs -full64 ../tb/top_tb.v -o simv_top_tb`

2. Execute the following command to run the simulation.
    - `./simv_top_tb`

3. Execute the following commands in sequence to open Synopsys WaveView.
    - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
    - `wv &`

4. In WaveView, open `top_tb.dump` to view the simulation waveform.

5. Add the following signals

6. Take a screenshot of the waveforms for the lab report.

7. Exit WaveView.


## Submission

Please submit a single PDF file containing the following:

`top` RTL design:
1. Screenshots of the terminal output after running `./simv`.
2. Justification of the correctness of the results.
3. Screenshots or copy of `arbiter.v`, `uart_tx_wrapper.v`, `sram_wrapper.v` and `top.v`.

`top` gate-level design:
1. Screenshots of the simulation output after running `./simv`.
2. Justification of the correctness of the results.
