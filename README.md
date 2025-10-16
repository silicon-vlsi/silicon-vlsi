This repo mostly hosts courses and projects from the [Advanced VLSI Lab](https://silicon-vlsi.github.io) at [Silicon University](https://www.silicon.ac.in), Bhubaneswar.

**CURRENT ACTIVITIES**

- [TT SKY25b SRAM](https://github.com/silicon-vlsi/TTSKY25b-SRAM/)
- [Open-Source Standard Cell Development](https://github.com/silicon-vlsi/gf180mcu_as_7t3v3_ocd_io)
- CHIPATHON 2025
  - [Official Chipathon 2025 Github Page](https://github.com/sscs-ose/sscs-chipathon-2025)
    - [Fork of Official page](https://github.com/silicon-vlsi/sscs-chipathon-2025-D14)
  - [Mentoring page for Silicon participants](https://github.com/silicon-vlsi/chipathon25-SILICON_BBI)
  - [Member (srout) Chipathon page from template](https://github.com/silicon-vlsi/chipathon25-SILICON_BBI-sroutk)
- [VLSI 2026 Mentoring Portal](https://github.com/silicon-vlsi/VLSI-2026) : VLSI Mentoring portal for 2026 Silicon batch.


# Silicon Trainings

- [LaunchLab Freshers Training](https://github.com/silicon-vlsi/LaunchLab-Freshers-Training):  Basic MOSFET theory, CMOS technology, circuit and layout design, and basic PDK design. This is the material used for training all New College Graduates (NCG) with no or minimum knowledge in VLSI.

- [Introduction to SKILL Programming](https://github.com/silicon-vlsi/SKILL): Introduces users to basic SKILL programming.

- [Introduction to LVS using Calibre](https://github.com/silicon-vlsi/LVS-RULEDECK-DESIGN/tree/main/LVS)

- [Introduction to Verilog](https://github.com/silicon-vlsi-org/module-ee2.301): Module EE2.301 (WIP)

- [Introduction to Linux Commands](https://github.com/silicon-vlsi-org/module-cs3-301): Module CS 3.301 (WIP)
- [Introduction to Linux Scripting](https://github.com/VLSI-LaunchLab/Intro-to-Linux-Scripting)
- [AI Python for Beginners](https://github.com/VLSI-LaunchLab/AI-Python-for-Beginners)
- [Introduction to CocoTB](https://github.com/silicon-vlsi/Intro2Cocotb)

# External Trainings
- [GIAN Workshop 2024 IIT-BBS by Steve Hoover](https://github.com/silicon-vlsi/gian-course-2024-IITBBS)
- [RISC-V MYTH Workshop - Steve Hoover](https://github.com/silicon-vlsi/RISC-V_MYTH_Workshop)
- [ChipCraft: The Art of Chip Design for Non-Experts](https://github.com/efabless/chipcraft---mest-course) Efbaless's 2-week ASIC Desing Course by Steve Hoover's TL-Verilog to design and implement a RISC-V processor both in FPGA and ASIC (TinyTapeout).
- [Analog Circuit Desing](https://iic-jku.github.io/analog-circuit-design/) at IIC-JKU, Austria, using open source tools *Xschem* and *ngspice* and opensource PDK *SG13G2* from IHP Microelectronics.

# Projects

- [Tiny Tapeout Verilog Project: Digital Temperature Monitor](https://github.com/silicon-efabless/tt06-silicon-tinytapeout-lm07): This project is based on the same [FPGA Project](https://github.com/silicon-vlsi/LM70-Interfacing-FPGA) below that was enhanced and ported to SKY130 using the TinyTapeout shuttle.
- [Temperature Sensor (LM70) design cycle: Verilog-to-FPGA](https://github.com/silicon-vlsi/LM70-Interfacing-FPGA): A system design project using Verilog and Xilinx FPGA development board. RTL to read a temperature sensor (LM70) using the SPI protocol. Synthesize and burn it into a Xilinx FPGA which is interfaced to a LM70 and the temperature is decoded and displayed on a 7-segment display.

- [Analog Front-End (AFE) for a PT100 Temperature Sensor in HIMA temp chamber](https://github.com/silicon-vlsi-org/PT100-AFE-1): The goal of the project is to design and implement a AFE for PT100 temperature sensor that is embeddedd inside the temeperature chamber. The output is converted to the required dynamic range of a USB-based instrument Analog Discovery 2 (AD2). A Pyhton code is developed to read the ADC value from AD2 and onvert it to temperature in degree C. This, then can be used in temperature characterization of electronic systems. 

- [Bandgap Reference in SKY130](https://github.com/silicon-vlsi/BGR_DESIGN_SKY130nm): Complete design, layout and characterization of a Bandgap reference using open-source EDA tools (ngspice/netgen/Magic) and open-source PDK (SKY130)

- [Project Volta : SPI/I2C/SRAM/BGR in 0.6um CMOS](https://github.com/silicon-vlsi-org/project-volta): SPI-RW SRAM and bandgap reference in 0.6um CMOS technology
  - [I2C](https://github.com/silicon-vlsi/VOLTA)

- [Project Tesla](https://github.com/silicon-vlsi/TESLA):(WIP)

- [Project Nuemann (Shelved)](https://github.com/silicon-vlsi/neumann)

# CAD/Technology

- [ngspice](https://github.com/silicon-vlsi-org/eda-ngspice): Compiled `ngspice` source for Ubuntu 22.04/18.04 & CentOS7
- [netgen](https://github.com/silicon-vlsi-org/eda-netgen): Compiled `netgen` source for Ubuntu 22.04/18.04 & CentOS7
- [Magic](https://github.com/silicon-vlsi-org/eda-magic): Compiled `magic` sources for Ubuntu 22.04/18.04 & CentOS7
- [XSchem](https://github.com/silicon-vlsi-org/eda-xschem): Compiled sources for `xschem` schematic editor.
- [Sue2Plus](https://github.com/silicon-vlsi-org/eda-sue2Plus): Sue2 schematic editor + Python/Matlab/Octave tool box
- [Cadence](https://github.com/silicon-vlsi-org/eda-cadence): Information related to Cadence EDA tools eg. Virtuoso, ADE, Spectre, etc.
- [VirtualBox](https://github.com/silicon-vlsi-org/eda-virtualmachine): Setting up Virtual Machines (Virtual Box) and setup open source EDA tools
- [SKY130 for Analog Design in Xschem](https://github.com/silicon-vlsi-org/pdk-sky130-ana): A subset of SKY130 to do analog design using xschem and ngspice
- [Technology](https://github.com/silicon-vlsi-org/eda-technology): Technology files for the open-source EDA tools. **sky130-hd-ttharden24** can used for RTL2GDS flow. It's from the local hardening of TinyTapeout design.
- [RTL2GDS using openROAD](https://github.com/silicon-efabless/kws-genai-hw/tree/main/rtl2gds-tutorial): A quick tutorial for RTL2GDS using OpenROAD flow. 
- [Technology](https://github.com/silicon-vlsi-org/eda-technology): Technology files for the open-source EDA tools.
- [Open Source EDA on WSL](https://github.com/silicon-vlsi-org/eda-wsl2): Installing and setting up WSL2 on Windows 10/11 and installing open-source EDA tools on it.
- [Open Source EDA on VirtualBox](https://github.com/silicon-vlsi-org/eda-virtualmachine): Setting up Virtual Machines (Virtual Box) and setup open source EDA tools.

# Documentation

- [Website: Advanced VLSI Lab](https://github.com/silicon-vlsi/silicon-vlsi.github.io): Jekyll repo for GitHub page for Advanced VLSI Lab website.
- [Eagle PCB: A Quick Guide](https://github.com/silicon-vlsi/Learn-Eagle)
- [Calibre Setup](https://github.com/silicon-vlsi/LVS-RULEDECK-DESIGN)


# Course Portals

- [SI 2025 Analog IC Design](https://github.com/silicon-vlsi/SI-2025-AnalogIC): Summer Internship course "Analog IC Design"
- [SI 2025 Digital VLSI](https://github.com/silicon-vlsi/SI-2025-DigitalVLSI): Summer Internship course "Digital VLSI Design from Verilog RTL and Verification to Synthesis"
- [SI-2024: Introduction to CubeSat and Satellite Communication](https://github.com/silicon-sat/SI-2024-CubeSat) : 2024 Summer internship course.
- [SI-2021-03: Digital CMOS VLSI Design](https://github.com/silicon-vlsi/SI2021-03-CMOS-VLSI) : 2021 Summer Internship course.
- [15VLSI7T: VLSI Design Course, 7th Sem. AEI, 2020](https://github.com/silicon-vlsi/15VLSI7T) : 2020 7th Sem VLSI Design course.

# Project/Mentoring Portals

- [PS 2025 Sem 8 CMOS](https://github.com/silicon-vlsi/PS-2025-S8) : CMOS VLSI training.
- [PS 2025 Sem 8 AIedu](https://github.com/silicon-vlsi/AIedu) : AI-based assesment platform.
- [PS 2025 Sem 8 TT06](https://github.com/silicon-efabless/tt06-silicon-tinytapeout-lm07) : Test development for TinyTapeout project Digital Temperature Monitor.
  - [Introduction to RP2040 and using it as a SPI slave](https://github.com/03jayashree/rp2040-spi-temperature) : A valuable project showing the use of PIO in RP2040 to use it as a SPI slave.
- [PS 2025 Sem 8 HeartSound](https://github.com/silicon-vlsi/heart-sound) : AI-based heart sound monitoring.
- [Introduction to CocoTB](https://github.com/silicon-vlsi/Intro2Cocotb)
- [Potentiostat](https://github.com/silicon-vlsi/potentiostat)
- [KWS/HeartSound Monitor](https://github.com/silicon-vlsi/sound-analysis)
- [PS 2025 Project: 8T SRAM InCompute Memory](https://github.com/silicon-vlsi/PS-2025-RAM)
- [Potentiostat](https://github.com/silicon-vlsi/potentiostat): Building a simplified potentiostat or a three-electrode electrochemical measurement system.
- [Analog Char](https://github.com/silicon-vlsi/LaunchLab-AnalogChar): LaunchLab Analog Characterization portal.
- [Priyansu's Keyword Spotter](https://github.com/Priyansu122/Project_keywordSpotter) : Priyansu Sahoo's keyword spotter portal.
- [Priyansu's RTL2GDS Flow Page](https://github.com/Priyansu122/SI2024_RTL_TO_GDS)
- [Priyansu's FPGA Flow Page](https://github.com/Priyansu122/ASIC_FPGA_Design_Flow)
- [GPIO](https://github.com/silicon-vlsi/gpio) : GPIO/I3C Project portal
- [SIT 2024: Verilog SPI Temperature Project](https://github.com/silicon-vlsi/VLSI-2024) : 2024 Mentoring portal
- [SIT 2021: Project Page](https://github.com/silicon-vlsi/Project2021) : 2021 Mentoring portal
- [SIT 2020: Project Page](https://github.com/silicon-vlsi/project2020) : 2020 Mentoring portal

<!---
silicon-vlsi/silicon-vlsi is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
