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

Each of the five stages is one **operation** of a small state machine inside the module. While an operation is enabled, the internal `state` register counts clock cycles — it starts at `0`, advances one step per clock, and returns to `0` when the operation ends — so a stage's work is spread across a few `state` steps. The pixel window is held in `buf_x` (and, where noted, `buf_y`/`buf_z`), and each stage writes its result to one of `tmp_1`–`tmp_5`, which the testbench reads back.

## 2.1 Blurred Image
Image 1 is Image 0 smoothed with a \\(5 \times 5\\) Gaussian filter `gf` (its weights sum to 128) to suppress noise so it does not become false edges:

$$
I_1 = G * I_0 = \left( \sum_{i=0}^{24} b_i \, g_i \right) \gg 7
$$

where \\(b_i =\\) `buf_x[i]` and \\(g_i =\\) `gf[i]`; dividing the weighted sum by 128 is a right shift by 7.

**States (`MODE_GAUSSIAN`):**
- `state 0`: accumulate the 25-term weighted sum into `gaussian_sum`.
- `state 1`: shift right by 7 and store the blurred pixel in `tmp_1`.
- `state 2`: hold the result.

## 2.2 Gradient Image
Image 2 is the edge strength of Image 1. Two \\(3 \times 3\\) Sobel operators give the horizontal and vertical gradients, \\(G_x = S_x * I_1\\) and \\(G_y = S_y * I_1\\). The exact magnitude \\(\sqrt{G_x^2 + G_y^2}\\) needs a square root, so the design uses the cheaper \\(L_1\\) (Manhattan) approximation, scaled down by 8:

$$
I_2 = \frac{|G_x| + |G_y|}{8} = \big(|G_x| + |G_y|\big) \gg 3
$$

**States (`MODE_SOBEL`, magnitude):**
- `state 0`: compute the Sobel gradients `gradient_x` and `gradient_y`.
- `state 1`: form \\(|G_x| + |G_y|\\), shift right by 3, and store the magnitude in `tmp_2`.

The same operation continues into `state 2`–`state 3` to compute the direction (Section 2.3).

## 2.3 Direction Image
Image 3 is the gradient direction that NMS needs. Instead of computing \\(\theta = \operatorname{atan2}(G_y, G_x)\\), which requires an arctangent, the design **quantizes** the direction into four orientations — `0`, `45`, `90`, `135` (degrees) — using only comparisons and shifts on \\(G_x, G_y\\).

**States (`MODE_SOBEL`, direction):**
- `state 2`: because orientation repeats every 180°, fold the gradient so its vertical part is non-negative (negate both \\(G_x, G_y\\) when \\(G_y < 0\\)); keep the folded pair in `abs_gradient_x`/`abs_gradient_y`.
- `state 3`: choose the orientation by comparing \\(|G_y|\\) against \\(\approx \tfrac{1}{2}|G_x|\\) and \\(\approx \tfrac{5}{2}|G_x|\\) (the 22.5° and 67.5° sector boundaries) and store the code in `tmp_3`.

The colored direction image is only a visualization of these four codes and is produced by the testbench.

## 2.4 Non-Maximum Suppression (NMS) Image
Image 4 thins the edges: a magnitude pixel is kept only if it is a local maximum **along the gradient direction**, otherwise it is set to 0. This reduces thick gradient ridges to edges about one pixel wide. The stage uses the magnitude window in `buf_x` (center = current pixel) and the direction in `buf_y`.

**States (`MODE_NMS`):**
- `state 0`: convert the direction code into the offset `(dx, dy)` that selects the two neighbors lying along the gradient.
- `state 1`: if the center pixel is \\(\ge\\) both neighbors, keep it and zero the two neighbors; otherwise zero the center. The resulting window is stored in `tmp_4`.

## 2.5 Hysteresis Image
Image 5 is the final binary edge map, produced by double-threshold hysteresis on the NMS magnitude (`buf_x`) using a high and a low threshold (`THRESHOLD_UPPER = 10`, `THRESHOLD_LOWER = 3`):

- magnitude \\(\ge\\) high threshold → strong edge (`1`);
- magnitude \\(\le\\) low threshold → discarded (`0`);
- in between → a candidate, kept (`1`) only if it connects to an edge, else `0`.

Connectivity is checked **along the edge** (perpendicular to the gradient): a candidate is kept if an along-edge neighbor is already strong, or is already marked as an edge in the running edge map `buf_z` (which holds Image 5 so far and is supplied by the testbench each pixel).

**States (`MODE_HYSTERESIS`):**
- `state 0`: convert the direction code into the along-edge offset `(dx, dy)`.
- `state 1`: apply the threshold-and-connectivity rule and store the 1-bit result in `tmp_5`.

The result is the final Canny edge map.

## 2.6 Implementation Requirements
- Implement Canny edge detector in `canny_edge_detector.v`.
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

## 3.1 Simulating Canny Edge Detector RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab10/sim`
    - `vcs -full64 ../tb/canny_edge_detector_tb.v -o simv_canny_edge_detector_tb`

2. Execute the following command to run the simulation.
    - `./simv_canny_edge_detector_tb`

3. Execute the following command to compare the generated images with the reference images.
    - `python compare.py`

---

# 4. Submission
Please submit a single PDF file containing the following:

- Canny edge detector RTL design:
    1. Screenshot of the terminal output after running `./simv_canny_edge_detector_tb`.
    2. Screenshots or copy of the content of the following files:
        - `canny_edge_detector.v`
        - `system_bus_canny_edge_detector_wrapper.v`
