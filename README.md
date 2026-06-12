# Intro to FPGA Design

![Verilog](https://img.shields.io/badge/HDL-Verilog-blue)
![Quartus Prime](https://img.shields.io/badge/Tool-Quartus%20Prime-darkblue)
![ModelSim](https://img.shields.io/badge/Simulation-ModelSim-orange)
![FPGA](https://img.shields.io/badge/Hardware-Cyclone%20IV-success)
![License](https://img.shields.io/badge/License-MIT-green)
![Last Commit](https://img.shields.io/github/last-commit/SardarZiyatkhanov/Intro-to-FPGA-Design)
![Repo Size](https://img.shields.io/github/repo-size/SardarZiyatkhanov/Intro-to-FPGA-Design)

This repository contains my lab work for **Introduction to FPGA Design**, covering digital logic design and implementation using Verilog HDL in Quartus Prime, simulation in ModelSim, and deployment on Altera/Alinx Cyclone IV FPGA boards.

**Student:** Sardar Ziyatkhanov

<p align="center">
  <img src="Lab-1/Screenshot 2025-09-30 195802_updated.png" alt="Full Adder circuit schematic" width="600">
</p>

---

## Table of Contents

- [Labs Included](#labs-included)
- [Repository Structure](#repository-structure)
- [Tools Used](#tools-used)
- [Contributing](#contributing)
- [License](#license)

---

## Labs Included

| Lab | Title | Summary |
|-----|-------|---------|
| [Lab 1](Lab-1/LAB_1_REPORT.MD) | Half-Adder and Full-Adder | Designing and representing Half-Adder and Full-Adder circuits in Verilog using Quartus Prime, based on Boolean algebra, truth tables, and minterm/maxterm representations. |
| [Lab 2](Lab-2/LAB_2_REPORT.MD) | Multiplexer and Decoder Simulation | Simulating a 4x1 multiplexer and a decoder in Quartus Prime with testbench code and waveform verification. |
| [Lab 3](Lab-3/LAB_3_README.MD) | 7-Segment Display | Implementing pin allocation in Pin Planner and programming an Altera Cyclone IV FPGA board to display 4-bit binary numbers as decimal digits on a 7-segment display. |
| [Lab 4](Lab-4/LAB_4_README.MD) | 3-Bit Counter with Enable and Reset | Designing and simulating a 3-bit counter with Enable and Reset inputs in Verilog, verified in ModelSim and deployed on an Alinx AX4010 Cyclone IV FPGA board with output shown on a 7-segment display. |
| [Lab 5](Lab-5/LAB_5_REPORT.MD) | Sequence Detector State Machine | Implementing a Moore state machine on the FPGA board that detects the bit sequence 1011 from an input key, using a clock-driven state machine with LED and 7-segment display output. |
| [Final Project](Final-Project/FINAL_PROJECT_REPORT.MD) | Vending Machine | Implementing a vending machine on the FPGA board using buttons as coin inputs (nickel, dime, quarter), with 7-segment displays showing the inserted amount and LEDs indicating dispense and change, using Quartus Prime and ModelSim. |

---

## Repository Structure

```text
Intro-to-FPGA-Design/
├── Lab-1/
├── Lab-2/
├── Lab-3/
├── Lab-4/
├── Lab-5/
└── Final-Project/
```

---

## Tools Used

- **Quartus Prime**
- **ModelSim**
- **MultiSim**
- **Altera Cyclone IV FPGA board**
- **Alinx AX4010 Cyclone IV FPGA board**

---

## Contributing

This is a personal academic repository, so it isn't actively seeking code contributions. That said, if you spot an error or have a suggestion, feel free to open an issue. See [CONTRIBUTING.md](CONTRIBUTING.md) for details, including a note on academic integrity.

---

## License

This project is licensed under the [MIT License](LICENSE).
