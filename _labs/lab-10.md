---
layout: manual
title: 'Lab 10: Design of Canny Edge Detector (Verilog)'
session: 'Week 11 (Nov 2 – Nov 6)'
report_due: 'Week 12 (Nov 9 – Nov 13)'
manual_pdf: /assets/files/lab10/lab10_manual.pdf
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab10/lab10_code.tar.gz
---

## Objectives

In this lab, we will implement a Canny edge detector and use `xrun` for simulation and verification.

## Introduction

**Edge detection** refers to identifying and locating sharp discontinuities in an image. These discontinuities are abrupt changes in pixel intensity that characterize the boundaries of objects in a scene. Classical edge detection methods convolve the image with an operator (a 2-D filter) designed to be sensitive to large gradients while returning zero in uniform regions.

Many edge detectors are available, each designed to be sensitive to certain types of edges. Factors in choosing an edge detection operator include edge orientation and the noise environment. The geometry of an operator determines the direction in which it is most sensitive to edges; operators can be optimized for horizontal, vertical, or diagonal edges. Edge detection is also difficult in noisy images, because high-frequency components appear in both the noise and the edges, and attempts to reduce the noise tend to blur and distort the edges.

There are many ways to perform edge detection, but most methods fall into two categories: the gradient method and the Laplacian method. The Laplacian method generally yields higher quality at the cost of greater complexity. We will use the gradient method, which detects edges by looking for the maximum and minimum of the first derivative of the image.

![Figure 1. An example of Canny Edge Detection]({{ "/assets/files/lab10/img/1.png" | relative_url }})

*Figure 1. An example of Canny Edge Detection*

### Edge Detection Flow

![Figure 2. Data flow of Edge Detection]({{ "/assets/files/lab10/img/2.png" | relative_url }})

*Figure 2. Data flow of Edge Detection*

Figure 2 shows the data flow of a Canny Edge Detector. The input image goes through four operations: **Noise Reduction**, **Gradient Calculation**, **Non-maximum Suppression**, and **Thresholding**. The following sections describe each in detail.

### Noise Reduction

The first step is to filter out noise in the original image with a Gaussian filter. The filter must be chosen carefully, because filtering removes noise but can also degrade image quality.

Once a suitable mask has been chosen, Gaussian smoothing is performed using standard convolution. A convolution mask is usually much smaller than the image, so it slides over the image, covering a square of pixels at a time. The larger the Gaussian mask, the less sensitive the detector is to noise, although the localization error in the detected edges also increases slightly as the mask grows. The Gaussian mask shown in Figure 3 is used in this lab.

![Figure 3. Gaussian Mask G]({{ "/assets/files/lab10/img/3.png" | relative_url }})

*Figure 3. Gaussian Mask G*

![Figure 4. Convolution using the Gaussian mask]({{ "/assets/files/lab10/img/4.png" | relative_url }})

*Figure 4. Convolution using the Gaussian mask*

Figure 4 shows the convolution process. For the grey pixel p[i,j] in the left image, the window M is selected from the original image and used in the convolution. The value of the corresponding pixel in the smoothed image is computed using the equation shown in the figure.

### Gradient Calculation

After smoothing the image and removing the noise, the next step is to find the edge strength by taking the gradient of the image. We use the Sobel operators in Figure 5 to perform a 2-D gradient measurement. The Sobel operator uses a pair of 3x3 convolution masks: one estimates the gradient in the x-direction (vertical edges) and the other in the y-direction (horizontal edges). The equations in Figure 6 give the gradient values Gx and Gy in the x and y directions.

![Figure 5. Sobel operators (a) Sobel X, (b) Sobel Y]({{ "/assets/files/lab10/img/5.png" | relative_url }})

*Figure 5. Sobel operators (a) Sobel X, (b) Sobel Y*

![Figure 6. Equations of gradient calculation]({{ "/assets/files/lab10/img/6.png" | relative_url }})

*Figure 6. Equations of gradient calculation*

![Figure 7. An example of gradient calculation]({{ "/assets/files/lab10/img/7.png" | relative_url }})

*Figure 7. An example of gradient calculation*

Figure 7 shows an example of gradient calculation. The 5x5 image in Figure 7(a) has the pixel values shown in Figure 7(b) and contains vertical edges (i.e., a large gradient in the x direction). Figures 7(c) and (d) show the gradients after applying the Sobel X and Sobel Y masks. For pixel [1,2], Gx = 400 and Gy = 0; the large x-direction gradient corresponds to the vertical edge at that pixel. For pixel [2,3], Gx = 280 and Gy = 0, again indicating a vertical edge.

Based on Gx and Gy, we can calculate the **magnitude** and the **direction** of the gradient.

The **magnitude**, or edge strength, is approximated using the equation in Figure 8, which performs well at low computational cost.

![Figure 8. The magnitude of the gradient]({{ "/assets/files/lab10/img/8.png" | relative_url }})

*Figure 8. The magnitude of the gradient*

In this equation, the constant α is used for normalization. In our lab, each pixel is represented by eight bits, giving a maximum value of 255, so the maximum value of Gx and Gy is `255*4` and the maximum of |Gx| + |Gy| is `255*8`. Setting α = 8 therefore keeps |G| between 0 and 255.

However, for real images, normalizing the gradient with a constant of 8 makes most gradient magnitudes very small. We can therefore adjust the constant for the images we use, as long as the magnitude does not exceed 255 in our system. In this lab, use α = 2.

The **direction** of the gradient can be computed using the equation in Figure 9. However, implementing the arctan function in hardware is expensive, so we use an approximation instead.

![Figure 9. The direction of the gradient]({{ "/assets/files/lab10/img/9.png" | relative_url }})

*Figure 9. The direction of the gradient*

We compute the gradient direction in order to determine the edge direction. Edges are categorized as horizontal (0 degrees), vertical (90 degrees), or diagonal (45 and 135 degrees), as shown in Figure 10. Using the approximation in Figure 11, we can quickly determine the edge direction by comparing Gx and Gy.

![Figure 10. Directions of the edges]({{ "/assets/files/lab10/img/10.png" | relative_url }})

*Figure 10. Directions of the edges*

![Figure 11. An approximation method for gradient directions]({{ "/assets/files/lab10/img/11.png" | relative_url }})

*Figure 11. An approximation method for gradient directions*

### Non-maximum Suppression

Figure 12 shows an example of non-maximum suppression. Suppose we want to perform non-maximum suppression for pixel C. Following the two directions perpendicular to the edge, we find its two neighboring pixels, A and B.

1. If M(C) ≥ M(A) and M(C) ≥ M(B): discard pixels A and B by setting M(A) = M(B) = 0;
2. Otherwise (M(C) < M(A) or M(C) < M(B)): discard pixel C by setting M(C) = 0.

![Figure 12. Non-maximum suppression]({{ "/assets/files/lab10/img/12.png" | relative_url }})

*Figure 12. Non-maximum suppression*

### Hysteresis Thresholding

The output of non-maximum suppression still contains noisy local maxima. In this lab, we use hysteresis thresholding to further remove noise and improve image quality.

The thresholds need to be set carefully to remove the weak edges while preserving the connectivity of the contours. This algorithm uses two thresholds, `T_high` and `T_low`.

1. A pixel (x,y) is called **strong** if M(x,y) ≥ `T_high`;
2. A pixel (x,y) is called **weak** if M(x,y) ≤ `T_low`;
3. A pixel (x,y) is a candidate pixel otherwise (i.e., `T_low` < M(x,y) < `T_high`).

In each position of (x,y), we:

1. discard the pixel (x,y) if it is weak;
2. keep the pixel if it is strong;
3. If the pixel is a candidate, we should check its two neighbor pixels on its edge directions.
   - If the candidate pixel (x,y) is connected to a strong neighbor, keep the pixel;
   - If the candidate pixel (x,y) is connected to another candidate pixel that has already been regarded as a strong pixel, keep this candidate pixel;
   - Otherwise, discard the candidate pixel.

In our lab, please use `T_high` = 15 and `T_low` = 10.

![Figure 13. Hysteresis thresholding]({{ "/assets/files/lab10/img/13.png" | relative_url }})

*Figure 13. Hysteresis thresholding*

## Implementation & Simulation

![Figure 14. Data flow of the top module and test bench.]({{ "/assets/files/lab10/img/14.png" | relative_url }})

*Figure 14. Data flow of the top module and test bench.*

![Figure 15. File in/out process]({{ "/assets/files/lab10/img/15.png" | relative_url }})

*Figure 15. File in/out process*

Figures 14 and 15 show the structure of the design.

![Figure 16. Data flow of noise reduction]({{ "/assets/files/lab10/img/16.png" | relative_url }})

*Figure 16. Data flow of noise reduction*

Figure 16 shows the data flow of the noise reduction process. First, the top module receives a 5x5 block of the original image (`**X`) from the test bench. This block is convolved with a Gaussian mask to reduce noise. Finally, the result is loaded into the register `Out_gf` and sent to another memory area (`**XG`) in the test bench.

### Useful Verilog Skills

#### One-Dimensional Arrays

In SystemC, we implemented the two-dimensional array `sc_uint<8> regX[5][5]`. Most Verilog simulators do not support two-dimensional expressions, so you can implement it as a one-dimensional array and compute the index yourself. For example, to implement the 5x5 array, declare `reg [7:0] regX[0:24]`, with indices from 0 to 24. For portability across simulators and for synthesis, one-dimensional arrays are recommended; however, you may use two-dimensional arrays for functional simulation.

`Index = Row number * 5 + Column number`

#### Signed Registers

```verilog
reg [7:0] RegX[0:24];        // 25 unsigned registers, 8 bits wide
reg signed [7:0] RegX[0:24]; // 25 signed registers, 8 bits wide
```

#### Shift Instead of Division

When you synthesize modules, the cell library may not provide a divider. To work with any library, use the right-shift operator `>>`:

- Divided by 2 → `(variable) >> 1`
- Divided by 4 → `(variable) >> 2`
- e.g., `var <= var / 128;` → `var <= (var >> 7);`

### Simulation and Verification

Please login to the Olympus server and create a working directory for this lab using the following commands.

```bash
## Create and navigate to the working directory.
mkdir -p $HOME/ECEN468/Lab10/src
cd $HOME/ECEN468/Lab10/src
```

Download the tar.gz file from the lab website and extract it. In the extracted folders, you will find the following files:

- `CannyEdge.v`
- `tb.v`
- `tb_comp.v`
- `generic.sdb`
- `osu018_stdcells.v`
- `osu018_stdcells.db`

Copy them to the working directory.

Implement your design in `CannyEdge.v`.

The simulation reports the matching ratio between the images your implementation generates and the reference images.

Commands for reference:

```bash
load-ecen-468   # skip this line on machines in ZACH 127
source /opt/coe/cadence/XCELIUM240/setup.XCELIUM240.linux.bash
xrun -c   # (complete the rest of the cmd)
xrun -R   # (complete the rest of the cmd)
```

Take screenshots of the terminal output and include them in your report.

Then use `check.py` to verify your result:

`python3 check.py`

## Synthesis

Gate-level design generation

Write your own TCL file that uses `dc_shell` to generate the netlist of `CannyEdge.v`.

Name the gate-level design `CannyEdge_gate.v`.

Attach both your TCL file and your `CannyEdge_gate.v` file to your lab report.

Commands for reference:

```bash
source /opt/coe/synopsys/syn/V-2023.12-SP1/setup.syn.sh
dc_shell -f <your_own_tcl> > output.txt
```

Attach your `output.txt` to the lab report.

The gate-level simulation is not required.

## Submission

Please submit a single PDF file containing the following:

1. Screenshots of the images generated by the simulation.
2. Screenshots of the simulation outputs.
3. Screenshots of running `check.py`.
4. Source code in this design with reasonable comments.
5. Gate level design. (`CannyEdge_gate.v`)
6. `dc_shell` output. (`output.txt`)

A matching ratio of at least 98% is required to get full marks.
