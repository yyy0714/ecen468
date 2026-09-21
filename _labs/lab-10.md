---
layout: manual
title: 'Lab 10: Design of Canny Edge Detector (Verilog)'
session: 'Week 11 (Nov 2 – Nov 6)'
report_due: 'Week 12 (Nov 9 – Nov 13)'
# downloads:
#   - label: code (tar.gz)
#     file: /assets/files/lab10/lab10_code.tar.gz
---

# 1. Objectives
- Complete RTL design of a Canny edge detector in Verilog.
- Simulate the RTL design.

---

# 2. Introduction
Canny edge detection is a multi-stage image-processing algorithm that identifies object boundaries while reducing the effects of noise and weak intensity variations. In this lab, the input is a \\(200 \times 200\\) grayscale image (Image 0), and the successive processing stages generate intermediate images that can be stored and observed through the testbench.

Figure 1 shows the overall image-processing flow.

![Canny Edge Detector Workflow]({{ "/assets/files/lab10/img/workflow.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 1. Canny Edge Detector Workflow*

Figure 2 shows the system that you will implement in this lab.

![System Overview]({{ "/assets/files/lab10/img/system_overview.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 2. System Overview (clock and reset signals are ommited)*

## 2.1 Blurred Image
Image 1 is obtained by applying a \(5 \times 5\) Gaussian filter (`gf`) to the original image. The purpose of this stage is to reduce high-frequency noise and small intensity variations that could otherwise generate false edges in later stages.

Conceptually,

$$
I_1(x,y) = G(x,y) * I_0(x,y)
$$

where \(G\) is the Gaussian kernel and \(*\) denotes convolution.

## 2.2 Gradient Image
Image 2 is generated from the blurred image by calculating the image-intensity gradients in the horizontal and vertical directions. The design uses two \(3 \times 3\) Sobel filters (`sobel_x` and `sobel_y`) to obtain

$$
G_x = S_x * I_1 \qquad
G_y = S_y * I_1
$$

The gradient magnitude indicates how strongly the image intensity changes at each pixel:

$$
G = \sqrt{G_x^2 + G_y^2}
$$

Pixels with large gradient magnitude are therefore potential edge pixels.

## 2.3 Direction Image
Image 3 represents the gradient direction at each pixel. It is calculated from the horizontal and vertical gradient components using

$$
\theta = \operatorname{atan2}(G_y, G_x)
$$

The direction determines the orientation of the local intensity change and is required by the non-maximum suppression stage. In the displayed image, different colors are used to visualize different gradient directions; the colors themselves are only a representation of the direction information.

## 2.4 Non-Maximum Suppression (NMS) Image
Image 4 is produced by non-maximum suppression (NMS) using the gradient magnitude from Image 2 and the gradient direction from Image 3. For each pixel, its gradient magnitude is compared with neighboring pixels along the gradient direction. The pixel is retained only if it is a local maximum.

This process suppresses weaker responses around an edge and reduces thick gradient regions to thin edge candidates, ideally close to one pixel wide.

## 2.5 Hysteresis Image
Image 5 is the final edge image produced by double-threshold hysteresis. The NMS result is compared with a high and a low threshold:
- Pixels above the high threshold are classified as strong edges.
- Pixels below the low threshold are rejected.
- Pixels between the two thresholds are classified as weak edges and are retained when they are connected to strong edges.

This stage removes isolated weak responses while preserving meaningful, continuous edges. The resulting Image 5 is the final Canny edge map.

## 2.6 Implementation Requirements
- Implement Canny edge detector module in `canny_edge_detector.v`.
- Implement system bus Canny edge detector wrapper in `system_bus_canny_edge_detector_wrapper.v`.

---

# 3. Lab Procedure

## 3.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `mkdir -p $HOME/ecen468/lab10/`
  - `cd $HOME/ecen468/lab10/`

2. Download `lab10_code.tar.gz` from the lab website and put it the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab10_code.tar.gz`
  - `rm lab10_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `lib` (directory for technology libraries)
        - `osu018_stdcells.db`
        - `osu018_stdcells.v`
        - `generic.sdb`
    - `rtl` (directory for RTL verilog code)
        - `canny_edge_detector.v`
        - `system_bus_canny_edge_detector_wrapper.v`
    - `sim` (directory where Synopsys VCS will be run)
    - `syn` (directory where Synopsys Design Vision will be run)
        - `netlist` (directory for gate-level verilog code)
        - `sdf` (directory for SDF files)
    - `tb` (directory for testbenches)
        - `canny_edge_detector_tb.v`

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

## 3.1 Simulating System Bus RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab10/sim`
    - `vcs -full64 ../tb/canny_edge_detector_tb.v -o simv_canny_edge_detector_tb`

2. Execute the following command to run the simulation.
    - `./simv_canny_edge_detector_tb`

3. Take a screenshot of the terminal outputs of the simulation.

## 3.2 [Optional] Synthesizing Canny Edge Detector RTL Design Using Synopsys Design Vision
1. Execute the following commands in sequence to open Design Vision:
    - `source /opt/coe/synopsys/syn/V-2023.12-SP1/setup.syn.sh`
    - `cd $HOME/ecen468/lab10/syn`
    - `design_vision &`

2. Execute the following commands to set up Design Vision.
    - `set_app_var link_path ../lib/osu018_stdcells.db`
    - `set_app_var target_library ../lib/osu018_stdcells.db`
    - `set_app_var symbol_library ../lib/osu018_stdcells.db`

3. Execute the following command to analyze the design.
    - `analyze -format verilog {../rtl/canny_edge_detector.v}`

4. Execute the following command to elaborate the design.
    - `elaborate canny_edge_detector`

5. Execute the following command to synthesize the design.
    - `compile -exact_map`

6. Execute the following command to save the optimized netlist.
    - `write -hierarchy -format verilog -output ./netlist/canny_edge_detector.v`

7. Execute the following command to save the Standard Delay Format (SDF) file.
    - `write_sdf ./sdf/canny_edge_detector.sdf`

8. Exit Design Vision.

## 3.3 [Optional] Simulating Canny Edge Detector Gate-Level Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab09/sim`
    - `vcs -full64 ../tb/top_netlist_tb.v -o simv_top_netlist_tb`

2. Execute the following command to run the simulation.
    - `./simv_top_netlist_tb`

3. Take a screenshot of the terminal outputs of the simulation.

---

# 4. Submission
Please submit a single PDF file containing the following:

Canny edge detector RTL design:
1. Screenshot of the terminal output after running `./simv_canny_edge_detector_tb`.
2. Justification of the simulation results.
3. Screenshots or copy of the content of the following files:
    - `system_bus_uart_tx_wrapper.v`
    - `system_bus_sram_wrapper.v`
