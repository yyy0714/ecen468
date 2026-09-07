---
layout: manual
title: 'Lab 1: Introduction to SystemC and Simulator'
session: 'Week 2 (Aug 31 – Sep 4)'
report_due: 'Week 3 (Sep 7 – Sep 11)'
manual_pdf: /assets/files/lab01/lab01_manual.pdf
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab01/lab01_code.tar.gz
---

## Objectives

In the first several labs, we will design a separate module of our whole system for every lab and simulate it with a test bench. Finally, we will simulate the entire module, which is designed using SystemC.

SystemC is a system-level modeling language, and Vista is a platform for architecture design. In this lab, we will design a simple memory application with SystemC and use Vista as a tool for simulation and verification.

## Introduction

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

## Implementation

We will now implement a 256K (262,144) x 8 SRAM, which has 18 address pins and 8 data pins. To generate `CS_b` for every memory cell, the 18-bit address must be decoded to control all 256K cells. The amount of decoding logic grows as the number of address pins increases. To reduce the size of the address decoder, a two-level addressing scheme can be used. Figure 4 shows an example of two-level addressing for an 8 x 1 SRAM.

![Figure 4. Address decoder for the 8 x 1 SRAM]({{ "/assets/files/lab01/img/4.png" | relative_url }})

*Figure 4. Address decoder for the 8 x 1 SRAM*

We will use Vista to implement the design. Vista is a native electronic system level (ESL) platform for architecture design, verification, analysis, and virtual prototyping, with an advanced toolset aimed at high-level transaction-level modeling (TLM) hardware platforms.

Please login to the Olympus server and create a working directory for this lab using the following commands.

```bash
## Create and navigate to the working directory.
mkdir -p $HOME/ECEN468/Lab1/src
cd $HOME/ECEN468/Lab1/src
```

Download the tar.gz file from the lab website and extract it. In the extracted folders, you will find the following files:

- `RAM.cpp`
- `test_RAM.cpp`

Copy them to the working directory.

In the terminal, open Vista using the following commands.

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/mentorgraphics/vista/2024_2/setup.vista.linux.bash
vista &
```

Once Vista is launched, create a new project: click **Project -> New Project**, set the project name to `RAM.v2p`, go to the **Files** tab (Figure 5), click **Add Files**, and add `RAM.cpp` and `test_RAM.cpp`.

![Figure 5. Project settings window]({{ "/assets/files/lab01/img/5.png" | relative_url }})

*Figure 5. Project settings window*

Now implement the RAM design in `RAM.cpp`. For reference, an implementation of a 4-bit adder is provided at the end of this manual.

Once you finish your design, compile it to check for syntax and behavioral errors by right-clicking the tab of your design (`RAM`) and selecting **Build**, as shown in Figure 6.

![Figure 6. Screenshot of building project]({{ "/assets/files/lab01/img/6.png" | relative_url }})

*Figure 6. Screenshot of building project*

## Simulation

Once the design is complete, we test its functionality by applying a test bench and checking the outputs. The test bench instantiates the design under test (DUT) and drives its input signals. It is compiled together with the design module, and the simulation results are displayed at the end of compilation. Figure 7 shows the block diagram of the 256K x 8 RAM and its test bench.

![Figure 7. The block diagram of the SRAM and its test bench]({{ "/assets/files/lab01/img/7.png" | relative_url }})

*Figure 7. The block diagram of the SRAM and its test bench*

To simulate the design, expand the hierarchy and click **Simulate**, as shown in Figure 8. In the pop-up window, make sure the target is set to the design name, de-select the **Stop After Elaboration** option (Figure 9), and click **OK**.

![Figure 8. Screenshot of simulating the project]({{ "/assets/files/lab01/img/8.png" | relative_url }})

*Figure 8. Screenshot of simulating the project*

![Figure 9. The window of simulation settings]({{ "/assets/files/lab01/img/9.png" | relative_url }})

*Figure 9. The window of simulation settings*

After simulation, click the simulation output frame at the bottom of the window and press `Ctrl-X` then `Ctrl-S` to save the output. Enter `sim.out` as the filename and press `Enter` to save, as shown in Figure 10.

![Figure 10. Saving the simulation]({{ "/assets/files/lab01/img/10.png" | relative_url }})

*Figure 10. Saving the simulation*

Once the simulation completes, the waveform is saved as a `*.vcd` file. We can view it using WaveView. To launch WaveView, open a new terminal on the server and run the following commands.

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh
wv &
```

In WaveView, open the `*.vcd` file generated by the simulation to see the waveform. Refer to Figure 11 for the expected result, then take a screenshot of your waveform and include it in your lab report.

![Figure 11. The waveform of the SRAM]({{ "/assets/files/lab01/img/11.png" | relative_url }})

*Figure 11. The waveform of the SRAM*

## Submission

Please submit a single PDF file containing the following:

1. Screenshots of the waveform with analysis.
2. Screenshots of your code in this design with reasonable comments.
3. Q1: What are the differences between asynchronous and synchronous SRAM?

## Code Example

```cpp
//===========================================
// Function : 4Bit Adder
//===========================================
#include "systemc.h"

#define DATA_WIDTH 4

SC_MODULE (mAdder) {
    sc_in <sc_uint<DATA_WIDTH> > dInA ;
    sc_in <sc_uint<DATA_WIDTH> > dInB ;
    sc_out <sc_uint<DATA_WIDTH+1> > dOut ;

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
    sc_signal < sc_uint<DATA_WIDTH> > tInA;
    sc_signal < sc_uint<DATA_WIDTH> > tInB;
    sc_signal < sc_uint<DATA_WIDTH+1> > tOut;

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
