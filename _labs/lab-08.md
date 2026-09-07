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

In this lab, we will design the transmitter of a Universal Asynchronous Receiver/Transmitter (UART) using Verilog, and use Design Vision for synthesis. This module will later be attached to our complete system to transmit data to other devices or processors.

# 2. Introduction of UART

A UART is a piece of computer hardware that translates data between parallel and serial forms. UARTs are commonly used with communication standards such as EIA RS-232, RS-422, and RS-485. The term *universal* indicates that the data format and transmission speed are configurable. Figure 1 shows communication between processors over a serial channel. These processors use parallel data internally for speed, but communicate with one another over a serial channel to reduce the number of wires, and therefore the hardware cost.

![Figure 1. Communication over a serial channel]({{ "/assets/files/lab08/img/1.png" | relative_url }})

*Figure 1. Communication over a serial channel*

Here, a UART transmits 8-bit data without a parity bit. For transmission, the modem wraps the 8-bit word with a start-bit at the least significant bit (LSB) and a stop-bit at the most significant bit (MSB), producing the 10-bit word format shown in Figure 2. The first nine bits of the word are transmitted in sequence, beginning with the start-bit, and each bit is asserted on the serial line for one cycle (bit-time) of the modem clock. The stop-bit may be asserted for more than one clock.

![Figure 2. Data format for UART transmission]({{ "/assets/files/lab08/img/2.png" | relative_url }})

*Figure 2. Data format for UART transmission*

The simplified architecture of a UART is shown in Figure 3, including the signals a host processor uses to control the UART and to move data to and from the data bus in the host machine.

![Figure 3. Block diagram of the UART]({{ "/assets/files/lab08/img/3.png" | relative_url }})

*Figure 3. Block diagram of the UART*

The input signals are provided by the host processor, and the output is the serial data stream. The transmitter consists of a control unit, a data register (`XMT_datareg`), a data shift register (`XMT_shftreg`), and a status register (`bit_count`) that counts the transmitted bits.

The controller's inputs are listed below: primary (external) inputs and status inputs from the datapath. Note that the signal `Load_XMT_datareg` could be passed directly to the datapath; instead, we pass it to the control unit and assert `Load_XMT_DR` only when the state is `idle` and the external `Load_XMT_datareg` signal is asserted. The status signal `BC_lt_BCmax` is asserted while bits are being sent, i.e., while `bit_count` < `word_size` + 1.

- `Load_XMT_datareg`: when asserted in state `idle`, this asserts `Load_XMT_DR`, which loads the contents of `Data_Bus` into `XMT_datareg`.
- `Byte_ready`: assertion causes `Load_XMT_shftreg` to assert, which loads the contents of `XMT_datareg` into `XMT_shftreg`.
- `T_byte`: assertion initiates transmission of a byte of data, including the stop, start, and parity bits.
- `BC_lt_BCmax`: indicates the status of the bit counter in the datapath unit.

![Figure 4. Algorithmic State Machine and Datapath Chart (ASMD) for the UART transmitter]({{ "/assets/files/lab08/img/4.png" | relative_url }})

*Figure 4. Algorithmic State Machine and Datapath Chart (ASMD) for the UART transmitter*

The ASMD chart of the state machine controlling the transmitter is shown in Figure 4. The machine has three states: `idle`, `waiting`, and `sending`. When the active-low, synchronous reset signal `rst_b` is asserted, the machine enters `idle`, `bit_count` is cleared, and `XMT_shftreg` is loaded with 1s. In `idle`, if an active edge of `Clock` occurs while the external host asserts `Load_XMT_datareg`, `XMT_datareg` is loaded with the contents of `Data_Bus`. The machine remains in `idle` until `start` is asserted to drop `XMT_shftreg[0]`.

![Figure 5. Waveforms of the 8-bit UART transmitter]({{ "/assets/files/lab08/img/5.png" | relative_url }})

*Figure 5. Waveforms of the 8-bit UART transmitter*

Figure 5 shows an example of the transmission timing. You can check your results against this timing diagram.

# 3. User-Defined Primitives (UDP)

Verilog has built-in primitives such as gates, transmission gates, and switches. For more complex primitives, Verilog provides user-defined primitives (UDPs). Using UDPs, we can model both combinational and sequential logic, and we can include timing information to model complete ASIC library cells. In this lab, we will design a comparison block that generates the signal `BC_lt_BCmax` using a UDP. `BC_lt_BCmax` should be `1` if `bit_count` is less than 9, and `0` otherwise. Put the output port as the first variable.

```verilog
// Example of UDP in Verilog.
// This code shows how the UDP body looks like

primitive UDP_NAME (
    a, // Port a
    b, // Port b
    c, // Port c
);

// In/Out declaration
output a;
input b, c;

// UDP function code here
// A = B | C;

table
    // B C : A
    ? 1 : 1;
    1 ? : 1;
    0 0 : 0;
endtable

endprimitive
```


# 4. Design of UART Transmitter

## 4.1 Simulating UART Transmitter RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to open Synopsys VCS.
  - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
  - `cd $HOME/ecen468/lab08/sim`
  - `vcs -full64 ../tb/uart_tx_tb.v -o simv_uart_tx_tb`

2. Execute the following command to run the simulation.
  - `./simv_uart_tx_tb`

3. Take a screenshot of the terminal output for the lab report.

4. Execute the following commands in sequence to open Synopsys WaveView.
  - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
  - `wv &`

5. In WaveView, open "uart_tx_tb.dump" to view the simulation waveform.

6. Exit WaveView.


## 4.2 Synthesizing UART Transmitter RTL Design Using Synopsys Design Vision
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


## 4.3 Simulating UART Transmitter Gate-Level Design Using Synopsys VCS
Refer to Section Simulating UART Transmitter RTL Design Using Synopsys VCS.


## 5. Submission
Please submit a single PDF file containing the following:

UART transmitter RTL design:
1. Screenshot of the terminal output of `./simv_uart_tx_tb`.
2. Justification of the simulation results.
3. Screenshots or copy of `uart_tx.v`, `control_unit.v` and `datapath_unit.v`.

UART transmitter gate-level design:
1. Screenshot of the terminal output of `./simv_uart_tx_netlist_tb`.
2. Justification of the simulation results.
