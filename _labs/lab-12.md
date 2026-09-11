---
layout: manual
title: 'Lab 12: Introduction to Verilog AMS and Simulator'
session: 'Week 13 (Nov 16 – Nov 20)'
report_due: 'Week 14 (Nov 23 – Nov 27)'
---

## Purpose

Verilog-AMS HDL is a standard modeling language for analog circuits, and Virtuoso is a simulator for Verilog-AMS. We will design a simple application — a phase-locked loop (PLL) — in Verilog-AMS and use Virtuoso for simulation.

## Preparation

### 1. Brief introduction to PLL (Phase Locked Loop)

We will simulate the functionality of a PLL. A phase-locked loop or phase lock loop (PLL) is a control system that generates an output signal whose phase is related to the phase of an input "reference" signal. It is an electronic circuit with a variable frequency oscillator and a phase detector. This circuit compares the phase of the input signal with the phase of the signal derived from its output oscillator and adjusts the frequency of its oscillator to keep the phases matched. The signal from the phase detector is used to control the oscillator in a feedback loop.

![Figure 1. Clock Distribution]({{ "/assets/files/lab12/img/1.png" | relative_url }})

*Figure 1. Clock Distribution*

Frequency is the derivative of phase, so keeping the input and output phases in lock step also keeps the input and output frequencies in lock step. Consequently, a phase-locked loop can track an input frequency, or generate a frequency that is a multiple of the input frequency. We will simulate properties used for indirect frequency synthesis or demodulation.

Figure 1 shows the clock distribution of the PLL block. Typically, the reference clock enters the chip and drives a phase-locked loop (PLL), which drives the system's clock distribution. The clock distribution is usually balanced so that the clock arrives at every endpoint simultaneously. One of those endpoints is the PLL's feedback input. The function of the PLL is to compare the distributed clock to the incoming reference clock and vary the phase and frequency of its output until the reference and feedback clocks are phase and frequency matched.

A PLL includes a Phase detector, Low-pass filter, Variable-frequency oscillator, and feedback path (which may include a frequency divider), as shown in Figure 2.

![Figure 2. Phase Locked Loop block diagram]({{ "/assets/files/lab12/img/2.png" | relative_url }})

*Figure 2. Phase Locked Loop block diagram*

A phase detector compares two input signals and produces an error signal which is proportional to their phase difference. The error signal is then low-pass filtered and used to drive a VCO, which creates an output phase. The output is fed through an optional divider back to the input of the system, producing a negative feedback loop. If the output phase drifts, the error signal will increase, driving the VCO phase in the opposite direction to reduce the error. Thus the output phase is locked to the phase at the other input. This input is called the reference.

### 2. Description of Verilog AMS

Verilog-AMS Hardware Description Language (HDL) is derived from the IEEE 1364 Verilog HDL specification. The intent of Verilog-AMS HDL is to let designers create and use modules that encapsulate high-level behavioral descriptions as well as structural descriptions of systems and components. The behavior of each module can be described mathematically in terms of its terminals and external parameters applied to the modules. The structure of each component can be described in terms of interconnected sub-components. These descriptions can be used in many disciplines, such as electrical, mechanical, fluid dynamics, and thermodynamics.

A system is considered to be a collection of interconnected components that are acted upon by a stimulus and produce a response. The components themselves might also be systems. If a component does not have any sub-components, then it is considered a primitive component. Each primitive component connects to one or more nodes. The behavior of each component is defined in terms of signal values at each node. The components connect to nodes through ports to build a hierarchy as shown in Figure 3.

![Figure 3. Components connect to nodes through ports]({{ "/assets/files/lab12/img/3.png" | relative_url }})

*Figure 3. Components connect to nodes through ports*

To simulate systems, it is necessary to have a complete description of the system and all of its components. Descriptions of systems are given structurally. That is, the description of a system contains instances of components and how they are interconnected. Descriptions of primitive components are given behaviorally. That is, a mathematical description is given that relates the signals at the ports of the component.

### 3. Functional Simulation

Make a working folder for this lab.

```bash
mkdir -p $HOME/ECEN468/Lab12/src              # make work folder
cd $HOME/ECEN468/Lab12/src                    # move to work folder
/opt/coe/ncsu/ncsu-cdk-1.6.0.beta/ncsu.sh     # start Virtuoso
```

Then, this will load Virtuoso, and you will see the Command Interpreter Window (CIW) as shown in Figure 4.

![Figure 4. Command Interpreter Window (CIW)]({{ "/assets/files/lab12/img/4.png" | relative_url }})

*Figure 4. Command Interpreter Window (CIW)*

Select **Tools -> Library Manager** to open the Library Manager (Figure 5). It has three main sections: Library, Cell, and View. The Library section lists all libraries, each containing cells such as `vexp`, `vnpn`, `vpnp`, and `vpulse`. Each cell can be viewed in several ways — for example, as a symbol or a schematic — as shown in the View section.

![Figure 5. Library Manager]({{ "/assets/files/lab12/img/5.png" | relative_url }})

*Figure 5. Library Manager*

You will follow these steps to design and simulate the PLL in this lab:

- 3.1. Creating Library
- 3.2. Creating Cells
- 3.3. Connect Cells and Make PLL
- 3.4. Connect input pulse to PLL
- 3.5. Simulation

#### 3.1. Creating Library

You will create a library that will contain cells such as the Phase Frequency Detector (PFD), Voltage Controlled Oscillator (VCO), Frequency Divider (FD), and the top module. From the Library Manager, select **File -> New -> Library**. In the New Library window (Figure 6(a)), enter `PLL` as the library name and click **OK**. When prompted for a technology file, choose to reference existing technology libraries (Figure 6(b)), and move all technology libraries from the left box to the right box (Figure 6(c)). The `PLL` library will then appear in the Library section of the Library Manager.

![Figure 6. Creating a Library]({{ "/assets/files/lab12/img/6.png" | relative_url }})

*Figure 6. Creating a Library. (a) New Library window, (b) Technology options, (c) reference existing technology libraries window*

#### 3.2. Creating Cells

We will create cells in `PLL`, starting with the Phase Frequency Detector (PFD). Figure 7 shows the complete schematic of the input signal and PLL to be implemented in this lab. To create the PFD cell, click `PLL` in the Library section and select **File -> New -> Cell View**. Enter `pfd` as the cell name, choose `VerilogA` as the cell type (Figure 8(a)), and click **OK**. If you are asked about the license, click **Always** to keep this setting for future licenses (Figure 8(b)).

![Figure 7. Schematic of the input signal and PLL]({{ "/assets/files/lab12/img/7.png" | relative_url }})

*Figure 7. Schematic of the input signal and PLL*

![Figure 8. Processes of creating a cell]({{ "/assets/files/lab12/img/8.png" | relative_url }})

*Figure 8. Processes of creating a cell*

An editor opens with the default contents of the `pfd` cell. Enter only the input and output signal names, as shown in Figure 9; the ports will be generated automatically when we create the cell's symbol.

![Figure 9. Verilog A file of the PFD cell]({{ "/assets/files/lab12/img/9.png" | relative_url }})

*Figure 9. Verilog A file of the PFD cell*

After you close the editor, a dialog about the symbol appears, because the PFD cell does not have a symbol yet. Click **Yes** to create one. The default signal names appear in the Symbol Generation Options dialog (Figure 10(a)), because we already defined the input and output signals in the Verilog-A module. Place the pins in their proper positions: put `ref` and `fb` (feedback) in the Left Pins field and `out` in the Right Pins field. Then click the **List** button to set the signal directions: set `ref` and `fb` to inputs and `out` to an output.

![Figure 10. Symbol generation option dialog for the PFD cell]({{ "/assets/files/lab12/img/10.png" | relative_url }})

*Figure 10. Symbol generation option dialog for the PFD cell*

The default `pfd` symbol then appears, as shown in Figure 11. You can leave it as is or redraw it in any shape you like. Close the window when you are done.

![Figure 11. Symbol view of pfd module]({{ "/assets/files/lab12/img/11.png" | relative_url }})

*Figure 11. Symbol view of pfd module*

After you close the window, you return to the Library Manager, which now shows the `pfd` cell in the `PLL` library (Figure 12).

![Figure 12. Library Manager, pfd cell generation]({{ "/assets/files/lab12/img/12.png" | relative_url }})

*Figure 12. Library Manager, pfd cell generation*

In the View section, the Phase Frequency Detector cell has a `symbol` view and a `veriloga` view. Clicking `symbol` shows its symbol (Figure 11), and clicking `veriloga` opens the Verilog-A editor.

You can describe the PFD module directly in this editor, or edit the Verilog-A file (`veriloga.va`) with any editor. You will find it at the path below. You may refer to the code at the end of this manual.

`ECEN468/Lab12/src/PLL/PFD/veriloga`

Create the Frequency Divider and Voltage Controlled Oscillator by following Section 3.2.

- **Frequency Divider**
  - name: `fd`
  - input name: `in` (to Right Pins)
  - output name: `out` (to Left Pins)
- **Voltage Controlled Oscillator**
  - name: `vco`
  - input name: `in` (to Left Pins)
  - output name: `out` (to Right Pins)

Describe their behavior in the Verilog-A files, referring to the code at the end of this manual. After creating all the PLL cells, you will have three cells — `fd`, `pfd`, and `vco` — as shown in Figure 13.

![Figure 13. Library Manager, pfd, fd and vco cells generation]({{ "/assets/files/lab12/img/13.png" | relative_url }})

*Figure 13. Library Manager, pfd, fd and vco cells generation*

#### 3.3. Connect Cells and Make PLL

Now we have all the cells needed to build the PLL block. We will connect the `pfd`, `fd`, and `vco` cells to form the PLL top module, which is also a cell. To create it, click `PLL` in the Library section and select **File -> New -> Cell View**. Enter `plltop` as the cell name, choose `schematic` as the cell type (Figure 14(a)), and click **OK**. A blank Virtuoso schematic editor then appears (Figure 14(b)).

![Figure 14. Creating a cell, plltop]({{ "/assets/files/lab12/img/14.png" | relative_url }})

*Figure 14. Creating a cell, plltop*

In the schematic editor, you will load the three sub-cells and connect them. Select **Create -> Instance**, enter `PLL` as the library and `pfd` as the cell, and place the `pfd` cell in the editor (Figure 15). Place `fd` and `vco` as well.

![Figure 15. Creating a cell, plltop]({{ "/assets/files/lab12/img/15.png" | relative_url }})

*Figure 15. Creating a cell, plltop*

Also create a resistor and two capacitors using **Create -> Instance**. Then connect all the cells as in Figure 7, with the result shown in Figure 16. For the library names, cell names, and values, refer to Table I. To connect the instances, draw wires using **Create -> Wire (narrow)**.

| Component | Library | Cell | Value |
|---|---|---|---|
| Resistor R1 | `analogLib` | `res` | `134.041K` |
| Capacitor C1 | `analogLib` | `cap` | `475p` |
| Capacitor C2 | `analogLib` | `cap` | `32p` |
| Ground | `analogLib` | `gnd` | non |

*Table I. Resistor, Capacitor and Ground*

Figure 16 shows the final schematic of `plltop`. Save the file using **File -> Save**.

![Figure 16. The final schematic of plltop cell]({{ "/assets/files/lab12/img/16.png" | relative_url }})

*Figure 16. The final schematic of plltop cell*

#### 3.4. Connect input pulse to PLL

An input pulse source is used as the input to `plltop`, connected to the `ref` port of the `pfd` cell. Select **Create -> Instance** and enter `analogLib` as the library name and `vpulse` as the cell name. Set the `vpulse` parameters to `+1V` for Vmax (voltage1), `-1V` for Vmin (voltage2), `100ns` for Period, and `1ns` for both Rise time and Fall time. The circuit is now complete. Name all the nets as shown in Figure 17, then save the schematic. Whenever you modify the schematic, click **File -> Check and Save**.

![Figure 17. The schematic of plltop and input pulse]({{ "/assets/files/lab12/img/17.png" | relative_url }})

*Figure 17. The schematic of plltop and input pulse*

Next, name the wires using the net names in Table II. Click **Create -> Wire Name**, enter the wire name, and click **Hide**, then click the wire you want to name.

| Wires | Wire names |
|---|---|
| `ref` of `pfd` | `REF_PFD` |
| `out` of `fd` | `OUT_FD` |
| `out` of `vco` | `OUT_VCO` |
| `in` of `vco` | `IN_VCO` |

*Table II. Net names*

#### 3.5. Simulation

Now we will simulate the `plltop` module, which we described with Verilog-A and a schematic. In the Virtuoso schematic editor, select **Launch -> ADE (Analog Design Environment) L** to start the simulator. Choose the analysis type with **Analyses -> Choose** (Figure 18): select `tran` as the analysis type and enter an appropriate Stop Time. In this lab, we simulate for `50us`, so enter `50u`. Then set the Accuracy Defaults to **moderate** and click **OK**.

![Figure 18. Analyses Dialog]({{ "/assets/files/lab12/img/18.png" | relative_url }})

*Figure 18. Analyses Dialog*

In the **Outputs** section of the ADE window, right-click and select **Edit** to choose the wires to plot. Click **From Schematic**, then click the nets you want to plot and click **OK**. Choose the nets `REF_PFD`, `OUT_FD`, `OUT_VCO`, and `IN_VCO`. The waveforms will then be displayed, as shown in Figure 21.

You may want to save the current settings so you do not have to reconfigure them for future simulations. Save the current state by selecting **Sessions -> Save state**, checking **CellView**, and clicking **OK** (Figure 19). To reload the saved state later, select **Sessions -> Load state** and check **CellView**.

![Figure 19. Saving state]({{ "/assets/files/lab12/img/19.png" | relative_url }})

*Figure 19. Saving state*

With all settings in place, select **Simulation -> Netlist and Run** to simulate `plltop`. The simulation status appears as in Figure 20, and the waveforms are displayed as in Figure 21.

![Figure 20. Simulation process]({{ "/assets/files/lab12/img/20.png" | relative_url }})

*Figure 20. Simulation process*

![Figure 21 (a). Final simulation results]({{ "/assets/files/lab12/img/21.png" | relative_url }})

*Figure 21 (a). Final simulation results*

![Figure 21 (b). Final simulation results]({{ "/assets/files/lab12/img/22.png" | relative_url }})

*Figure 21 (b). Final simulation results*

## Requirements

1. General requirements.
   - a. It should control the `PLL` correctly.
   - b. Make sure the net and cell names match those described in this manual.
   - c. Late penalty: 20% of the total score will be deducted for each subsequent day after the due date.
2. Please submit a single PDF file containing the following:
   - a. Simulation results including the four nets in Table II (like Figure 21).
   - b. Suppose you want the Voltage Controlled Oscillator to generate a 1GHz output from a 1MHz reference clock. Which parts would need to be modified?

## Verilog-A Code

```verilog
//--------------------------------------------------
// Phase Frequency Detector
//--------------------------------------------------
// VerilogA for PLL, pfd, veriloga

`include "constants.vams"
`include "disciplines.vams"

module pfd(out, ref, fb);
output out; electrical out;  // current output
input ref; voltage ref;      // positive input (edge triggered)
input fb; voltage fb;        // inverting input (edge triggered)
parameter real iout=100u;
parameter real vh=1;
parameter real vl=-1;
parameter real vth=(vh+vl)/2;
parameter integer dir=1 from [-1:1] exclude 0;
// dir=1 for positive edge trigger
// dir=-1 for negative edge trigger
parameter real tt=1n from (0:inf);
parameter real td=0 from [0:inf);
integer state;

analog begin
    // Implement phase detector
    @(cross(V(ref)-vth, dir))
        if (state > -1) state = state - 1;
    @(cross(V(fb)-vth, dir))
        if (state < 1) state = state + 1;
    // Implement charge pump
    I(out) <+ transition(iout*state, td, tt);
end
endmodule
```

```verilog
//--------------------------------------------------
// Frequency Divider
//--------------------------------------------------
// VerilogA for PLL, fd, veriloga

`include "constants.vams"
`include "disciplines.vams"

module fd(out, in);
output out; voltage out;  // output
input in; voltage in;     // input (edge triggered)
parameter real vh=+1;
parameter real vl=-1;
parameter real vth=(vh+vl)/2;
parameter integer ratio=10 from [2:inf);
parameter integer dir=1 from [-1:1] exclude 0;
// dir=1 for positive edge trigger
// dir=-1 for negative edge trigger
parameter real tt=1n from (0:inf);
parameter real td=0 from [0:inf);
integer count, n;

analog begin
    @(cross(V(in) - vth, dir)) begin
        count = count + 1;
        if (count >= ratio)
            count = 0;
        n = (2*count >= ratio);
    end
    V(out) <+ transition(n ? vh : vl, td, tt);
end
endmodule
```

```verilog
//--------------------------------------------------
// Voltage Controlled Oscillator
//--------------------------------------------------
// VerilogA for PLL, vco, veriloga

`include "disciplines.vams"
`include "constants.vams"

`define PI 3.14159265358979323846264338327950288419716939937511

module vco(in, out);
input in;
output out;
voltage in, out;
parameter real amp = 1;
parameter real center_freq = 100M;
parameter real vco_gain = 50M;
parameter integer steps_per_period = 32;
real phase;
real inst_freq;
real real_vout;

analog begin
    inst_freq = center_freq + vco_gain * V(in);
    $bound_step (1.0 / (steps_per_period*inst_freq));
    phase = idtmod(inst_freq,0,1);
    V(out) <+ amp * sin (2 * `PI * phase);
end
endmodule
```
