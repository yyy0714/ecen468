---
layout: manual
title: 'Lab 10: Design of Canny Edge Detector (Verilog)'
session: 'Week 11 (Nov 2 – Nov 6)'
report_due: 'Week 12 (Nov 9 – Nov 13)'
# downloads:
#   - label: code (tar.gz)
#     file: /assets/files/lab10/lab10_code.tar.gz
---

## 1. Objectives
- Complete RTL design of a Canny edge detector in Verilog.
- Simulate the RTL design.

---

## 2. Canny Edge Detector

### 2.1 Overview
The Canny edge detector is a widely used edge detection algorithm in computer vision and image processing. Its purpose is to identify and extract the boundaries of objects within an image by detecting significant intensity changes, which correspond to edges.

### 2.2 Processing Steps
The processing steps of the Canny edge detector is shown in figure 1. Each processing step will be explained in details in the following sections.
1. Noise Reduction
2. Gradient Calculation
3. Non-Maximum Suppression
4. Hysteresis Thresholding

### 2.3 Noise Reduction
The purpose of this step is to smooth the original image using a 2D Gaussian filter \\(GF\\). This is achieve by convolving the original image with the Gaussian filter:

$$
I(x,y) = f(x,y) * GF(x,y)
$$

Where
- \\(I(x,y)\\) denotes the smoothed image
- \\(f(x,y)\\) denotes the original image
- \\(GF(x,y)\\) denotes the Gaussian filter

In this lab, the Gaussian filter is

$$
GF(x,y) = 
\frac{1}{128}
\begin{bmatrix}
1 & 3 & 4 & 3 & 1 \\
3 & 7 & 10 & 7 & 3 \\
4 & 10 & 16 & 10 & 4 \\
3 & 7 & 10 & 7 & 3 \\
1 & 3 & 4 & 3 & 1
\end{bmatrix}
$$

### 2.4 Gradient Calculation
The purpose of gradient calculation is to measure the **magnitude** and **direction** of pixel intensity changes across the image to locate potential edge boundaries.

The horizontal gradient \\(G_x\\) and vertical gradient \\(G_y\\) can be calculated by convolving the smoothed image \\(I(x,y)\\) with two Sobel kernels \\(S_x\\) and \\(S_y\\):

$$
G_x = I * S_x, \quad G_y = I * G_y
$$

#### 2.4.1 Gradient Magnitude Calculation
The gradient magnitude image \\(M\\) can be obtained by

$$
M = \sqrt{G_x^2 + G_y^2}
$$

In this lab, the following approximation will be used for faster calculation:

$$
M = \frac{|G_x| + |G_y|}{8}
$$

#### 2.4.2 Gradient Direction Calculation
The gradient direction image \\(\theta\\) can be obtained by

$$
\theta = \operatorname{arctan2}(G_y, G_x)
$$

In this lab, the exact continuous angle will be rounded to the nearest of four standardized directions: \\(0\degree\\), \\(45\degree\\), \\(90\degree\\), or \\(135\degree\\) using the following two steps:

$$
\texttt{if}\ G_y < 0: \\
    \quad G_x = -G_x \\
    \quad G_y = -G_y
$$

$$
\begin{align*}
\texttt{if}\ G_x \geq 0 \\
    \quad \texttt{if} G_y \leq 0.5G_x \\
        \quad \quad \theta = 0 \\
    \quad \texttt{elif} 0.5G_x < G_y \leq 2.5G_x \\
        \quad \quad \theta = 45 \\
    \quad \texttt{elif}\ 2.5G_x < G_y \\
        \quad \quad \theta = 90 \\

\texttt{if}\ G_x < 0 \\
    \quad \texttt{if} G_y \leq -0.5G_x \\
        \quad \quad \theta = 0 \\
    \quad \texttt{elif} -0.5G_x < G_y \leq -2.5G_x \\
        \quad \quad \theta = 135 \\
    \quad \texttt{if} -2.5G_x < G_y \\
        \quad \quad \theta = 90
\end{align*}
$$

### 2.5 Non-Maximum Suppression
The purpose of Non-Maximum Suppression (NMS) is to thin out thick, blurry edge regions by suppressing all pixels that are not local maxima along the gradient direction, leaving sharp, one-pixel-wide lines.

The NMS image \\(N(x,y)\\) can be calcualted using the gradient magnitude image \\(M(x,y)\\) and gradient direction image \\(\theta\\):

For \\(\theta(x,y) = 0\\):

$$
N(x,y) = 
\begin{cases} 
M(x,y) & \text{if } M(x,y) \geq M(x+1, y) \text{ and } M(x,y) \geq M(x-1, y) \\ 
0 & \text{otherwise} 
\end{cases}
$$

For \\(\theta(x,y) = 45\\):

$$
N(x,y) = 
\begin{cases} 
M(x,y) & \text{if } M(x,y) \geq M(x+1, y+1) \text{ and } M(x,y) \geq M(x-1, y-1) \\ 
0 & \text{otherwise} 
\end{cases}
$$

For \\(\theta(x,y) = 90\\):

$$
N(x,y) = 
\begin{cases} 
M(x,y) & \text{if } M(x,y) \geq M(x, y+1) \text{ and } M(x,y) \geq M(x, y-1) \\ 
0 & \text{otherwise} 
\end{cases}
$$

For \\(\theta(x,y) = 135\\):

$$
N(x,y) = 
\begin{cases} 
M(x,y) & \text{if } M(x,y) \geq M(x-1, y+1) \text{ and } M(x,y) \geq M(x+1, y-1) \\ 
0 & \text{otherwise} 
\end{cases}
$$


### 2.6 Hysteresis Thresholding

## 3. Implementation
The block diagram of a Canny edge detector and testbench is shown in figure 2. 

Figure 1 shows the overall image-processing flow.

![Canny Edge Detector Workflow]({{ "/assets/files/lab10/img/workflow.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 1. Canny Edge Detector Workflow*

Figure 2 shows the system that you will implement in this lab.

![System Overview]({{ "/assets/files/lab10/img/system_overview.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 2. System Overview (clock and reset signals are ommited)*

In the equations below, \\(b_i\\) denotes `buf_x[i]` and \\(z_i\\) denotes `buf_z[i]` (the pixel windows written in by the testbench), and `buf_y[6]` holds the direction code \\(\theta\\). Each stage writes the result register shown; the module spreads the arithmetic over a few clock cycles.

### 2.3 Blurred Image
This stage blurs the input image to suppress noise, preventing small intensity fluctuations from being mistaken for edges in later stages. It is computed as a \\(5 \times 5\\) Gaussian smoothing with the kernel `gf`, whose weights sum to 128 (so the division becomes a right shift by 7):

$$
\texttt{tmp_1} = \frac{\sum_{i=0}^{24} b_i \cdot \texttt{gf}[i]}{128}
$$

## 2.2 Gradient Image
This stage measures the edge strength at each pixel — how sharply the brightness changes — so that likely edge pixels can be identified. Sobel operators produce the gradients \\(G_x\\) and \\(G_y\\), and their magnitude is approximated by \\(|G_x| + |G_y|\\) instead of \\(\sqrt{G_x^2 + G_y^2}\\), then scaled by \\(\tfrac{1}{8}\\):

$$
G_x = (b_2 + 2b_7 + b_{12}) - (b_0 + 2b_5 + b_{10})
$$

$$
G_y = (b_0 + 2b_1 + b_2) - (b_{10} + 2b_{11} + b_{12})
$$

$$
\texttt{tmp_2} = \frac{|G_x| + |G_y|}{8}
$$

## 2.3 Direction Image
This stage determines the orientation of each edge — the direction of steepest brightness change — which the next stage needs in order to compare each pixel against the correct neighbors. The gradient angle \\(\theta = \operatorname{atan2}(G_y, G_x)\\) is quantized into four codes. Because orientation repeats every 180°, the gradient is first folded so that \\(G_y \ge 0\\): if \\(G_y < 0\\), it is replaced by \\((-G_x, -G_y)\\). The folded \\(G_x\\) and \\(G_y\\) then give:

$$
\texttt{tmp_3} =
\begin{cases}
0   & |G_y| \le \dfrac{1}{2}|G_x| \\
45  & G_x \ge 0 \ \text{and}\ |G_y| \le \dfrac{5}{2}|G_x| \\
135 & G_x < 0 \ \text{and}\ |G_y| \le \dfrac{5}{2}|G_x| \\
90  & \text{otherwise}
\end{cases}
$$

The thresholds \\(\tfrac{1}{2}\\) and \\(\tfrac{5}{2}\\) approximate \\(\tan 22.5^\circ\\) and \\(\tan 67.5^\circ\\). The colors in the displayed image only visualize these codes.

## 2.4 Non-Maximum Suppression (NMS) Image
This stage thins the thick gradient response into edges about one pixel wide, by keeping a pixel only where it is a local maximum along the gradient. The center pixel is \\(b_6\\), and its two neighbors along the gradient are \\(b_{6-n}\\) and \\(b_{6+n}\\), where \\(n = 5\,dy + dx\\) is taken from the direction:

$$
(dx, dy) =
\begin{cases}
(1, 0)  & \theta = 0 \\
(1, -1) & \theta = 45 \\
(0, 1)  & \theta = 90 \\
(1, 1)  & \theta = 135
\end{cases}
$$

The window is copied into `tmp_4`. If \\(b_6 \ge b_{6-n}\\) and \\(b_6 \ge b_{6+n}\\), the two neighbors `tmp_4[6-n]` and `tmp_4[6+n]` are cleared to 0; otherwise the center `tmp_4[6]` is cleared to 0.

## 2.5 Hysteresis Image
This stage produces the final binary edge map: it keeps strong pixels as edges, discards very weak pixels, and keeps in-between pixels only when they connect to a strong edge. Two thresholds are used, `THRESHOLD_UPPER = 10` and `THRESHOLD_LOWER = 3`:

$$
\texttt{tmp_5} =
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
