---
layout: manual
title: 'Lab 6: SystemC Verification Library (SCV)'
session: 'Week 11 (Nov 2 – Nov 6)'
report_due: 'Week 12 (Nov 9 – Nov 13)'
downloads:
  - label: code (tar.gz)
    file: /assets/files/lab06/lab06_code.tar.gz
---

## 1. Objectives
- Verify the RAM module using the SystemC Verification Library (SCV).

---

## 2. Introduction
SCV is a C++ library built on SystemC that adds verification features. Instead of hand-picking test values, the testbench describes which values are legal, and SCV generates random values within those limits.

Data introspection lets SCV see inside user-defined data types. C++ has traditionally given a library no way to list a struct's fields, so the user registers them with SCV. SCV can then randomize, constrain and print a composite type, such as a bus packet, as easily as a built-in type.

Constrained randomization decides which values are legal. Constraints are rules over the fields, such as value ranges or relations between fields, and every generated value satisfies all of them at once. Among the legal combinations, SCV picks evenly, so the stimulus stays valid while still covering the design broadly.

Weighted randomization decides how often each legal value appears. Users give individual values or ranges relative weights, and SCV draws them in proportion, for example favoring boundary values or typical traffic. Testing then concentrates where bugs are most likely, while rarer cases still appear occasionally.

---

## 3. Lab Procedures

### 3.0 Setup
1. Execute the following commands to create and enter the working directory.
    - `cd $HOME/ecen468/`

2. Download `lab06_code.tar.gz` from the lab website and put it in the working directory.

3. Execute the following commands to extract the files.
    - `tar -xvf lab06_code.tar.gz`
    - `rm lab06_code.tar.gz`

4. Confirm the following directories and files exist in the working directory.
    - `ram.cpp`
    - `test_ram.cpp`
    - `workspace` (directory where Siemens Vista will be run)

5. Execute the following command if you are not using a computer in ZACH 127.
    - `load-ecen-468`

### 3.1 Building SystemC Design Using Siemens Vista
1. Execute the following commands in sequence to open Siemens Vista.
    - `source /opt/coe/mentorgraphics/vista/2024_2/setup.vista.linux.bash`
    - `source /opt/rh/gcc-toolset-13/enable`
    - `cd $HOME/ecen468/lab03/workspace`
    - `vista &`

2. In Vista graphical interface, click **Project** on the menu bar, then click **New Project...**.

3. In the **New Project** dialog, click **Save** (You don't need to change the default file name).

4. In the **Create New Project** dialog, click **Files** on the tab bar, then click **Add Files** on the bottom right.

5. In the **Select Files** dialog, click the folder icon and select the C++ source code and header files, click **Open**, then click **OK**.

6. In the **Create New Project** dialog, click **Link** on the tab bar, check **SCV**, then click **OK**.

7. On the side bar, right click on **Project** and click **Build**. Every time you modify the source code or header files, you need to rebuild the project.

8. If no errors occur, proceed to the next section.

### 3.2 Simulating SystemC Design Using Siemens Vista
1. On the side bar, click the **+** next to **Project** expand its content.

2. Click the **+** next to **Simulation**, then right click on **main.cpp** and click **Simulate**.

3. In the **simulation** dialog, deselect **Stop after elaboration** under **Debugging**, then click **OK**.

4. Take a screenshot of the simulation output in Vista for the lab report.

5. After the simulation exits, a waveform file named `wave.vcd` will be generated in `workspace`.

6. In the terminal, execute the following commands to Synopsys WaveView.
    - `source /opt/coe/synopsys/wv/V-2023.12-4/setup.wv.sh`
    - `wv &`

7. In WaveView, open `wave.vcd` to view the simulation waveform.

8. Take a screenshot of the waveform for the lab report.

---

## 4. Submission
Please submit a single PDF file containing the following:

1. Screenshot of the waveform with analysis.
3. Screenshots or copy of the content of the following files:
    - `test_ram.cpp`
