# RISC-V-SOC-Tapeout-Program_VSD

<details>
  <summary>Week 0 </summary>

# Week 0 - Tools Installation

### Yosys

$ sudo apt-get update
$ git clone https://github.com/YosysHQ/yosys.git
$ cd yosys
$ sudo apt install make (If make is not installed please install it)
$ sudo apt-get install build-essential clang bison flex \
libreadline-dev gawk tcl-dev libffi-dev git \
graphviz xdot pkg-config python3 libboost-system-dev \
libboost-python-dev libboost-filesystem-dev zlib1g-dev
$ make config-gcc
$ make
$ sudo make install

![Week 1 Screenshot](week1.png)  
*(Upload your image named `week1.png` to the repo root)*

### Code
```python
print("Hello from Week 1!")

