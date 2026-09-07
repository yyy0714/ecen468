---
layout: manual
title: 'Lab 5: Combining Canny Edge Detector with System Bus, SRAM and UART (SystemC)'
session: 'Week 6 (Sep 28 – Oct 2)'
report_due: 'Week 7 (Oct 5 – Oct 9)'
manual_pdf: /assets/files/lab05/lab05_manual.pdf
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab05/lab05_code.tar.gz
---

## Objectives

In this lab, we will combine the Canny Edge Detector, SRAM, and UART through the system bus, using Vista for simulation and verification.

![Figure 1. Block diagram of the system]({{ "/assets/files/lab05/img/1.png" | relative_url }})

*Figure 1. Block diagram of the system*

## Combining the edge detector with the system bus

In Lab 4, the image data moved between the test bench and the modules, and the large internal memory required for the image was declared in the test bench. In this lab, we will use the SRAM to store the image. Figure 1 shows the complete system. The test bench now handles only BMP image input/output and control logic, and it still keeps internal memory for reading and writing BMP files.

![Figure 2. Connection of the Canny Edge Detector and its decoder]({{ "/assets/files/lab05/img/2.png" | relative_url }})

*Figure 2. Connection of the Canny Edge Detector and its decoder*

![Figure 3. Address map of the Canny Edge Detector]({{ "/assets/files/lab05/img/3.png" | relative_url }})

*Figure 3. Address map of the Canny Edge Detector*

Figure 2 shows the signals connecting the Canny Edge Detector module and its decoder. To attach the module to the system bus, an address decoder (wrapper) is required to decode the control signals. Figure 3 shows the address map. The four most significant bits hold the device identification number; we assume the ID of the Canny Edge Detector is `0100`. The remaining bits carry control and address signals.

Figure 4 shows an example of the noise reduction process. First, the test bench sends all the data to memory. Then only the data needed for processing is fetched from memory to the test bench and passed to the Canny Edge Detector module. After the detector finishes processing the block, the test bench reads the result and writes it back to the appropriate memory.

![Figure 4. Data flow of Noise Reduction]({{ "/assets/files/lab05/img/4.png" | relative_url }})

*Figure 4. Data flow of Noise Reduction*

For debugging, the system sends several messages through the UART to indicate the completion of each stage (noise reduction, gradient extraction, non-maximum suppression, and hysteresis thresholding).

The messages below should be sent through UART:

- Noise reduction: `'N'` = `0x4E`
- Gradient extraction: `'G'` = `0x47`
- Non-maximum suppression: `'S'` = `0x53`
- Hysteresis thresholding: `'H'` = `0x48`

## Reconfiguration of SRAM

We will expand the memory to support the Canny Edge Detector. Figure 5 shows the corresponding memory map. The detector needs space to store several intermediate results: the original image, the noise-reduced image, the gradient information, the direction information, the image after non-maximum suppression, and the final image after hysteresis thresholding. You will assign a memory address range to each of these blocks. Figure 5 shows an example of memory allocation.

![Figure 5. SRAM memory allocation]({{ "/assets/files/lab05/img/5.png" | relative_url }})

*Figure 5. SRAM memory allocation*

The memory size depends on the size of the images to be processed, so you need to estimate the image size. For example, if the largest image you want to simulate is 200x200 pixels, you need memory larger than 200 x 200 x 5 x 8 bits. In that case, the SRAM address port must be at least 18 bits wide (`2^17 < 200 x 200 x 5 < 2^18`).

## Implementation & Simulation

In this lab, you will implement the wrapper for the Canny Edge Detector.

Please login to the Olympus server and create a working directory for this lab using the following commands.

```bash
## Create and navigate to the working directory.
mkdir -p $HOME/ECEN468/Lab5/src
cd $HOME/ECEN468/Lab5/src
```

Download the tar.gz file from the lab website and extract it. In the extracted folders, you will find the following files:

- `Canny_Edge_WRAP.cpp`
- `Canny_Edge_WRAP.h`
- `Arbiter.cpp`
- `Arbiter.h`
- `test.cpp`
- `test.h`
- `main.cpp`

Copy them to the working directory. Also copy `SRAM_WRAP.cpp`, `SRAM_WRAP.h`, `SRAM.cpp`, `UART_XMTR.cpp`, `UART_XMTR.h`, `UART_XMTR_WRAP.cpp`, and `UART_XMTR_WRAP.h` from Lab 3, and `Canny_Edge.cpp` and `Canny_Edge.h` from Lab 4. Figure 6 shows the file hierarchy for this lab.

![Figure 6. Hierarchy of the files]({{ "/assets/files/lab05/img/6.png" | relative_url }})

*Figure 6. Hierarchy of the files*

Simulating a 200 x 200 BMP file can take several minutes, so you may want to start with a smaller file (`kodim23_50.bmp`) for faster debugging. To switch the input image, change the input file name in `test.cpp`. You also need to change the variable `dDummyData`: use `dDummyData` = 0 for a 200 x 200 BMP and `dDummyData` = 2 for a 50 x 50 BMP.

Once you complete the implementation, verify your design by simulation. Take screenshots of the simulation output and include them in your report.

Commands for reference:

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/mentorgraphics/vista/2024_2/setup.vista.linux.bash
vista &

source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh
wv &
```

## Submission

Please submit a single PDF file containing the following:

1. Screenshots of the output images with analysis.
2. Screenshots of the simulation output in Vista.
3. Screenshots of your code in this design with reasonable comments.
