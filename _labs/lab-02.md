---
layout: manual
title: 'Lab 2: Design of UART Transmitter (SystemC)'
session: 'Week 3 (Sep 7 – Sep 11)'
report_due: 'Week 4 (Sep 14 – Sep 18)'
manual_pdf: /assets/files/lab02/lab02_manual.pdf
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab02/lab02_code.tar.gz
---

## Objectives

In this lab, we will design the transmitter of a Universal Asynchronous Receiver/Transmitter (UART) using SystemC. This module will later be attached to our complete system to transmit data to other devices or processors.

## Introduction

A UART is a piece of computer hardware that translates data between parallel and serial forms. UARTs are commonly used with communication standards such as EIA RS-232, RS-422, and RS-485. The term *universal* indicates that the data format and transmission speed are configurable. Figure 1 shows communication between processors over a serial channel. These processors use parallel data internally for speed, but communicate with one another over a serial channel to reduce the number of wires, and therefore the hardware cost.

![Figure 1. Communication over a serial channel]({{ "/assets/files/lab02/img/1.png" | relative_url }})

*Figure 1. Communication over a serial channel*

Here, a UART transmits 8-bit data without a parity bit. For transmission, the modem wraps the 8-bit word with a start-bit at the least significant bit (LSB) and a stop-bit at the most significant bit (MSB), producing the 10-bit word format shown in Figure 2. The first nine bits of the word are transmitted in sequence, beginning with the start-bit, and each bit is asserted on the serial line for one cycle (bit-time) of the modem clock. The stop-bit may be asserted for more than one clock.

![Figure 2. Data format for UART transmission]({{ "/assets/files/lab02/img/2.png" | relative_url }})

*Figure 2. Data format for UART transmission*

The simplified architecture of a UART is shown in Figure 3, including the signals a host processor uses to control the UART and to move data to and from the data bus in the host machine.

![Figure 3. Block diagram of the UART]({{ "/assets/files/lab02/img/3.png" | relative_url }})

*Figure 3. Block diagram of the UART*

The input signals are provided by the host processor, and the output is the serial data stream. The transmitter consists of a control unit, a data register (`XMT_datareg`), a data shift register (`XMT_shftreg`), and a status register (`bit_count`) that counts the transmitted bits.

The controller's inputs are listed below: primary (external) inputs and status inputs from the datapath. Note that the signal `Load_XMT_datareg` could be passed directly to the datapath; instead, we pass it to the control unit and assert `Load_XMT_DR` only when the state is `idle` and the external `Load_XMT_datareg` signal is asserted. The status signal `BC_lt_BCmax` is asserted while bits are being sent, i.e., while `bit_count` < `word_size` + 1.

- `Load_XMT_datareg`: when asserted in state `idle`, this asserts `Load_XMT_DR`, which loads the contents of `Data_Bus` into `XMT_datareg`.
- `Byte_ready`: assertion causes `Load_XMT_shftreg` to assert, which loads the contents of `XMT_datareg` into `XMT_shftreg`.
- `T_byte`: assertion initiates transmission of a byte of data, including the stop, start, and parity bits.
- `BC_lt_BCmax`: indicates the status of the bit counter in the datapath unit.

![Figure 4. Algorithmic State Machine and Datapath Chart (ASMD) for the UART transmitter]({{ "/assets/files/lab02/img/4.png" | relative_url }})

*Figure 4. Algorithmic State Machine and Datapath Chart (ASMD) for the UART transmitter*

The ASMD chart of the state machine controlling the transmitter is shown in Figure 4. The machine has three states: `idle`, `waiting`, and `sending`. When the active-low, synchronous reset signal `rst_b` is asserted, the machine enters `idle`, `bit_count` is cleared, and `XMT_shftreg` is loaded with 1s. In `idle`, if an active edge of `Clock` occurs while the external host asserts `Load_XMT_datareg`, `XMT_datareg` is loaded with the contents of `Data_Bus`. The machine remains in `idle` until `start` is asserted to drop `XMT_shftreg[0]`.

![Figure 5. Waveforms of the 8-bit UART transmitter]({{ "/assets/files/lab02/img/5.png" | relative_url }})

*Figure 5. Waveforms of the 8-bit UART transmitter*

Figure 5 shows an example of the transmission timing. You can use this timing diagram both to design your test bench and to check your results.

## Implementation & Simulation

We will now implement the transmitter portion of the UART design.

Please login to the Olympus server and create a working directory for this lab using the following commands.

```bash
## Create and navigate to the working directory.
mkdir -p $HOME/ECEN468/Lab2/src
cd $HOME/ECEN468/Lab2/src
```

Download the tar.gz file from the lab website and extract it. In the extracted folders, you will find the following files:

- `UART_XMTR.cpp`
- `UART_XMTR.h`
- `test.cpp`
- `test.h`
- `main.cpp`

Copy them to the working directory.

You will write your code in `UART_XMTR.cpp`, `UART_XMTR.h`, and `main.cpp`. Figure 6 shows the file hierarchy for this lab.

![Figure 6. Hierarchy of the files for the UART system]({{ "/assets/files/lab02/img/6.png" | relative_url }})

*Figure 6. Hierarchy of the files for the UART system*

Once you complete the implementation, verify your design by simulating it and viewing the waveform as in Lab 1. Take screenshots of the simulation output and the waveform, and include them in your report.

Commands for reference:

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh
wv &
```

## Submission

Please submit a single PDF file containing the following:

1. Screenshots of the waveform with analysis.
2. Screenshots of the simulation output in Vista.
3. Screenshots of your code in this design with reasonable comments.

## Code Example

Pseudocode of the UART state machine:

```
// State Machine Initialization
Initialize State Machine:
  If rst_b signal is low:
    CurrentState = IDLE
    Reset ShiftRegister and BitCounter
  Else:
    CurrentState = NextState

// State Machine Operation Logic
FSM:
  Execute the following actions based on the CurrentState:
  If CurrentState is IDLE:
    If Load_XMT_datareg signal is high:
      DataRegister = DataBus value
      NextState = IDLE
    Else If Byte_ready signal is high:
      ShiftRegister = DataRegister shifted left and appended with start bit (1'b1)
      NextState = WAITING
  If CurrentState is WAITING:
    If T_byte signal is high:
      ShiftRegister's LSB = 0
      NextState = SENDING
    Else:
      NextState = WAITING
  If CurrentState is SENDING:
    If BitCounter is less than WORD_SIZE + 1:
      ShiftRegister = ShiftRegister shifted right and appended with stop bit (1'b1)
      BitCounter = BitCounter + 1
      NextState = SENDING
    Else:
      BitCounter = 0
      NextState = IDLE
  Else:
    NextState = IDLE
  Write the LSB of ShiftRegister to Serial_out signal
```
