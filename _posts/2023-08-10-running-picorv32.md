---
layout: post
title: Running your own program on the PicoRV32
---

[PicoRV32](https://github.com/YosysHQ/picorv32) is an open source core that implements the RISC-V RV32IMC Instruction Set. Here's how to use the core to run your own programs.

This is the structure of the repository:
```
x@y:~/Desktop/picorv32-1.0$ tree -L 1
.
├── dhrystone
├── dotfiles
├── firmware
├── iverilog
├── Makefile
├── obj_dir
├── picoramsoc-master
├── picorv32.core
├── picorv32.v
├── picosoc
├── README.md
├── scripts
├── self
├── shell.nix
├── showtrace.py
├── testbench.cc
├── testbench_ez.v
├── testbench.v
├── testbench.vvp
├── testbench_wb.v
├── tests
└── yosys
```

Run `make test ` just to make sure you have all the prerequisites installed.

<p class="message"> The version of `iverilog` on the debian apt repository is **old**(10.3). Build it from source by following the instructions on the iverilog repository. </p>
