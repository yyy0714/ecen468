---
layout: manual
title: 'Lab 11: Combining Canny Edge Detector with System Bus, Memory and UART (Verilog)'
session: 'Week 12 (Nov 9 – Nov 13)'
report_due: 'Week 13 (Nov 16 – Nov 20)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab11/lab11_code.tar.gz
---

## 1. Objectives
- Complete RTL design integrating the system bus, SRAM, UART transmitter and Canny edge detector.
- Simulate the RTL design.

---

## 2. Implementation

### 2.1 Device IDs
The ID of the devices are as follows:

| Item                | ID   |
| ------------------- | ---- |
| SRAM                | 0001 |
| UART Transmitter    | 0010 |
| Testbench           | 0011 |
| Canny Edge Detector | 0100 |

### 2.2 Device Priorities
The priorities of the devices are as follows:

| Item                | Priority |
| ------------------- | -------- |
| Testbench           | Highest  |
| SRAM                |          |
| UART Transmitter    |          |
| Canny Edge Detector | Lowest   |

### 2.3 Canny Edge Detector Address Map
Figure 1 shows the address map of Canny edge detector.

![Canny Edge Detector Address Map]({{ "/assets/files/lab11/img/canny_edge_detector_addr_map.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 1. Canny Edge Detector Address Map*

### 2.4 Requirement
- Implement system bus Canny edge detector wrapper in `system_bus_canny_edge_detector_wrapper.v`.

---

## 3. Lab Procedure

### 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `mkdir -p $HOME/ecen468/lab11/`
  - `cd $HOME/ecen468/lab11/`

2. Download `lab11_code.tar.gz` from the lab website and put it the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab11_code.tar.gz`
  - `rm lab11_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `images` (directory for images)
        - `generated` (directory for generated images)
        - `reference` (directory for reference images)
            - `img_1.bmp`
            - `img_2.bmp`
            - `img_3.bmp`
            - `img_4.bmp`
            - `img_5.bmp`
    - `lib` (directory for technology libraries)
        - `osu018_stdcells.db`
        - `osu018_stdcells.v`
        - `generic.sdb`
    - `rtl` (directory for RTL verilog code)
        - `sram.v`
        - `system_bus_arbiter.v`
        - `system_bus_canny_edge_detector_wrapper.v`
        - `top.v`
    - `sim` (directory where Synopsys VCS will be run)
    - `syn` (directory where Synopsys Design Vision will be run)
        - `netlist` (directory for gate-level verilog code)
        - `sdf` (directory for SDF files)
    - `tb` (directory for testbenches)
        - `top_tb.v`

5. Copy the following files from lab 8 into the `rtl` directory:
    - `uart_tx_controller.v`
    - `uart_tx_datapath.v`
    - `uart_tx.v`

6. Copy the following files from lab 9 into the `rtl` directory:
    - `system_bus_sram_wrapper.v`
    - `system_bus_uart_tx_wrapper.v`

7. Copy the following files from lab 10 into the `rtl` directory:
    - `canny_edge_detector.v`

8. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

### 4.1 Simulating Canny Edge Detector RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab11/sim`
    - `vcs -full64 ../tb/top_tb.v -o simv_top_tb`

2. Execute the following command to run the simulation.
    - `./simv_top_tb`

4. Five BMP images should be generated in `../images/generated`.

### 4.2 Calculate the Match Ratio of the Generated Images
1. Execute the following commands to create a Python virtual environment and install the necessary packages.
    - `cd $HOME/ecen468/lab11/images`
    - `python3.12 -m venv .venv`
    - `source .venv/bin/activate`
    - `pip install --upgrade pip`
    - `pip install pillow numpy`

2. Execute the following command to run the Python script to calculate the match ratio.
    - `python calculate_match_ratio.py`

3. Take a screenshot of the terminal output of the script. Match ratio no less than 98% is required for a full score.   

---

## 5. Submission
Please submit a single PDF file containing the following:

- Canny edge detector RTL design:
    1. Screenshot of the terminal output after running `python calculate_match_ratio.py`.
    2. Screenshots or copy of the content of the following file:
        - `system_bus_canny_edge_detector_wrapper.v`
