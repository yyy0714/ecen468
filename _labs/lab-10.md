---
layout: manual
title: 'Lab 10: Design of Canny Edge Detector (Verilog)'
session: 'Week 11 (Nov 2 – Nov 6)'
report_due: 'Week 12 (Nov 9 – Nov 13)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab10/lab10_code.tar.gz
---

# 1. Objectives
- Complete RTL design of a canny edge detector in Verilog.
- Simulate the RTL design.
- Synthesize the RTL design and generate gate-level design.
- Simulate the gate-level design.

---

# 2. Introduction

## 2.1 Overview

---

# 3. Lab Procedure

## 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `mkdir -p $HOME/ecen468/lab10/`
  - `cd $HOME/ecen468/lab10/`

2. Download `lab10_code.tar.gz` from the lab website and put it the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab10_code.tar.gz`
  - `rm lab10_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `lib` (directory for technology libraries)
        - `osu018_stdcells.db`
        - `osu018_stdcells.v`
        - `generic.sdb`
    - `rtl` (directory for RTL verilog code)
        - `canny_edge_detector.v`
        - `system_bus_canny_edge_detector_wrapper.v`
    - `sim` (directory where Synopsys VCS will be run)
    - `syn` (directory where Synopsys Design Vision will be run)
        - `netlist` (directory for gate-level verilog code)
        - `sdf` (directory for SDF files)
    - `tb` (directory for testbenches)
        - `canny_edge_detector_tb.v`

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

## 3.1 Simulating System Bus RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab10/sim`
    - `vcs -full64 ../tb/canny_edge_detector_tb.v -o simv_canny_edge_detector_tb`

2. Execute the following command to run the simulation.
    - `./simv_canny_edge_detector_tb`

3. Take a screenshot of the terminal outputs of the simulation.

## 3.2 [Optional] Synthesizing UART Transmitter RTL Design Using Synopsys Design Vision
1. Execute the following commands in sequence to open Design Vision:
    - `source /opt/coe/synopsys/syn/V-2023.12-SP1/setup.syn.sh`
    - `cd $HOME/ecen468/lab10/syn`
    - `design_vision &`

2. Execute the following commands to set up Design Vision.
    - `set_app_var link_path ../lib/osu018_stdcells.db`
    - `set_app_var target_library ../lib/osu018_stdcells.db`
    - `set_app_var symbol_library ../lib/osu018_stdcells.db`

3. Execute the following command to analyze the design.
    - `analyze -format verilog {../rtl/canny_edge_detector.v}`

4. Execute the following command to elaborate the design.
    - `elaborate canny_edge_detector`

5. Execute the following command to synthesize the design.
    - `compile -exact_map`

6. Execute the following command to save the optimized netlist.
    - `write -hierarchy -format verilog -output ./netlist/canny_edge_detector.v`

7. Execute the following command to save the Standard Delay Format (SDF) file.
    - `write_sdf ./sdf/canny_edge_detector.sdf`

8. Exit Design Vision.

## 3.3 [Optional] Simulating UART Transmitter Gate-Level Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab09/sim`
    - `vcs -full64 ../tb/top_netlist_tb.v -o simv_top_netlist_tb`

2. Execute the following command to run the simulation.
    - `./simv_top_netlist_tb`

3. If you get the inout port connection width mismatch error for the `addr_bus_io` ports of the `system_bus_sram_wrapper` and `system_bus_uart_tx` modules, do the following:
    - Change `tri [19:0] addr_bus_io` to `tri [31:0] addr_bus_io`
    - Change `tri [31:28] addr_bus_io` to `tri [31:0] addr_bus_io`

3. Take a screenshot of the terminal outputs of the simulation.

---

# 4. Submission
Please submit a single PDF file containing the following:

Top module RTL design:
1. Screenshot of the terminal output after running `./simv_top_tb`.
2. Justification of the simulation results.
3. Screenshots or copy of the content of the following files:
    - `system_bus_uart_tx_wrapper.v`
    - `system_bus_sram_wrapper.v`
