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

In the equations below, \\(b_i\\) denotes `buf_x[i]` and \\(z_i\\) denotes `buf_z[i]` (the pixel windows written in by the testbench), and `buf_y[6]` holds the direction code \\(\theta\\). Each stage writes the result register shown; the module spreads the arithmetic over a few clock cycles.

## 2.1 Blurred Image
Gaussian smoothing with the \\(5 \times 5\\) kernel `gf` (its weights sum to 128, so the division is a right shift by 7):

$$
\text{tmp\_1} = \left( \sum_{i=0}^{24} b_i \cdot \text{gf}[i] \right) \gg 7
$$

## 2.2 Gradient Image
Sobel gradients, then a magnitude approximated as \\(|G_x| + |G_y|\\) (instead of \\(\sqrt{G_x^2 + G_y^2}\\)) and scaled by \\(\tfrac{1}{8}\\):

$$
G_x = (b_2 + 2b_7 + b_{12}) - (b_0 + 2b_5 + b_{10})
$$

$$
G_y = (b_0 + 2b_1 + b_2) - (b_{10} + 2b_{11} + b_{12})
$$

$$
\text{tmp\_2} = \big(|G_x| + |G_y|\big) \gg 3
$$

## 2.3 Direction Image
Quantize the gradient direction \\(\theta = \operatorname{atan2}(G_y, G_x)\\) into four codes. First fold to \\(G_y \ge 0\\): if \\(G_y < 0\\), replace \\((G_x, G_y)\\) with \\((-G_x, -G_y)\\). Then, using the folded \\(G_x, G_y\\):

$$
\text{tmp\_3} =
\begin{cases}
0   & |G_y| \le |G_x|/2 \\
45  & G_x \ge 0 \ \text{and}\ |G_y| \le 5|G_x|/2 \\
135 & G_x < 0 \ \text{and}\ |G_y| \le 5|G_x|/2 \\
90  & \text{otherwise}
\end{cases}
$$

The thresholds \\(\tfrac{1}{2}\\) and \\(\tfrac{5}{2}\\) approximate \\(\tan 22.5^\circ\\) and \\(\tan 67.5^\circ\\). The colors in the displayed image only visualize these codes.

## 2.4 Non-Maximum Suppression (NMS) Image
Keep a magnitude pixel only if it is a local maximum along the gradient (this thins edges to about one pixel). The center is \\(b_6\\); its two neighbors along the gradient are \\(b_{6-n}\\) and \\(b_{6+n}\\), with \\(n = 5\,dy + dx\\) selected from the direction:

$$
(dx, dy) =
\begin{cases}
(1, 0)  & \theta = 0 \\
(1, -1) & \theta = 45 \\
(0, 1)  & \theta = 90 \\
(1, 1)  & \theta = 135
\end{cases}
$$

Copy the window into `tmp_4`; then if \\(b_6 \ge b_{6-n}\\) **and** \\(b_6 \ge b_{6+n}\\), set \\(\text{tmp\_4}[6-n] = \text{tmp\_4}[6+n] = 0\\); otherwise set \\(\text{tmp\_4}[6] = 0\\).

## 2.5 Hysteresis Image
Final edge map from double thresholds `THRESHOLD_UPPER = 10` and `THRESHOLD_LOWER = 3`:

$$
\text{tmp\_5} =
\begin{cases}
1 & b_6 \ge \text{UPPER} \\
0 & b_6 \le \text{LOWER} \\
c & \text{otherwise (candidate)}
\end{cases}
$$

A candidate is kept (\\(c = 1\\)) if an along-edge neighbor is already strong or already an edge, otherwise \\(c = 0\\):

$$
c = (b_{6-n} \ge \text{UPPER}) \ \text{or}\ (b_{6+n} \ge \text{UPPER}) \ \text{or}\ (z_{6-n} = 1) \ \text{or}\ (z_{6+n} = 1)
$$

The offset uses the **along-edge** direction (perpendicular to the gradient), \\(n = 5\,dy + dx\\) with \\((dx, dy) = (0,1), (1,1), (1,0), (1,-1)\\) for \\(\theta = 0, 45, 90, 135\\). `buf_z` (\\(z_i\\)) is the edge map computed so far, supplied by the testbench.

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
