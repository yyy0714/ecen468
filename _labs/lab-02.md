---
layout: manual
title: 'Lab 2: Design of UART Transmitter (SystemC)'
session: 'Week 7 (Oct 5 – Oct 9)'
report_due: 'Week 8 (Oct 12 – Oct 16)'
manual_pdf: /assets/files/lab02/lab02_manual.pdf
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab02/lab02_code.tar.gz
---

# 1. Objectives
- Complete design of UART transmitter in SystemC.
- Build and simulate the design.

---

## 2. Introduction
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

# 3. Lab Procedures

## 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `cd $HOME/ecen468/`

2. Download `lab02_code.tar.gz` from the lab website and put it in the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab01_code.tar.gz`
  - `rm lab01_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `UART` (directory for C++ source code and header files)
        - `main.cpp`
        - `test.cpp`
        - `UART_XMTR.cpp`
        - `test.h`
        - `UART_XMTR.cpp`

    - `workspace` (directory where Siemens Vista will be run)

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

## 3.1 Building SystemC Design Using Siemens Vista
1. Execute the following commands in sequence to open Siemens Vista.
    - `source /opt/coe/mentorgraphics/vista/2024_2/setup.vista.linux.bash`
    - `source /opt/rh/gcc-toolset-13/enable`
    - `cd $HOME/ecen468/lab02/workspace`
    - `vista &`

2. In Vista graphical interface, click **Project** on the menu bar, then click **New Project...**.

3. In the **New Project** dialog, click **Save** (You don't need to change the default file name).

4. In the **Create New Project** dialog, click **Files** on the tab bar, then click **Add Files** on the bottom right.

5. In the **Select Files** dialog, click the folder icon and select the C++ source code and header files, click **Open**, then click **OK**.

6. On the side bar, right click on **Project** and click **Build**. Every time you modify the source code or header files, you need to rebuild the project.

7. If no errors occur, proceed to the next section.

## 3.2 Simulating SystemC Design Using Siemens Vista
1. On the side bar, click the **+** next to **Project** expand its content.

2. Click the **+** next to **Simulation**, then right click on **main.cpp** and click **Simulate**.

3. In the **simulation** dialog, deselect **Stop after elaboration** under **Debugging**, then click **OK**.

4. After the simulation exits, a waveform file named `wave.vcd` will be generated in `workspace`.

5. In the terminal, execute the following commands to Synopsys WaveView.
    - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
    - `wv &`

6. In WaveView, open `wave.vcd` to view the simulation waveform.

7. Take a screenshot of the waveform for the lab report.

---

## 4. Submission
Please submit a single PDF file containing the following:
1. Screenshots of the waveform with analysis.
2. Screenshots of the simulation output in Vista.
3. Screenshots or copy of the content of the following files:
    - `UART_XMTR.cpp`
    - `UART_XMTR.h`

## Appendix: Code Example
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
