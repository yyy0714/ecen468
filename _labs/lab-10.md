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

The module is a **register-mapped datapath**: the testbench streams the pixels of one neighborhood into the module, runs a processing mode, and reads the resulting pixel back. Every stage in Sections 2.1–2.5 is coded against the same interface and windowing conventions described here.

**Storage.** Three 25-entry input buffers hold the current pixel window — `buf_x`, `buf_y`, `buf_z` — plus the coefficient buffers `gf` (Gaussian), `sobel_x`, and `sobel_y`. Five result registers `tmp_1`…`tmp_5` hold the outputs of the five stages. The coefficient buffers are constants and are loaded once, while `rst_ni` is asserted.

**Transactions.** For each output pixel the testbench issues three kinds of bus cycles:

| Cycle | Control | Action |
|---|---|---|
| Write | `ce_ni=0, we_ni=0` | store `data_i` into the buffer chosen by `write_reg_select_i` (`WRITE_BUF_X/Y/Z`) at index `row_i*5 + col_i` |
| Operate | `ce_ni=1, op_enable_ni=0` | run the datapath selected by `op_mode_i` (`MODE_GAUSSIAN/SOBEL/NMS/HYSTERESIS`) |
| Read | `ce_ni=0, we_ni=1` | drive the register chosen by `read_reg_select_i` (`REG_GAUSSIAN/GRADIENT/DIRECTION/NMS/HYSTERESIS`) onto `data_o` |

**Multi-cycle operations.** Each operation is a short sequence of clock cycles counted by the `state` register. `state` is cleared to `0` whenever `op_enable_ni` is deasserted, so every operation begins at `state = 0` and advances one step per clock while it is enabled. Split each stage into the `state` steps listed in its section below.

**Window indexing.** A window is stored row-major, so the pixel at (row, col) is at index `row*5 + col`: moving one column is `±1`, moving one row is `±5`.

```
5×5 window (Gaussian)          3×3 window (Sobel / NMS / Hysteresis)
  0  1  2  3  4                   0  1  2
  5  6  7  8  9                   5  6  7
 10 11 12 13 14                  10 11 12
 15 16 17 18 19
 20 21 22 23 24                 center pixel = buf_x[6]
center pixel = buf_x[12]
```

The 3×3 stages use only indices `{0,1,2,5,6,7,10,11,12}` with the center at `buf_x[6]`. Given a direction step `(dx, dy)`, the two opposite neighbors of the center are

$$
\text{buf\_x}\big[\,6 - (5\cdot dy + dx)\,\big] \qquad\text{and}\qquad
\text{buf\_x}\big[\,6 + (5\cdot dy + dx)\,\big]
$$

## 2.1 Blurred Image
Image 1 removes high-frequency noise and small intensity variations that would otherwise create false edges. It is the \\(5 \times 5\\) Gaussian convolution of Image 0, \\(I_1 = G * I_0\\).

**Kernel.** The Gaussian kernel is stored as integer weights in `gf[0..24]` (loaded while `rst_ni` is low):

$$
\text{gf} =
\begin{bmatrix}
1 & 3 & 4 & 3 & 1 \\
3 & 7 & 10 & 7 & 3 \\
4 & 10 & 16 & 10 & 4 \\
3 & 7 & 10 & 7 & 3 \\
1 & 3 & 4 & 3 & 1
\end{bmatrix}
$$

**Normalization.** The 25 weights sum to 128, so the blurred pixel is the weighted sum divided by 128. Because \\(128 = 2^{7}\\), the division is simply a right shift by 7 (no divider required):

$$
I_1 = \left( \sum_{i=0}^{24} b_i \, g_i \right) \gg 7
$$

where \\(b_i =\\) `buf_x[i]` and \\(g_i =\\) `gf[i]`.

**Implementation (`MODE_GAUSSIAN`).**
- `state 0`: accumulate the 25-term weighted sum into the 32-bit `gaussian_sum`.
- `state 1`: `tmp_1 <= gaussian_sum >> 7`.

`tmp_1` is read back through `REG_GAUSSIAN`.

## 2.2 Gradient Image
Image 2 measures how strongly the intensity changes at each pixel of the blurred image, using two \\(3 \times 3\\) Sobel kernels (stored in `sobel_x`, `sobel_y`):

$$
S_x =
\begin{bmatrix} -1 & 0 & 1 \\ -2 & 0 & 2 \\ -1 & 0 & 1 \end{bmatrix}
\qquad
S_y =
\begin{bmatrix} 1 & 2 & 1 \\ 0 & 0 & 0 \\ -1 & -2 & -1 \end{bmatrix}
$$

**Gradients.** Applying the kernels to the 3×3 window in `buf_x` (with \\(b_k =\\) `buf_x[k]`) gives the horizontal and vertical gradients. Store them in the signed 32-bit `gradient_x` and `gradient_y`:

$$
G_x = (b_2 + 2b_7 + b_{12}) - (b_0 + 2b_5 + b_{10})
$$

$$
G_y = (b_0 + 2b_1 + b_2) - (b_{10} + 2b_{11} + b_{12})
$$

**Magnitude.** The ideal magnitude is \\(\sqrt{G_x^2 + G_y^2}\\), but a square root is expensive in hardware. This design uses the \\(L_1\\) (Manhattan) approximation and scales it down by 8 (\\(\gg 3\\)):

$$
I_2 = \frac{|G_x| + |G_y|}{8} = \big(|G_x| + |G_y|\big) \gg 3
$$

Compute \\(|G_x| + |G_y|\\) by branching on the signs of `gradient_x` and `gradient_y` (four cases: `++`, `+-`, `-+`, `--`), so no separate absolute-value step is needed.

**Implementation (`MODE_SOBEL`, part 1).**
- `state 0`: compute `gradient_x` and `gradient_y`.
- `state 1`: store the scaled magnitude in `tmp_2` (read back through `REG_GRADIENT`).

The same `MODE_SOBEL` operation also produces the direction image in `state 2`–`state 3` (Section 2.3), reusing `gradient_x`/`gradient_y`.

## 2.3 Direction Image
Image 3 records the gradient direction, which non-maximum suppression needs. Computing \\(\theta = \operatorname{atan2}(G_y, G_x)\\) exactly requires an arctangent, so instead the design **quantizes** the direction into four codes — `0`, `45`, `90`, `135` (degrees) — using only comparisons and shifts.

**Fold to the upper half-plane (`state 2`).** Edge orientation is periodic modulo 180°, so if `gradient_y < 0`, negate both components:
\\((G_x, G_y) \rightarrow (-G_x, -G_y)\\). Store the folded pair in `abs_gradient_x`, `abs_gradient_y`. After folding, `abs_gradient_y` \\(\ge 0\\); `abs_gradient_x` keeps its sign (it is negative when the vector points into the second quadrant).

**Classify by slope (`state 3`).** Compare `abs_gradient_y` against fractions of `abs_gradient_x` that approximate the 22.5° and 67.5° sector boundaries — \\(\tan 22.5^\circ \approx \tfrac{1}{2}\\) and \\(\tan 67.5^\circ \approx \tfrac{5}{2}\\) (the \\(\div 2\\) is `>>1`). Writing \\(a_x =\\) `abs_gradient_x` and \\(a_y =\\) `abs_gradient_y`:

| Condition | `tmp_3` |
|---|---|
| \\(a_x \ge 0\\) and \\(a_y \le a_x/2\\) | `0` |
| \\(a_x \ge 0\\) and \\(a_x/2 < a_y \le 5a_x/2\\) | `45` |
| \\(a_x \ge 0\\) and \\(a_y > 5a_x/2\\) | `90` |
| \\(a_x < 0\\) and \\(a_y \le -a_x/2\\) | `0` |
| \\(a_x < 0\\) and \\(-a_x/2 < a_y \le -5a_x/2\\) | `135` |
| \\(a_x < 0\\) and \\(a_y > -5a_x/2\\) | `90` |

`tmp_3` is read back through `REG_DIRECTION`. The colored direction image is produced entirely in the testbench and is only a visualization of these four codes.

## 2.4 Non-Maximum Suppression (NMS) Image
Image 4 thins edges by keeping a gradient-magnitude pixel only if it is a local maximum **along the gradient direction**; otherwise the pixel is set to 0. The stage reads the 3×3 magnitude window from `buf_x` (Image 2, center `buf_x[6]`) and the direction code from `buf_y[6]` (Image 3).

**Step 1 — direction to offset (`state 0`).** Convert the direction code into a step `(dx, dy)` that points along the gradient, and record it in `dx`, `dy`:

| `buf_y[6]` | `(dx, dy)` | Neighbors compared |
|---|---|---|
| `0`   | `(1, 0)`  | `buf_x[5]`, `buf_x[7]`  (left / right) |
| `45`  | `(1, -1)` | `buf_x[2]`, `buf_x[10]` (anti-diagonal) |
| `90`  | `(0, 1)`  | `buf_x[1]`, `buf_x[11]` (top / bottom) |
| `135` | `(1, 1)`  | `buf_x[0]`, `buf_x[12]` (main diagonal) |

The two neighbors are `buf_x[6 - (5*dy+dx)]` and `buf_x[6 + (5*dy+dx)]`.

**Step 2 — compare and suppress (`state 1`).** First copy the 3×3 window into `tmp_4` (so the non-suppressed pixels keep their magnitude). Then:
- if `buf_x[6]` is \\(\ge\\) **both** neighbors, it is the local maximum — set the two neighbor positions in `tmp_4` to 0;
- else if `buf_x[6]` is `<` **either** neighbor, it is not the maximum — set `tmp_4[6]` to 0.

`tmp_4` is read back position-by-position through `REG_NMS` (using `row_i`/`col_i`). The testbench writes the surviving window back into Image 4 so that suppression accumulates across overlapping windows.

## 2.5 Hysteresis Image
Image 5 is the final binary edge map. It applies two thresholds to the NMS magnitude in `buf_x[6]` (Image 4), using the parameters `THRESHOLD_UPPER` (= 10) and `THRESHOLD_LOWER` (= 3):

- `buf_x[6] >= THRESHOLD_UPPER` → **strong** edge → `tmp_5 = 1`.
- `buf_x[6] <= THRESHOLD_LOWER` → non-edge → `tmp_5 = 0`.
- otherwise → **candidate**: keep it (`tmp_5 = 1`) only if it connects to an edge, else `tmp_5 = 0`.

**Connectivity is checked along the edge** (perpendicular to the gradient), so the `(dx, dy)` mapping is rotated 90° relative to NMS:

| `buf_y[6]` | `(dx, dy)` | Neighbors checked (along the edge) |
|---|---|---|
| `0`   | `(0, 1)`  | `buf_x[1]`, `buf_x[11]` |
| `45`  | `(1, 1)`  | `buf_x[0]`, `buf_x[12]` |
| `90`  | `(1, 0)`  | `buf_x[5]`, `buf_x[7]` |
| `135` | `(1, -1)` | `buf_x[2]`, `buf_x[10]` |

A candidate becomes an edge (`tmp_5 = 1`) if either along-edge neighbor is itself strong (`buf_x[neighbor] >= THRESHOLD_UPPER`) **or** is already marked as an edge in the running edge map `buf_z` (`buf_z[neighbor] == 1`). `buf_z` holds Image 5 as computed so far and is supplied by the testbench for each pixel, which lets confirmed edges propagate to connected weak pixels.

**Implementation (`MODE_HYSTERESIS`).**
- `state 0`: map `buf_y[6]` to `(dx, dy)`.
- `state 1`: apply the threshold-and-connectivity rule above and write `tmp_5` (read back through `REG_HYSTERESIS`).

This stage removes isolated weak responses while preserving continuous edges. The resulting Image 5 is the final Canny edge map.

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
