---
layout: manual
title: 'Lab 11: Combining Canny Edge Detector with System Bus, Memory and UART (Verilog)'
session: 'Week 12 (Nov 9 – Nov 13)'
report_due: 'Week 13 (Nov 16 – Nov 20)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab11/lab11_code.tar.gz
---

## Objectives

In this lab, we will combine the Canny Edge Detector, SRAM, and UART through the system bus.

![Figure 1. Block diagram of the system]({{ "/assets/files/lab11/img/1.png" | relative_url }})

*Figure 1. Block diagram of the system*

## Combining the edge detector with the system bus

In Lab 10, the image data moved between the test bench and the modules, and the large internal memory required for the image was declared in the test bench. In this lab, we will use the SRAM to store the image. Figure 1 shows the complete system. The test bench now handles only BMP image input/output and control logic, and it still keeps internal memory for reading and writing BMP files.

![Figure 2. Connection of the Canny Edge Detector and its decoder]({{ "/assets/files/lab11/img/2.png" | relative_url }})

*Figure 2. Connection of the Canny Edge Detector and its decoder*

![Figure 3. Address map of the Canny Edge Detector]({{ "/assets/files/lab11/img/3.png" | relative_url }})

*Figure 3. Address map of the Canny Edge Detector*

Figure 2 shows the signals connecting the Canny Edge Detector module and its decoder. To attach the module to the system bus, an address decoder (wrapper) is required to decode the control signals. Figure 3 shows the address map. The four most significant bits hold the device identification number; we assume the ID of the Canny Edge Detector is `0100`. The remaining bits carry control and address signals.

Figure 4 shows an example of the noise reduction process. First, the test bench sends all the data to memory. Then only the data needed for processing is fetched from memory to the test bench and passed to the Canny Edge Detector module. After the detector finishes processing the block, the test bench reads the result and writes it back to the appropriate memory.

![Figure 4. Data flow of Noise Reduction]({{ "/assets/files/lab11/img/4.png" | relative_url }})

*Figure 4. Data flow of Noise Reduction*

For debugging, the system sends several messages through the UART to indicate the completion of each stage (noise reduction, gradient extraction, non-maximum suppression, and hysteresis thresholding).

The messages below should be sent through UART:

- Noise reduction: `'N'` = `0x4E`
- Gradient extraction: `'G'` = `0x47`
- Non-maximum suppression: `'S'` = `0x53`
- Hysteresis thresholding: `'H'` = `0x48`

## Reconfiguration of SRAM

We will expand the memory to support the Canny Edge Detector. Figure 5 shows the corresponding memory map. The detector needs space to store several intermediate results: the original image, the noise-reduced image, the gradient information, the direction information, the image after non-maximum suppression, and the final image after hysteresis thresholding. You will assign a memory address range to each of these blocks. Figure 5 shows an example of memory allocation.

![Figure 5. SRAM memory allocation]({{ "/assets/files/lab11/img/5.png" | relative_url }})

*Figure 5. SRAM memory allocation*

The memory size depends on the size of the images to be processed, so you need to estimate the image size. For example, if the largest image you want to simulate is 200x200 pixels, you need memory larger than 200 x 200 x 5 x 8 bits. In that case, the SRAM address port must be at least 18 bits wide (`2^17 < 200 x 200 x 5 < 2^18`).

## Implementation & Simulation

In this lab, you will implement the wrapper for the Canny Edge Detector.

Please login to the Olympus server and create a working directory for this lab using the following commands.

```bash
## Create and navigate to the working directory.
mkdir -p $HOME/ECEN468/Lab11/src
cd $HOME/ECEN468/Lab11/src
```

Download the tar.gz file from the lab website and extract it. In the extracted folders, you will find the following files:

- `WRAP_CANNY.v`
- `Arbiter.v`
- `systemtop.v`
- `tb.v`
- `tb_comp.v`
- `generic.sdb`
- `osu018_stdcells.v`
- `osu018_stdcells.db`

Copy them to the working directory.

Copy `WRAP_SRAM.v`, `SRAM.v`, `virtSRAM.v`, `WRAP_UART.v`, `UART_XMTR.v`, `Control_Unit.v`, `Datapath_Unit.v`, and `CannyEdge.v` from previous labs into the working directory. For the Canny Edge design, change the high threshold to 10 and the low threshold to 3.

You will need to insert code in the test bench (`tb.v`). Refer to the code for the Gaussian smoothing process, then complete the test bench for the memory operations, UART transmitter, gradient magnitude and direction, non-maximum suppression, and hysteresis thresholding.

Commands for reference:

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh
vcs -full64 tb.v   # if you use a two-dimensional array
./simv
vcs -full64 tb_comp.v
./simv
```

## Synthesis

The 256K SRAM is too large to synthesize, so we will replace `SRAM.v` with `virtSRAM.v`. This module has exactly the same input and output ports as your SRAM module.

Open `systemtop.v` and change `` `include "SRAM.v" `` to `` `include "virtSRAM.v" ``.

Run the command below to invoke Design Vision.

```bash
## Create and navigate to a separate working directory.
mkdir $HOME/ECEN468/Lab11/src/dv_Work
cd $HOME/ECEN468/Lab11/src/dv_Work

## Launch Design Vision
source /opt/coe/synopsys/syn/V-2023.12-SP1/setup.syn.sh
design_vision &
```

In Design Vision, follow the same steps as in the previous labs:

1. Link the libraries
2. Analyze
3. Elaborate
4. Compile
5. **File -> Save as** -> `systemtop_gate.v`
6. `write_sdf systemtop.sdf`
7. Copy `systemtop.sdf` and `systemtop_gate.v` to the parent directory

Gate Simulation

For gate simulation, we need `systemtop_gate.v`, `systemtop.sdf`, `osu018_stdcells.v`, and the test bench.

Open `systemtop_gate.v`; you will find the SRAM module synthesized from `virtSRAM.v`. Comment out that SRAM module, because we will use the original SRAM module from `SRAM.v` for the gate simulation.

We will use the previous test bench as a template to create a test bench for the gate simulation.

`cp tb.v gate_tb.v`

Open the new test bench `gate_tb.v` and make the following modifications:

- Make sure the following four lines are at the top of the file.

  ```verilog
  `timescale 1ns/10ps
  `include "systemtop_gate.v"
  `include "osu018_stdcells.v"
  `include "SRAM.v"
  ```

- Remove the following line if it exists.

  ```verilog
  `include "systemtop.v"
  ```

- Make sure the following two lines are at the bottom of the file (within the module).

  ```verilog
  initial
      $sdf_annotate("systemtop.sdf", systemtop_01);
  ```

- If you want to generate the waveform file, change the name of the dump file to `wave_gate.dump` in your test bench. Note that this file can be very large, so be careful when generating it; you may delete it after completing this lab to free space on your Linux server.

  `$dumpfile("wave_gate.dump");`

Run the following command to start the simulation.

```bash
source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh
vcs -full64 gate_tb.v
./simv
vcs -full64 tb_comp.v
./simv
```

If you see the error below, make changes to modules `WRAP_SRAM`, `WRAP_UART`, and `WRAP_CANNY` in `systemtop_gate.v`.

```
Err : The 32-bit expression "AddressBus" is connected to 20-bit port "AddressBus". - change "tri [19:0] AddressBus" to "tri [31:0] AddressBus";
Err : The 32-bit expression "AddressBus" is connected to 4-bit port "AddressBus". - change "tri [31:28] AddressBus" to "tri [31:0] AddressBus";
Err : The 32-bit expression "AddressBus" is connected to 16-bit port "AddressBus". - change "tri [31:16] AddressBus" to "tri [31:0] AddressBus";
```

## Submission

Please submit a single PDF file containing the following:

systemtop:

1. Screenshots of the terminal output after running `./simv`.
2. Images generated by the simulation.
3. Justification of the correctness of the results.
4. Screenshots of the code you implemented.

systemtop Gate:

1. Screenshots of the simulation output after running `./simv`.
