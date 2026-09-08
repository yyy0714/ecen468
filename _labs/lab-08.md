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

# 2. Introduction of UART
Below is the psuedocode of UART transmitter that your code should base on:
```
state IDLE:
  if rst_n == 0:
    bit_counter <= 0
    tsr <= 9'b111111111
    goto IDLE
  else:
    if load_tdr:
      tdr <= data_bus
      goto IDLE
    else:
      if load_tsr:
        tsr <= {tdr[7:0], 1'b1}
        goto WAIT
      else:
        goto IDLE

state WAIT:
  if tsr_get_ready == 1:
    tsr[0] <= 0
    goto SEND
  else:
    goto WAIT

state SEND:
  if bit_counter < 9:
    txd <= tsr[0]
    tsr <= {1'b1, tsr[8:1]}
    bit_counter <= bit_counter + 1
    goto SEND
  else:
    bit_counter <= 0
    tsr <= 9'b111111111
    goto IDLE
```

Figure 1 shows the expected behavior of the UART transmitter when transmitting `0xA7`.

![Waveform of UART Transmission]({{ "/assets/files/lab08/img/1.png" | relative_url }})

*Figure 1. Waveform of UART Transmission*


The UART transmitter has 6 operations as shown in the table below.

| Operation                      | Condition                           | Effects                                  |
| ------------------------------ | ----------------------------------- | ---------------------------------------- |
| Load byte to TDR from data bus | `tdr_load` is asserted              | `tdr <= data_bus`                        |
| Load byte to TSR from TDR      | `tsr_load` is asserted              | `tsr <= {tdr[7:0], 1'b1}`                |
| Set start bit in TSR           | `tsr_set_start_bit` is asserted     | `tsr[0] <= 0`                            |
| Shift one bit out of TSR       | `tsr_shift_bit` is asserted         | `tsr <= {1'b1, tsr[8:1]}`                |
| Increment bit counter          | `bit_counter_increment` is asserted | `bit_counter <= bit_counter + 1`         |
| Reset                          | `reset` is asserted                 | `tsr <= 8'b11111111`, `bit_counter <= 0` |

1. Load byte to TDR from data bus
  - when `tdr_load` is asserted
  - `tdr <= data_bus`
  - IDLE -> IDLE
2. Load byte to TSR from TDR
  - when `tsr_load` is asserted
  - `tsr <= {tdr[7:0], 1'b1}`
  - IDLE -> WAIT
3. Set start bit in TSR
  - when `tsr_set_start_bit` is asserted
  - `tsr[0] <= 0`
  -  WAIT -> SEND
4. Shift one bit out of TSR
  - when `tsr_shift_bit` is asserted
  - `tsr <= {1'b1, tsr[8:1]}`
  - SEND -> SEND
5. Increment bit counter
  - when `bit_counter_increment` is asserted
  -`bit_counter <= bit_counter + 1`
  - SEND -> SEND
5. Reset
  - when `reset` is asserted
  - `tsr <= 8'b11111111`
  - `bit_counter <= 0`
  - SEND -> IDLE

You will implement the controller (`uart_tx_controller`) and datapath (`uart_tx_datapath`) of the UART transmitter.


# 3. Design of UART Transmitter

## 3.1 Simulating UART Transmitter RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
  - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
  - `cd $HOME/ecen468/lab08/sim`
  - `vcs -full64 ../tb/uart_tx_tb.v -o simv_uart_tx_tb`

2. Execute the following command to run the simulation.
  - `./simv_uart_tx_tb`

3. Take a screenshot of the terminal output for the lab report.

4. Execute the following commands in sequence to open Synopsys WaveView.
  - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
  - `wv &`

5. In WaveView, open `uart_tx_tb.dump` to view the simulation waveform.

6. Exit WaveView.


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

<!-- Save the optimized Verilog netlist.
- Select **TOP** design (`UART_XMTR`)
- **File -> Save as**
- Enter `UART_gate.v` as the file name and choose **Verilog** as the file type.
- Check the option **Save All Designs in Hierarchy**. -->

You will see an error message stating that the UDP design is not synthesizable.
To fix the error, use the line below to replace the UDP.
`assign BC_lt_BCmax = (bit_count < word_size + 1);`


## 3.3 Simulating UART Transmitter Gate-Level Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
  - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
  - `cd $HOME/ecen468/lab08/sim`
  - `vcs -full64 ../tb/uart_tx_netlist_tb.v -o simv_uart_tx_netlist_tb`

2. Execute the following command to run the simulation.
  - `./simv_uart_tx_netlist_tb`

3. Take a screenshot of the terminal output for the lab report.

4. Execute the following commands in sequence to open Synopsys WaveView.
  - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
  - `wv &`

5. In WaveView, open `uart_tx_netlist_tb.dump` to view the simulation waveform.

6. Exit WaveView.


# 4. Submission
Please submit a single PDF file containing the following:

UART transmitter RTL design:
1. Screenshot of the terminal output of `./simv_uart_tx_tb`.
2. Justification of the simulation results.
3. Screenshots or copy of `uart_tx.v`, `control_unit.v` and `datapath_unit.v`.

UART transmitter gate-level design:
1. Screenshot of the terminal output of `./simv_uart_tx_netlist_tb`.
2. Justification of the simulation results.
