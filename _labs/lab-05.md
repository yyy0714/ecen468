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

## 1. Objectives
- Integrate Canny edge detector, SRAM, UART transmitter and system bus in SystemC.
- Build and simulate the design.

---

## 2. Combining the edge detector with the system bus
![Figure 1. Block diagram of the system]({{ "/assets/files/lab05/img/1.png" | relative_url }})

*Figure 1. Block diagram of the system*

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

---

## 3. Reconfiguration of SRAM
We will expand the memory to support the Canny Edge Detector. Figure 5 shows the corresponding memory map. The detector needs space to store several intermediate results: the original image, the noise-reduced image, the gradient information, the direction information, the image after non-maximum suppression, and the final image after hysteresis thresholding. You will assign a memory address range to each of these blocks. Figure 5 shows an example of memory allocation.

![Figure 5. SRAM memory allocation]({{ "/assets/files/lab05/img/5.png" | relative_url }})

*Figure 5. SRAM memory allocation*

The memory size depends on the size of the images to be processed, so you need to estimate the image size. For example, if the largest image you want to simulate is 200x200 pixels, you need memory larger than 200 x 200 x 5 x 8 bits. In that case, the SRAM address port must be at least 18 bits wide (`2^17 < 200 x 200 x 5 < 2^18`).

---

## 4. Lab Procedures

### 4.0 Setup
1. Execute the following commands to create and enter the working directory.
    - `cd $HOME/ecen468/`

2. Download `lab05_code.tar.gz` from the lab website and put it in the working directory.

3. Execute the following commands to extract the files.
    - `tar -xvf lab05_code.tar.gz`
    - `rm lab05_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `Arbiter.cpp`
    - `Canny_Edge_WRAP.cpp`
    - `main.cpp`
    - `test.cpp`
    - `Arbiter.h`
    - `Canny_Edge_WRAP.h`
    - `test.h`
    - `workspace` (directory where Siemens Vista will be run)
        - `0.ref.bmp`
        - `1.ref.bmp`
        - `2.ref.bmp`
        - `3.ref.bmp` 
        - `4.ref.bmp`
        - `5.ref.bmp`
        - `6.ref.bmp`
        - `cman_200.bmp`

5. Copy the following files from lab 3 into the working directory.
    - `SRAM_WRAP.cpp`
    - `SRAM.cpp`
    - `UART_XMTR_WRAP.cpp`
    - `UART_XMTR.cpp`
    - `SRAM_WRAP.h`
    - `SRAM.h`
    - `UART_XMTR_WRAP.h`
    - `UART_XMTR.h`

6. Copy the following files from lab 4 into the working directory.
    - `Canny_Edge.cpp`
    - `Canny_Edge.h`

7. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

### 4.1 Building SystemC Design Using Siemens Vista
1. Execute the following commands in sequence to open Siemens Vista.
    - `source /opt/coe/mentorgraphics/vista/2024_2/setup.vista.linux.bash`
    - `source /opt/rh/gcc-toolset-13/enable`
    - `cd $HOME/ecen468/lab03/workspace`
    - `vista &`

2. In Vista graphical interface, click **Project** on the menu bar, then click **New Project...**.

3. In the **New Project** dialog, click **Save** (You don't need to change the default file name).

4. In the **Create New Project** dialog, click **Files** on the tab bar, then click **Add Files** on the bottom right.

5. In the **Select Files** dialog, click the folder icon and select the C++ source code and header files, click **Open**, then click **OK**.

6. On the side bar, right click on **Project** and click **Build**. Every time you modify the source code or header files, you need to rebuild the project.

7. If no errors occur, proceed to the next section.

### 4.2 Simulating SystemC Design Using Siemens Vista
1. On the side bar, click the **+** next to **Project** expand its content.

2. Click the **+** next to **Simulation**, then right click on **main.cpp** and click **Simulate**.

3. In the **simulation** dialog, deselect **Stop after elaboration** under **Debugging**, then click **OK**.

4. Take a screenshot of the simulation output in Vista for the lab report.

---

## 5. Submission

Please submit a single PDF file containing the following:

1. Screenshots of the images generated by the simulation.
2. Screenshot of the simulation output in Vista.
3. Screenshots or copy of the content of the following files:
    - `Canny_Edge_WRAP.cpp`

