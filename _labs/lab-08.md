---
layout: manual
title: 'Lab 8: Design of UART Transmitter (Verilog)'
session: 'Week 9 (Oct 19 – Oct 23)'
report_due: 'Week 10 (Oct 26 – Oct 30)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab08/lab08_code.tar.gz
---

## Objectives

In this lab, we will design the transmitter of a Universal Asynchronous Receiver/Transmitter (UART) using Verilog, and use Design Vision for synthesis. This module will later be attached to our complete system to transmit data to other devices or processors.

## Introduction

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

## User-Defined Primitives (UDP)

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

## Implementation & Simulation

We will now implement the transmitter portion of the UART design.

Please login to the Olympus server and create a working directory for this lab using the following commands.

```bash
## Create and navigate to the working directory.
mkdir -p $HOME/ECEN468/Lab8/src
cd $HOME/ECEN468/Lab8/src
```

Download the tar.gz file from the lab website and extract it. In the extracted folders, you will find the following files:

- `UART_XMTR.v`
- `Control_Unit.v`
- `Datapath_Unit.v`
- `UART_tb.v`
- `generic.sdb`
- `osu018_stdcells.v`
- `osu018_stdcells.db`

Copy them to the working directory. You will write your code in `UART_XMTR.v`, `Control_Unit.v`, and `Datapath_Unit.v`.

Once you complete the implementation, verify your design. First, compile it using the commands below.

Commands for reference:

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh
vcs -full64 UART_tb.v
```

If there are compile errors, recheck your design and fix your code. Once compilation succeeds, go to the next step to view the results.

`./simv`

Take screenshots of the terminal output and include them in your report.

To view the graphical waveform of the simulation results, run the following command to invoke WaveView, and open `wave.dump` in WaveView.

```bash
source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh
wv &
```

The graphical waveform is for debugging. You don't need to include it in the report.

Once verification is complete, we will use Design Vision to synthesize the design. For convenience, invoke Design Vision from a separate directory to isolate the intermediate files it generates.

```bash
## Create and navigate to a separate working directory.
mkdir $HOME/ECEN468/Lab8/src/dv_Work
cd $HOME/ECEN468/Lab8/src/dv_Work

## Launch Design Vision
source /opt/coe/synopsys/syn/V-2023.12-SP1/setup.syn.sh
design_vision &
```

In Design Vision, we first link the libraries.

- Click on **File -> Setup -> Defaults**
- Add `osu018_stdcells.db` to **Link Library** (Remove all other library files)
- Add `osu018_stdcells.db` to **Target Library** (Remove all other library files)
- Add `osu018_stdcells.db` to **Symbol Library** (Remove all other library files)
- Click on **OK**

![Design Vision library setup under File -> Setup -> Defaults]({{ "/assets/files/lab08/img/6.png" | relative_url }})

Next, we *Analyze* the design. During analysis, the tool reads VHDL or Verilog files, checks for proper syntax and synthesizable logic, and stores the design in an intermediate format.

- Click **File -> Analyze**.
- In the pop-up window, add `UART_XMTR.v`, and click **OK**.

![Analyze dialog in Design Vision]({{ "/assets/files/lab08/img/7.png" | relative_url }})

You will see an error message stating that the UDP design is not synthesizable.

To fix the error, use the line below to replace the UDP.

`assign BC_lt_BCmax = (bit_count < word_size + 1);`

Elaborate Design

- Click on **File -> Elaborate**.
- In the pop-up window, type `work` in **Library**, and select `UART_XMTR` in **Design**.
- Click on **OK**.

![Elaborate dialog in Design Vision]({{ "/assets/files/lab08/img/8.png" | relative_url }})

Synthesize the module

Select **Design -> Compile Design**, then click **OK**.

![Compile Design dialog in Design Vision]({{ "/assets/files/lab08/img/9.png" | relative_url }})

Save the optimized Verilog netlist.

- Select **TOP** design (`UART_XMTR`)
- **File -> Save as**
- Enter `UART_gate.v` as the file name and choose **Verilog** as the file type.
- Check the option **Save All Designs in Hierarchy**.

Save the Standard Delay Format (SDF) by running the following command in the command window:

`write_sdf UART.sdf`

You may now close Design Vision.

Gate Simulation

a) Copy the Verilog netlist and SDF file to the work folder.

```bash
cp ./UART_gate.v $HOME/ECEN468/Lab8/src/
cp ./UART.sdf $HOME/ECEN468/Lab8/src/
cd $HOME/ECEN468/Lab8/src/
```

We will use the previous test bench as a template to create a test bench for the gate simulation.

`cp UART_tb.v UART_gate_tb.v`

Open the new test bench `UART_gate_tb.v` and make the following modifications:

- Make sure the following three lines are at the top of the file.

  ```verilog
  `timescale 1ns/10ps
  `include "UART_gate.v"
  `include "osu018_stdcells.v"
  ```

- Remove the following line if it exists.

  ```verilog
  `include "UART_XMTR.v"
  ```

- Make sure the following two lines are at the bottom of the file (within the module).

  ```verilog
  initial
      $sdf_annotate("UART.sdf", UART_XMTR_01);
  ```

- Change the name of the dump file in the following line in the test bench.

  `$dumpfile("wave_gate.dump");`

Make sure all the files listed below are available in your current directory.

- `UART_gate.v`
- `UART.sdf`
- `osu018_stdcells.v`
- `UART_gate_tb.v`

Once we have the files ready, we can run the following command to start the simulation.

```bash
source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh
vcs -full64 UART_gate_tb.v
```

If no error occurs, do the next step.

`./simv`

Take a screenshot of the simulation output and include it in your report.

## Submission

Please submit a single PDF file containing the following:

UART:

1. Screenshots of the simulation output after running `./simv`.
2. Justification of the correctness of the results.
3. Screenshots of the code `UART_XMTR.v`, `Control_Unit.v`, and `Datapath_Unit.v`.

UART Gate:

1. Screenshots of the simulation output after running `./simv`.
2. Justification of the correctness of the results.
