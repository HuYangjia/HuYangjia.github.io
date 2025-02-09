---
layout: archive
title: "Experience Page"
permalink: /experience/
author_profile: true
redirect_from:
  - /resume
---

> This page shows my research and project experience.


## Optimization of FPGA-Based Control and Data Transmission for Piezoresistive Array Devices

+ Nov 2023 - Dec 2024
+ Supervisor: Prof. Xiaohui Cai, USTC

This project aims to optimize the control and data transmission of piezoresistive array devices on FPGA. The project includes the following parts:

1. **System Development**: Design a system that can control the piezoresistive array devices and transmit the data to the host computer.
   + Determine the function and build a system prototype.
   + Develop an appropriate communication protocol.
   + FPGA Implement, Leverage Xilinx Virtex-7 FPGAs for system design to ensure hardware compatibility and performance.

2. **System Optimization**: Optimize the system to improve the performance.
    + Optimize the data transmission speed. From the original 40Hz to 80Hz.
    + Expand transmission modes by introducing two new modes: Bluetooth and SD card, in addition to the existing serial port and USB.
    + Enhance real-time signal processing capabilities. Implementing a command frame control system inorder to control in different modes.

In this project, I have learned how to design a system on FPGA, how to optimize the system, and how to improve the performance of the system. I have also learned how to use Xilinx Vivado to implement the system on FPGA.

## 8th National College Students Computer System Ability Competition, CPU Track

+ May 2024 - Aug 2024
+ Advisor: Prof. Junxia Zhang, USTC

In this competition, my team and I designed and implemented a sequential dual-issue 8-stage pipeline CPU based on the Loongarch instruction set. We finally run a simple uboot on our CPU. For me, Responsibilities included the following:

1. **System Design**: Undertake the task of CPU architecture design, including overall architecture planning and key component design.
2. **System Implementation**
   1. Implement the stage prior to Execute stage, including Fetch, Decode, and Issue stages.
   2. Implement the critical components including pre-decoder, branch prediction, and some part of L1 Cache.
3. **System Optimization**: Optimize the system to improve the performance. 
   1. Solve the problem of the critical path and increase the clock frequency from 65MHz to 79MHz.
   2. Verify the correctness of the CPU by building a golden trace in Verilator.

Through this project, I gained hands-on experience in CPU design and implementation, as well as optimization techniques. More importantly, I developed valuable skills in teamwork and system design and implementation on FPGA platforms.

## Rewriting Linux Kernel bpf_trace Module with Rust

+ Apr 2024 - Jul 2024
+ Supervisor: Prof. Kai Xing, USTC

This project aims to rewrite the Linux kernel bpf_trace module with Rust. My team and I translate and debug the bpf_trace module in the Linux kernel from C to Rust, enhancing the module’s safety and performance.

My contribution:
+ Reorganized and refactored C language structures into Rust, ensuring efficient data structure performance and
compatibility.
+ Translated key portions of the module’s C code to Rust, ensuring functional correctness and performance
optimization, and participated in the overall debugging process.


## Implemented a 2-issue out-of-order CPU(ongoing)

+ Dec 2024 - Present
+ Supervisor: Prof. Jian Weng, KAUST

In this project, I utilized a newly developed language(will be open-sourced in the near future) from the lab to design and implement a high-performance CPU, exploring the potential and practical application value of this language in hardware design.

The project includes the following parts:

1. **System Design & Implementation**: Design a 2-issue out-of-order CPU based on the new language.
   1. Design and build critical components including the issue queue, register renaming, and reorder buffer.
   2. Utilize the new hardware description language to enable more flexible and efficient design of the CPU’s components, leveraging its advanced features to streamline development.
2. **Challenges and Solutions**:
   1. Overcoming the limitations of traditional hardware description languages by taking advantage of the new language’s features to simplify and optimize the design process.

In this project, I constructed a high-performance CPU using a new hardware description language, gaining valuable experience in CPU design and implementation. In the following months, I will contribute to automatic pipline splitting feature for this language
