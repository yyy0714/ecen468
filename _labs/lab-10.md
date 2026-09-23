---
layout: manual
title: 'Lab 10: Design of Canny Edge Detector (Verilog)'
session: 'Week 5 (Sep 21 – Sep 25)'
report_due: 'Week 6 (Sep 28 – Oct 2)'
# session: 'Week 11 (Nov 2 – Nov 6)'
# report_due: 'Week 12 (Nov 9 – Nov 13)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab10/lab10_code.tar.gz
---

## 1. Objectives
- Understand the mechanism of Canny edge detector.
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

![Canny Edge Detector Workflow]({{ "/assets/files/lab10/img/canny_edge_detector_workflow.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 1. Canny Edge Detector Workflow*

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
G_x = I * S_x, \quad G_y = I * S_y
$$

In this lab, the Sobel kernels are

$$
S_x =
\begin{bmatrix}
-1 & 0 & +1 \\
-2 & 0 & +2 \\
-1 & 0 & +1
\end{bmatrix}

\quad

S_y =
\begin{bmatrix}
+1 & +2 & +1 \\
0 & 0 & 0 \\
-1 & -2 & -1
\end{bmatrix}
$$

Below is an example of the convolution:

$$
I = \begin{bmatrix}
0 & 0 & 30 & 100 & 100 \\
0 & 0 & 30 & 100 & 100 \\
0 & 0 & 30 & 100 & 100 \\
0 & 0 & 30 & 100 & 100 \\
0 & 0 & 30 & 100 & 100
\end{bmatrix}

\quad

G_x = \begin{bmatrix}
\cdot & \cdot & \cdot & \cdot & \cdot \\
\cdot & 120 & 400 & 280 & \cdot \\
\cdot & 120 & 400 & 280 & \cdot \\
\cdot & 120 & 400 & 280 & \cdot \\
\cdot & \cdot & \cdot & \cdot & \cdot
\end{bmatrix}

\quad

G_y = \begin{bmatrix}
\cdot & \cdot & \cdot & \cdot & \cdot \\
\cdot & 0 & 0 & 0 & \cdot \\
\cdot & 0 & 0 & 0 & \cdot \\
\cdot & 0 & 0 & 0 & \cdot \\
\cdot & \cdot & \cdot & \cdot & \cdot
\end{bmatrix}
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

In this lab, the exact continuous angle will be rounded to the nearest of four standardized directions: \\(0^\circ\\), \\(45^\circ\\), \\(90^\circ\\), or \\(135^\circ\\) using the following logic:


| \\(G_x\\) Condition | \\(G_y\\) Condition               | Resulting Direction |
| :------------------ | :-------------------------------- | :-----------------: |
| \\(G_x \ge 0\\)     | \\(G_y \le 0.5 G_x\\)             | \\(0^\circ\\)       |
|                     | \\(0.5 G_x < G_y \le 2.5 G_x\\)   | \\(45^\circ\\)      |
|                     | \\(G_y > 2.5 G_x\\)               | \\(90^\circ\\)      |
| \\(G_x < 0\\)       | \\(G_y \le -0.5 G_x\\)            | \\(0^\circ\\)       |
|                     | \\(-0.5 G_x < G_y \le -2.5 G_x\\) | \\(135^\circ\\)     |
|                     | \\(G_y > -2.5 G_x\\)              | \\(90^\circ\\)      |

### 2.5 Non-Maximum Suppression
The purpose of Non-Maximum Suppression (NMS) is to thin out thick, blurry edge regions by suppressing all pixels that are not local maxima along the gradient direction, leaving sharp, one-pixel-wide lines.

The NMS image \\(N(x,y)\\) can be calcualted using the gradient magnitude image \\(M(x,y)\\) and gradient direction image \\(\theta(x,y)\\):

For \\(\theta(x,y) = 0^\circ\\):

$$
N(x,y) = 
\begin{cases} 
    M(x,y) & \text{if } M(x,y) \geq M(x+1, y) \text{ and } M(x,y) \geq M(x-1, y) \\ 
    0 & \text{otherwise}
\end{cases}
$$

For \\(\theta(x,y) = 45^\circ\\):

$$
N(x,y) = 
\begin{cases} 
    M(x,y) & \text{if } M(x,y) \geq M(x+1, y+1) \text{ and } M(x,y) \geq M(x-1, y-1) \\ 
    0 & \text{otherwise}
\end{cases}
$$

For \\(\theta(x,y) = 90^\circ\\):

$$
N(x,y) = 
\begin{cases} 
    M(x,y) & \text{if } M(x,y) \geq M(x, y+1) \text{ and } M(x,y) \geq M(x, y-1) \\ 
    0 & \text{otherwise}
\end{cases}
$$

For \\(\theta(x,y) = 135^\circ\\):

$$
N(x,y) = 
\begin{cases} 
    M(x,y) & \text{if } M(x,y) \geq M(x-1, y+1) \text{ and } M(x,y) \geq M(x+1, y-1) \\ 
    0 & \text{otherwise}
\end{cases}
$$

#### 2.6 Hysteresis Thresholding
The purpose of hysteresis thresholding is to isolate true edge pixels from background noise by keeping weak edge pixels only if they are connected to strong edge pixels. The process consists of classifying pixels into three categories based on two threshold values \\(T_{\text{low}}\\) and \\(T_{\text{high}}\\) (**double thresholding**) and resolving the "weak" pixels by checking their two neighbor pixels on their edge directions (**hysteresis tracking**).

#### 2.6.1 Double Thresholding
A preliminary edge image \\(E_{\text{pre}}(x,y)\\) can be obtained by

$$
E_{\text{pre}}(x,y) = 
\begin{cases} 
    \text{Strong} & \text{if } N(x,y) \geq T_{\text{high}} \\ 
    \text{Weak} & \text{if } T_{\text{low}} < N(x,y) < T_{\text{high}} \\ 
    0 & \text{if } N(x,y) \leq T_{\text{low}} 
\end{cases}
$$

Where
- \\(T_{\text{high}}\\) denotes the high threshold
- \\(T_{\text{low}}\\) denotes the low threshold

#### 2.6.2 Hysteresis Tracking
The final edge image \\(E_{\text{final}}(x,y)\\) can be obtained by

$$
E_{\text{final}}(x, y) = \begin{cases} 
1 & \text{if } \alpha(x, y) = 0^\circ \text{ and } \big(E_{\text{pre}}(x, y-1) = \text{Strong} \lor E_{\text{pre}}(x, y+1) = \text{Strong}\big) \\ 
1 & \text{if } \alpha(x, y) = 90^\circ \text{ and } \big(E_{\text{pre}}(x-1, y) = \text{Strong} \lor E_{\text{pre}}(x+1, y) = \text{Strong}\big) \\ 
1 & \text{if } \alpha(x, y) = 45^\circ \text{ and } \big(E_{\text{pre}}(x-1, y+1) = \text{Strong} \lor E_{\text{pre}}(x+1, y-1) = \text{Strong}\big) \\ 
1 & \text{if } \alpha(x, y) = 135^\circ \text{ and } \big(E_{\text{pre}}(x-1, y-1) = \text{Strong} \lor E_{\text{pre}}(x+1, y+1) = \text{Strong}\big) \\ 
0 & \text{otherwise} 
\end{cases}
$$

Where \\(\alpha = \theta + 90^\circ\\)

---

## 3. Implementation

### 3.1 Block Diagram
The block diagram of a Canny edge detector and testbench is shown in figure 2.

![System Overview]({{ "/assets/files/lab10/img/system_overview.png" | relative_url }}){: style="zoom: 0.6;" }

*Figure 2. System Overview (clock and reset signals are ommited)*

### 3.2 Noise Reduction Implementation Example
The workflow of noise reduction mentioned in section 2.3 is shown in figure 3. During every compute cycle, a \\(5 \times 5\\) image window is read from the original image (`img_0`) into `buf_x` to be convolved with the \\(5 \times 5\\) Gaussian filter `gf`. The scalar result will be stored in `tmp_1` before being written into the smoothed image (`img_1`).

![Noise Reduction Workflow]({{ "/assets/files/lab10/img/noise_reduction_workflow.png" | relative_url }}){: style="zoom: 0.6;"}

*Figure 3. Noise Reduction Workflow*

### 3.3 Requirements
- Implement Canny edge detector in `canny_edge_detector.v`.

---

## 4. Lab Procedure

### 4.0 Setup
1. Execute the following commands to create and enter the working directory.
  - `mkdir -p $HOME/ecen468/lab10/`
  - `cd $HOME/ecen468/lab10/`

2. Download `lab10_code.tar.gz` from the lab website and put it the working directory.

3. Execute the following commands to extract the files.
  - `tar -xvf lab10_code.tar.gz`
  - `rm lab10_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `images` (directory for images)
        - `generated` (directory for generated images)
        - `reference` (directory for reference images)
            - `img_1.bmp`
            - `img_2.bmp`
            - `img_3.bmp`
            - `img_4.bmp`
            - `img_5.bmp`
    - `lib` (directory for technology libraries)
        - `osu018_stdcells.db`
        - `osu018_stdcells.v`
        - `generic.sdb`
    - `rtl` (directory for RTL verilog code)
        - `canny_edge_detector.v`
    - `sim` (directory where Synopsys VCS will be run)
    - `syn` (directory where Synopsys Design Vision will be run)
        - `netlist` (directory for gate-level verilog code)
        - `sdf` (directory for SDF files)
    - `tb` (directory for testbenches)
        - `canny_edge_detector_tb.v`

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

### 4.1 Simulating Canny Edge Detector RTL Design Using Synopsys VCS
1. Execute the following commands in sequence to generate simulation.
    - `source /opt/coe/synopsys/vcs/W-2024.09-SP2-4/setup.vcs.sh`
    - `cd $HOME/ecen468/lab10/sim`
    - `vcs -full64 ../tb/canny_edge_detector_tb.v -o simv_canny_edge_detector_tb`

2. Execute the following command to run the simulation.
    - `./simv_canny_edge_detector_tb`

4. Five BMP images should be generated in `../images/generated`.

### 4.2 Calculate the Match Ratio of the Generated Images
1. Execute the following commands to create a Python virtual environment and install the necessary packages.
    - `cd $HOME/ecen468/lab10/images`
    - `python3.12 -m venv .venv`
    - `source .venv/bin/activate`
    - `pip install --upgrade pip`
    - `pip install pillow numpy`

2. Execute the following command to run the Python script to calculate the match ratio.
    - `python calculate_match_ratio.py`

3. Take a screenshot of the terminal output of the script. Match ratio no less than 98% is required for a full score.   

---

## 5. Submission
Please submit a single PDF file containing the following:

- Canny edge detector RTL design:
    1. Screenshot of the terminal output after running `python calculate_match_ratio.py`.
    2. Screenshots or copy of the content of the following file:
        - `canny_edge_detector.v`
