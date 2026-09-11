---
layout: manual
title: 'Lab 8: Design of UART Transmitter (Verilog)'
session: 'Week 9 (Oct 19 – Oct 23)'
report_due: 'Week 10 (Oct 26 – Oct 30)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab08/lab08_code.tar.gz
---

# 1. Objectives
- Complete RTL design of an UART transmitter in Verilog.
- Simulate the RTL design.
- Synthesize the RTL design and generate gate-level design.
- Simulate the gate-level design.

# 2. Introduction of UART Transmitter

## 2.1 UART Transmitter Controller
The UART transmitter controller (psuedocode shown below) sends signal to datapath and handles state transitions.

```
state IDLE:
    if load_tdr_i asserted:
      assert tdr_load_o
      goto IDLE
    else:
      if load_tsr_i asserted:
        assert tsr_load_o
        goto WAIT
      else:
        goto IDLE

state WAIT:
  if tx_start_i asserted:
    assert tsr_set_start_bit_o
    goto SEND
  else:
    goto WAIT

state SEND:
  if tx_busy_i asserted:
    assert tsr_shift_bit_o
    goto SEND
  else:
    assert reset_o
    goto IDLE
```


## 2.2 UART Transmitter Datapath
The UART transmitter datapath (psuedocode shown below) performes various operations based on the signals received from the controller.

```
if tdr_load_i asserted:
  tdr <= data_bus

else if tsr_load_i asserted:
  tsr <= {tdr[7:0], 1'b1}

else if tsr_set_start_bit asserted:
  tsr[0] <= 0

else if tsr_shift_bit asserted:
  tsr <= {1'b1, tsr[8:1]}
  bit_counter <= bit_counter + 1

else if reset asserted:
  tsr <= 9'b111111111
  bit_counter <= 0
```


## 2.3 UART Transmitter Timing Diagram
The expected behavior of the UART transmitter when transmitting `0xA7` is shown in figure 1.

![Waveform of UART Transmission]({{ "/assets/files/lab08/img/1.png" | relative_url }})

*Figure 1. Waveform of UART Transmission*


## 2.4 Implementation Requirements
- Implement UART transimitter controller module in `uart_tx_controller.v`.
- Implement UART transmitter datapath modeule in `uart_tx_datapath.v`.
- Implement UART transmitter top module in `uart_tx.v`


# 3. Lab Procedures

## 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `mkdir -p $HOME/ecen468/lab08/`
  - `cd $HOME/ecen468/lab08/`

2. Download `lab08_code.tar.gz` from the lab website and put it the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab08_code.tar.gz`
  - `rm lab08_code.tar.gz`

4. Confirm that the following directories and files exist in the working directory.
    - `lib` (directory for technology libraries)
        - `osu018_stdcells.db`
        - `osu018_stdcells.v`
        - `generic.sdb`
    - `rtl` (directory for RTL verilog code)
        - `uart_tx_controller.v`
        - `uart_tx_datapath.v`
        - `uart_tx.v`
    - `sim` (directory where Synopsys VCS will be run)
    - `syn` (directory where Synopsys Design Vision will be run)
        - `netlist` (directory for gate-level verilog code)
        - `sdf` (directory for SDF files)
    - `tb` (directory for testbenches)
        - `uart_tx_netlist_tb.v`
        - `uart_tx_tb.v`

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`


## 3.1 Simulating UART Transmitter RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab08/sim`
    - `vcs -full64 ../tb/uart_tx_tb.v -o simv_uart_tx_tb`

2. Execute the following command to run the simulation.
    - `./simv_uart_tx_tb`

3. Execute the following commands in sequence to open Synopsys WaveView.
    - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
    - `wv &`

4. In WaveView, open `uart_tx_tb.dump` to view the simulation waveform.

5. Add the following signals
    - `uart_tx_tb/dut/uart_tx_datapath_inst/bit_counter`
    - `uart_tx_tb/dut/uart_tx_datapath_inst/tdr`
    - `uart_tx_tb/dut/uart_tx_datapath_inst/tsr`
    - `uart_tx_tb/dut/uart_tx_datapath_inst/txd_o`

6. Take a screenshot of the waveforms for the lab report.

7. Exit WaveView.


## 3.2 Synthesizing UART Transmitter RTL Design Using Synopsys Design Vision
1. Execute the following commands in sequence to open Design Vision:
    - `source /opt/coe/synopsys/syn/V-2023.12-SP1/setup.syn.sh`
    - `cd $HOME/ecen468/lab08/syn`
    - `design_vision &`

2. Execute the following commands to set up Design Vision.
    - `set_app_var link_path ../lib/osu018_stdcells.db`
    - `set_app_var target_library ../lib/osu018_stdcells.db`
    - `set_app_var symbol_library ../lib/osu018_stdcells.db`

3. Execute the following command to analyze the design.
    - `analyze -format verilog {../rtl/uart_tx.v}`

4. Execute the following command to elaborate the design.
    - `elaborate uart_tx`

5. Execute the following command to synthesize the design.
    - `compile -exact_map`

6. Execute the following command to save the optimized netlist.
    - `write -hierarchy -format verilog -output ./netlist/uart_tx.v`

7. Execute the following command to save the Standard Delay Format (SDF) file.
    - `write_sdf ./sdf/uart_tx.sdf`

8. Exite Design Vision.


## 3.3 Simulating UART Transmitter Gate-Level Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab08/sim`
    - `vcs -full64 ../tb/uart_tx_netlist_tb.v -o simv_uart_tx_netlist_tb`

2. Execute the following command to run the simulation.
    - `./simv_uart_tx_netlist_tb`

3. Execute the following commands in sequence to open Synopsys WaveView.
    - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
    - `wv &`

4. In WaveView, open `uart_tx_netlist_tb.dump` to view the simulation waveform.

5. Add the following signals
    - `uart_tx_netlist_tb/dut/uart_tx_datapath_inst/bit_counter`
    - `uart_tx_netlist_tb/dut/uart_tx_datapath_inst/tdr`
    - `uart_tx_netlist_tb/dut/uart_tx_datapath_inst/tsr`
    - `uart_tx_netlist_tb/dut/uart_tx_datapath_inst/txd_o`

6. Take a screenshot of the waveforms for the lab report.

7. Exit WaveView.


# 4. Submission
Please submit a single PDF file containing the following:

UART transmitter RTL design:
1. Screenshot of the waveforms of `uart_tx_tb`.
2. Justification of the simulation results.
3. Screenshots or copy of the content of the following files:
  - `uart_tx.v`
  - `uart_tx_controller.v`
  - `uart_tx_datapath.v`.

UART transmitter gate-level design:
1. Screenshot of the waveforms of `uart_tx_netlist_tb`.
2. Justification of the simulation results.
