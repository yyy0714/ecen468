---
layout: manual
title: 'Lab 1: Introduction to SystemC and Simulator'
# session: 'Week 2 (Aug 31 – Sep 4)'
# report_due: 'Week 3 (Sep 7 – Sep 11)'
session: 'Week 6 (Sep 28 – Oct 2)'
report_due: 'Week 7 (Oct 5 – Oct 9)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab01/lab01_code.tar.gz
---

## 1. Objectives
- Complete design of SRAM in SystemC.
- Build and simulate the design.

---

## 2. Introduction

**SystemC** is a set of C++ classes and macros which provides an event-driven simulation kernel in C++. These facilities enable a designer to simulate concurrent processes, and each is described using plain C++ syntax. SystemC processes can communicate in a simulated real-time environment, using signals of all the datatypes either offered by C++, provided by the SystemC library, or defined by a designer. In certain respects, SystemC deliberately mimics the hardware description language (VHDL and Verilog) but is more aptly described as a system-level modeling language.

**Static Random Access Memory** (SRAM) can be implemented with a storage cell structure that does not require a refresh. Therefore, it operates faster than Dynamic Random Access Memory (DRAM) and is used as fast-cache memory in a computer. As a starting point of our implementation, we will look into a simple SRAM cell, proceed to larger memory blocks with unidirectional data ports, and finally implement a memory block directly.

The block diagram symbol of a basic SRAM cell is shown in Figure 1. It has active-low inputs for Cell Select (`CS`) and Write Enable (`WE`). Note the absence of a clock signal. Storage registers and register files are implemented with flip-flops. The storage devices of RAMs, however (including SRAM and DRAM), use transparent latches, which support asynchronous storage and retrieval of data and minimize the time a RAM ties up a shared bus.

![Figure 1. SRAM cell: block diagram symbol]({{ "/assets/files/lab01/img/1.png" | relative_url }})

*Figure 1. SRAM cell: block diagram symbol*

An SRAM is built from many SRAM cells arranged in what is called an SRAM cell array. Figure 2 shows an 8 x 8 SRAM cell array that holds eight words of eight bits each. To write one word (8 bits) into this array, only one `CS` signal is enabled. The `data_in` and `data_out` signals are eight bits wide.

![Figure 2. Cell array of 8 x 8 SRAM]({{ "/assets/files/lab01/img/2.png" | relative_url }})

*Figure 2. Cell array of 8 x 8 SRAM*

An address decoder in the SRAM structure activates the cell-select signal (`CS_b`) for a single row at a time. Figure 3 shows the complete structure of the 8 x 8 SRAM, in which `data_in` and `data_out` are eight bits wide. Addressing requires three pins, since 2^3 = 8. A chip-enable signal `CE` is also added to control the entire SRAM; when `CE` is set to 1, all address pins are disabled.

![Figure 3. The entire structure of 8 x 8 SRAM]({{ "/assets/files/lab01/img/3.png" | relative_url }})

*Figure 3. The entire structure of 8 x 8 SRAM*

---

## 3. Implementation

We will now implement a 256K (262,144) x 8 SRAM, which has 18 address pins and 8 data pins. To generate `CS_b` for every memory cell, the 18-bit address must be decoded to control all 256K cells. The amount of decoding logic grows as the number of address pins increases. To reduce the size of the address decoder, a two-level addressing scheme can be used. Figure 4 shows an example of two-level addressing for an 8 x 1 SRAM.

![Figure 4. Address decoder for the 8 x 1 SRAM]({{ "/assets/files/lab01/img/4.png" | relative_url }})

*Figure 4. Address decoder for the 8 x 1 SRAM*

We will use Vista to implement the design. Vista is a native electronic system level (ESL) platform for architecture design, verification, analysis, and virtual prototyping, with an advanced toolset aimed at high-level transaction-level modeling (TLM) hardware platforms.

---

# 3. Lab Procedures

## 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `cd $HOME/ecen468/`

2. Download `lab01_code.tar.gz` from the lab website and put it in the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab01_code.tar.gz`
  - `rm lab01_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `SRAM` (directory for C++ source code and header files)
        - `main.cpp`
        - `RAM.cpp`
        - `test.cpp`
        - `RAM.h`
        - `test.h`

    - `workspace` (directory where Siemens Vista will be run)

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

## 3.1 Building SystemC Design Using Siemens Vista
1. Execute the following commands in sequence to open Siemens Vista.
    - `source /opt/coe/mentorgraphics/vista/2024_2/setup.vista.linux.bash`
    - `source /opt/rh/gcc-toolset-13/enable`
    - `cd $HOME/ecen468/lab01/workspace`
    - `vista &`

2. In Vista graphical interface, click **Project** on the menu bar, then click **New Project...**.

3. In the **New Project** dialog, click **Save** (You don't need to change the default file name).

4. In the **Create New Project** dialog, click **Files** on the tab bar, then click **Add Files** on the bottom right.

5. In the **Select Files** dialog, click the folder icon and select the C++ source code and header files, click **Open**, then click **OK**.

<!-- 6. In the **Create New Project** dialog, click **Compilation** on the menu bar, then add the following flag in **Compilation Options** (Make sure these is a space between flags), then click **OK**.
    - `-std=c++11` -->

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

1. Screenshot of the waveform with analysis.
2. Screenshots or copy of the content of the following files:
    - `RAM.cpp`
    - `RAM.h`
3. Answer to the following question:
    - What are the differences between asynchronous and synchronous SRAM?

---

## Appendix: Code Example

```cpp
//===========================================
// Function : 4Bit Adder
//===========================================
#include "systemc.h"

#define DATA_WIDTH 4

SC_MODULE (mAdder) {
    sc_in <sc_uint<DATA_WIDTH>> dInA;
    sc_in <sc_uint<DATA_WIDTH>> dInB;
    sc_out <sc_uint<DATA_WIDTH+1>> dOut;

    // ----- Code Starts Here -----
    void function_adder () {
        dOut.write(dInA.read()+dInB.read());
    }

    // ----- Constructor for the SC_MODULE -----
    // sensitivity list
    SC_CTOR(mAdder) {
        SC_METHOD (function_adder);
        sensitive << dInA << dInB;
    }
};

int sc_main (int argc, char* argv[]) {
    // Declare Input/Output Signals
    sc_signal <sc_uint<DATA_WIDTH>> tInA;
    sc_signal <sc_uint<DATA_WIDTH>> tInB;
    sc_signal <sc_uint<DATA_WIDTH+1>> tOut;

    int i,j;

    // Connect the DUT(Design Under Test)
    mAdder Adder_01("SIMULATION_adder");
    Adder_01.dInA(tInA);
    Adder_01.dInB(tInB);
    Adder_01.dOut(tOut);

    // Open VCD(Value Change Dump) file
    sc_trace_file *wf = sc_create_vcd_trace_file("VCD_ADDER");

    // Dump the desired signals
    sc_trace(wf, tInA, "strInA");
    sc_trace(wf, tInB, "strInB");
    sc_trace(wf, tOut, "strOut");

    // Initialize all variables
    tInA.write(0);
    tInB.write(0);
    sc_start(5);

    // Behavior
    for(i=0; i<16; i++){
        tInA.write(i);
        for(j=0; j<16; j++){
            tInB.write(j);
            sc_start(2);
            cout << "@" << sc_time_stamp() << ":: [" << tInA << "]+["
            << tInB << "]=[" << tOut << "]" << endl;
        }
    }

    // Close trace file
    sc_close_vcd_trace_file(wf);

    return 0; // Terminate simulation
}
```
