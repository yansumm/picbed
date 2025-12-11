# Lab-4 Design of a RISC-V Processor

# Introduction

The lab aims to design a RISC-V processor supporting MAC (Multiply Accumulate) operation. The lab introduces a two-issue processor with a scalar datapath and a vector datapath. By completing the Verilog codes and designing the corresponding assembly codes, you will have a deep understanding of the computer architecture.

![](images/5b511d9e78496804ed1d49bc526840e2c886017c6c33c4e2e53913fad0e9e8db.jpg)

# Background

# Vector Processor

We can use SIMD to achieve data parallelism. With the rise of applications such as multimedia, big data, and artificial intelligence, it has become increasingly important to endow processors with SIMD processing power. As these applications have a large number of fine-grained, homogeneous, and independent data operations that SIMD is inherently suited to handle.

The structure of a vector processor fits well with the problem of parallel computation of large amounts of data. A vector processor has multiple ALUs that are capable of performing the same operation many times at the same time. The basic idea of the vector architecture is to collect data elements from memory, put them into a large set of registers, then operate on them using a pipelined execution unit, and finally write the result back to memory. The key feature of vector architecture is a set of vector registers.

The next picture shows a simplified scalar processor's datapath.

![](images/c6361996f819809b40a83b125c513e0fb09f0e80c89f5302ca35e5f4a36f43ee.jpg)

The next picture shows a simplified vector processor's datapath. The scalar operand in scalar architecture is expanded to vector operand in vector architecture. And the relative REGFILE and ALU are also vectorized.

![](images/60ae84393cddbfed41fb7a9fc8b7b5d57d3ce51e857d60faaa803722a97f301d.jpg)

# Vector VS Scalar

In order to visualize the characteristics of vector processors and conventional processors, we provide one case study. We want to use a vector processor and a conventional processor to perform the following operations, respectively.

$$
Z = a \times X + Y
$$

Here $X$, $Y$, $Z$ are 8-dimensional vectors, and each element of the vector is one 32-bit integer data; $a$ is a 32-bit integer scalar.

![](images/5e74d870f24d2a35d32d08bc778afc327ddc1e1aa8117a24275bc3d529b7c2c0.jpg)

The address of  $a$  in memory is in register x4, the base address of  $\$ X$  in memory is in register x5, the base address of  $\$ Y$  in memory is in register x6, and the base address of  $\$ Z$  in memory is in register x7.

<table><tr><td>variables</td><td>a</td><td>X</td><td>Y</td><td>Z</td></tr><tr><td>base address register</td><td>x4</td><td>x5</td><td>x6</td><td>x7</td></tr></table>

# Scalar Processor

The assembly codes based on the RISC-V instruction set are shown below.

```asm
addi x1, $zero, 1 ; set x1 = 1
lw x11, 0(x4) ; load scalar a
addi x12, $zero, 8 ; upper bound of what to load
loop:
lw x13, 0(x5) ; load X[i]
mul x13, x13, x11 ; a x X[i]
lw x14, 0(x6) ; load Y[i]
add x14, x14, x13 ; a x X[i] + Y[i]
sw x14, 0(x7) ; store Z[i]
addi x5, x5, 4 ; increment index to x
addi x6, x6, 4 ; increment index to y
addi x7, x7, 4 ; increment index to z
sub x12, x12, x1 ; x12 = x12 - 1
bne x12, $zero, loop ; check if done
```

# Vector Processor

The assembly codes based on the RISC-V Vector-Extension instruction set are shown below.

```asm
lw x11, 0(x4) ; load scalar a  
vle32.v v13, 0(x5) ; load vector X  
vmul.vx v14, v13, x11 ; a x X  
vle32.v v15, 0(x6) ; load vector Y  
vadd.vv v16, v14, v15 ; Z = a x X + Y  
sle32.v v16, 0(x7) ; store Z
```

# Comparison

By comparing two assembly code implementations, we can discover that the number of instructions of the vector processor is much less than the number of instructions of the scalar version. This is mainly due to the vector processor's ability to compute multiple data in parallel, eliminating the need to use for loop.

# Multi-Issue Processor

In computer architecture, multi-issue processors enable a processor to launch multiple instructions in a single clock cycle. There are mainly two categories of multi-issue processors: static multi-issue and dynamic multi-issue.

Dynamic multi-issue is implemented in hardware where the processor dynamically decides at runtime which instructions to issue, resolving dependencies and reordering instructions on-the-fly. This approach offers flexibility but adds hardware complexity and power consumption.

Static multi-issue, on the other hand, relies on the compiler to analyze and schedule instructions. It simplifies hardware but requires sophisticated compiler techniques and can be less adaptable to runtime conditions.

In this lab, we only focus on the static multi-issue processors.

The following figure gives an instruction processing timing diagram for a 2-issue pipelined processor. ALU/Branch instructions and Load/Store instructions can be seen to be processed in parallel at the same time. In order to support this operation, the hardware needs to have separate data paths for each of them.

<table><tr><td>Instruction type</td><td colspan="8">Pipe stages</td></tr><tr><td>ALU or branch instruction</td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td><td></td><td></td><td></td></tr><tr><td>Load or store instruction</td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td><td></td><td></td><td></td></tr><tr><td>ALU or branch instruction</td><td></td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td><td></td><td></td></tr><tr><td>Load or store instruction</td><td></td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td><td></td><td></td></tr><tr><td>ALU or branch instruction</td><td></td><td></td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td><td></td></tr><tr><td>Load or store instruction</td><td></td><td></td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td><td></td></tr><tr><td>ALU or branch instruction</td><td></td><td></td><td></td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td></tr><tr><td>Load or store instruction</td><td></td><td></td><td></td><td>IF</td><td>ID</td><td>EX</td><td>MEM</td><td>WB</td></tr></table>

To help understand the concepts, we rewrite the previous assembly code for a single-issue scalar processor as two-issue assembly code. Note that the code presented on the same line is executable in parallel.

```asm
addi x1, $zero, 1 ; lw x11, 0(x4) ; set x1 = 1 & load scalar a
addi x12, $zero, 8 ; nop
upper bound of what to load
loop:
nop ; lw x13, 0(x5) ; load X[i]
mul x13, x13, x11 ; lw x14, 0(x6) ; a
x X[i] & load Y[i]
add x14, x14, x13 ; nop
x X[i] + Y[i]
addi x5, x5, 4 ; sw x14, 0(x7) ; increment index to x & store Z[i]
addi x6, x6, 4 ; nop
increment index to y
addi x7, x7, 4 ; nop
increment index to z
sub x12, x12, x1 ; nop
x12 = x12 - 1
bne x12, $zero, loop ; nop
check if done
```

# RISC-V

RISC-V is an open-source instruction set architecture (ISA) based on the principles of Reduced Instruction Set Computing (RISC), with V denoting the fifth generation of RISC.

RISC-V instruction set includes RV32I and RV32M, RV32F, RV32D, RV32A, RV32V extensions. The instruction set RV32I is a fixed basic integer instruction set. RV32V is the vector extension.

In this lab, RV32I, RV32M and RV32V instruction sets are used.

# RV32I

The following figure shows six basic instruction formats of RV32l:

1. R-type instructions for register-register operations  
2. I-type instructions for short immediate and load operations  
3. S-type instructions for store operations  
4. B-type instructions for conditional jump operations  
5. U-type instructions for long immediate  
6. J-type instructions for unconditional jumps

<table><tr><td>31</td><td>30</td><td>25</td><td>24</td><td>21</td><td>20</td><td>19</td><td>15</td><td>14</td><td>12</td><td>11</td><td>8</td><td>7</td><td>6</td><td>0</td></tr><tr><td colspan="3">funct7</td><td colspan="3">rs2</td><td>rs1</td><td colspan="2">funct3</td><td colspan="3">rd</td><td colspan="2">opcode</td><td>R-type</td></tr><tr><td colspan="6">imm[11:0]</td><td>rs1</td><td colspan="2">funct3</td><td colspan="3">rd</td><td colspan="2">opcode</td><td>I-type</td></tr><tr><td colspan="3">imm[11:5]</td><td colspan="3">rs2</td><td>rs1</td><td colspan="2">funct3</td><td colspan="3">imm[4:0]</td><td colspan="2">opcode</td><td>S-type</td></tr><tr><td>imm[12]</td><td colspan="2">imm[10:5]</td><td colspan="3">rs2</td><td>rs1</td><td colspan="2">funct3</td><td>imm[4:1]</td><td colspan="2">imm[11]</td><td colspan="2">opcode</td><td>B-type</td></tr><tr><td colspan="9">imm[31:12]</td><td colspan="3">rd</td><td colspan="2">opcode</td><td>U-type</td></tr><tr><td>imm[20]</td><td colspan="3">imm[10:1]</td><td colspan="2">imm[11]</td><td colspan="3">imm[19:12]</td><td colspan="3">rd</td><td colspan="2">opcode</td><td>J-type</td></tr></table>

The following figure shows the instructions of each type in RV321.

<table><tr><td colspan="4">imm[31:12]</td><td>rd</td><td>0110111</td></tr><tr><td colspan="4">imm[31:12]</td><td>rd</td><td>0010111</td></tr><tr><td colspan="4">imm[20|10:1|11|19:12]</td><td>rd</td><td>1101111</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>000</td><td>rd</td><td>1100111</td></tr><tr><td>imm[12|10:5]</td><td>rs2</td><td>rs1</td><td>000</td><td>imm[4:1|11]</td><td>1100011</td></tr><tr><td>imm[12|10:5]</td><td>rs2</td><td>rs1</td><td>001</td><td>imm[4:1|11]</td><td>1100011</td></tr><tr><td>imm[12|10:5]</td><td>rs2</td><td>rs1</td><td>100</td><td>imm[4:1|11]</td><td>1100011</td></tr><tr><td>imm[12|10:5]</td><td>rs2</td><td>rs1</td><td>101</td><td>imm[4:1|11]</td><td>1100011</td></tr><tr><td>imm[12|10:5]</td><td>rs2</td><td>rs1</td><td>110</td><td>imm[4:1|11]</td><td>1100011</td></tr><tr><td>imm[12|10:5]</td><td>rs2</td><td>rs1</td><td>111</td><td>imm[4:1|11]</td><td>1100011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>000</td><td>rd</td><td>0000011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>001</td><td>rd</td><td>0000011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>010</td><td>rd</td><td>0000011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>100</td><td>rd</td><td>0000011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>101</td><td>rd</td><td>0000011</td></tr><tr><td>imm[11:5]</td><td>rs2</td><td>rs1</td><td>000</td><td>imm[4:0]</td><td>0100011</td></tr><tr><td>imm[11:5]</td><td>rs2</td><td>rs1</td><td>001</td><td>imm[4:0]</td><td>0100011</td></tr><tr><td>imm[11:5]</td><td>rs2</td><td>rs1</td><td>010</td><td>imm[4:0]</td><td>0100011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>000</td><td>rd</td><td>0010011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>010</td><td>rd</td><td>0010011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>011</td><td>rd</td><td>0010011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>100</td><td>rd</td><td>0010011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>110</td><td>rd</td><td>0010011</td></tr><tr><td colspan="2">imm[11:0]</td><td>rs1</td><td>111</td><td>rd</td><td>0010011</td></tr><tr><td>0000000</td><td>shamt</td><td>rs1</td><td>001</td><td>rd</td><td>0010011</td></tr><tr><td>0000000</td><td>shamt</td><td>rs1</td><td>101</td><td>rd</td><td>0010011</td></tr><tr><td>0100000</td><td>shamt</td><td>rs1</td><td>101</td><td>rd</td><td>0010011</td></tr><tr><td>0000000</td><td>rs2</td><td>rs1</td><td>000</td><td>rd</td><td>0110011</td></tr><tr><td>0100000</td><td>rs2</td><td>rs1</td><td>000</td><td>rd</td><td>0110011</td></tr><tr><td>0000000</td><td>rs2</td><td>rs1</td><td>001</td><td>rd</td><td>0110011</td></tr></table>

<table><tr><td colspan="2">0000000</td><td>rs2</td><td>rs1</td><td>010</td><td>rd</td><td>0110011</td><td>R sit</td></tr><tr><td colspan="2">0000000</td><td>rs2</td><td>rs1</td><td>011</td><td>rd</td><td>0110011</td><td>Rsltu</td></tr><tr><td colspan="2">0000000</td><td>rs2</td><td>rs1</td><td>100</td><td>rd</td><td>0110011</td><td>R xor</td></tr><tr><td colspan="2">0000000</td><td>rs2</td><td>rs1</td><td>101</td><td>rd</td><td>0110011</td><td>R srl</td></tr><tr><td colspan="2">0100000</td><td>rs2</td><td>rs1</td><td>101</td><td>rd</td><td>0110011</td><td>R sra</td></tr><tr><td colspan="2">0000000</td><td>rs2</td><td>rs1</td><td>110</td><td>rd</td><td>0110011</td><td>R or</td></tr><tr><td colspan="2">0000000</td><td>rs2</td><td>rs1</td><td>111</td><td>rd</td><td>0110011</td><td>R and</td></tr><tr><td>0000</td><td>pred</td><td>succ</td><td>00000</td><td>000</td><td>00000</td><td>0001111</td><td>I fence</td></tr><tr><td>0000</td><td>0000</td><td>0000</td><td>00000</td><td>001</td><td>00000</td><td>0001111</td><td>I fence.i</td></tr><tr><td colspan="3">00000000000</td><td>00000</td><td>00</td><td>00000</td><td>1110011</td><td>I ecall</td></tr><tr><td colspan="3">00000000000</td><td>00000</td><td>000</td><td>00000</td><td>1110011</td><td>I ebreak</td></tr><tr><td colspan="3">csr</td><td>rs1</td><td>001</td><td>rd</td><td>1110011</td><td>I csrrw</td></tr><tr><td colspan="3">csr</td><td>rs1</td><td>010</td><td>rd</td><td>1110011</td><td>I csrrs</td></tr><tr><td colspan="3">csr</td><td>rs1</td><td>011</td><td>rd</td><td>1110011</td><td>I csrcc</td></tr><tr><td colspan="3">csr</td><td>zimm</td><td>101</td><td>rd</td><td>1110011</td><td>I csrwi</td></tr><tr><td colspan="3">csr</td><td>zimm</td><td>110</td><td>rd</td><td>1110011</td><td>I cssrrsi</td></tr><tr><td colspan="3">csr</td><td>zimm</td><td>111</td><td>rd</td><td>1110011</td><td>I csrcci</td></tr></table>

# RV32M

RV32M adds integer multiplication and division instructions to RV32I, including instructions for signed and unsigned integers, divide (div) and divide unsigned (divu), which put the quotient into the target register. In a few cases, the programmer needs the remainder rather than the quotient, so RV32M provides remainder (rem) and remainder unsigned (remu), which writes the remainder to the target register instead of the quotient.

To correctly obtain a signed or unsigned 64-bit product, RISC-V comes with four multiplication instructions. To get the integer 32-bit product (the lower 32 bits of 64 bits) use the mul instruction. To get the high 32 bits, use the mulh instruction for signed operands, the mulhu instruction for unsigned operands and the mulhsu instruction for signed multiplier and unsigned multiplicand.

<table><tr><td>31</td><td>25</td><td>24</td><td>20</td><td>19</td><td>15</td><td>14</td><td>12</td><td>11</td><td>7</td><td>6</td><td>0</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>000</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R mul</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>001</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R mulh</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>010</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R mulsu</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>011</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R mulhu</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>100</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R div</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>101</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R divu</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>110</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R rem</td></tr><tr><td>0000001</td><td>rs2</td><td></td><td>rs1</td><td></td><td>111</td><td></td><td>rd</td><td></td><td>0110011</td><td></td><td>R remu</td></tr></table>

# RV32V

The opcodes of the instructions in the vector extension either are the same as LOAD-FP and STORE-FP or use OP-V.

Format for Vector Load Instructions under LOAD-FP major opcode  

<table><tr><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td></td><td>0</td></tr><tr><td></td><td>nf</td><td></td><td>mew</td><td></td><td>mop</td><td>vm</td><td></td><td></td><td>lumop</td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td>width</td><td></td><td></td><td>vd</td><td></td><td></td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td></td><td>nf</td><td></td><td>mew</td><td></td><td>mop</td><td>vm</td><td></td><td></td><td>rs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td>width</td><td></td><td></td><td>vd</td><td></td><td></td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>4</td><td></td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td></td><td>nf</td><td></td><td>mew</td><td></td><td>mop</td><td>vm</td><td></td><td></td><td>vs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td>width</td><td></td><td></td><td>vd</td><td></td><td></td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></table>

Format for Vector Store Instructions under STORE-FP major opcode  

<table><tr><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td></td><td>0</td></tr><tr><td></td><td>nf</td><td></td><td>mew</td><td></td><td>mop</td><td>vm</td><td></td><td>sumop</td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td></td><td>width</td><td></td><td></td><td>vs3</td><td></td><td></td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td colspan="18">base address</td><td colspan="5">store data</td><td colspan="7">VS* unit-stride</td></tr><tr><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td></td><td>nf</td><td></td><td>mew</td><td></td><td>mop</td><td>vm</td><td></td><td>rs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td></td><td>width</td><td></td><td></td><td>vs3</td><td></td><td></td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td colspan="18">stride</td><td colspan="5">store data</td><td colspan="6">VSS* strided</td><td></td></tr><tr><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td></td><td>nf</td><td></td><td>mew</td><td></td><td>mop</td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td></td><td>width</td><td></td><td></td><td>vs3</td><td></td><td></td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td colspan="18">address offsets</td><td colspan="5">store data</td><td colspan="6">VSX* indexed</td><td></td></tr></table>

Formats for Vector Arithmetic Instructions under OP-V major opcode  

<table><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td></td><td>vs1</td><td></td><td></td><td></td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td colspan="24">OPIVV</td><td></td></tr><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td></td><td>vs1</td><td></td><td>0</td><td>0</td><td>1</td><td>vd / rd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="24">OPFVV</td><td></td></tr><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td></td><td>vs1</td><td>0</td><td>1</td><td>0</td><td>vd / rd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="24">OPMVV</td><td></td></tr><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td>imm[4:0]</td><td></td><td>0</td><td>1</td><td>1</td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="24">OPIVI</td><td></td></tr><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td>rs1</td><td></td><td>1</td><td>0</td><td>0</td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="24">OPIVX</td><td></td></tr><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td>rs1</td><td></td><td>0</td><td>1</td><td>1</td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="24">OPFVF</td><td></td></tr><tr><td>31</td><td>funct6</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td>vm</td><td>vs2</td><td>rs1</td><td></td><td>2</td><td>1</td><td>0</td><td>vd / rd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr></table>

Formats for Vector Configuration Instructions under OP-V major opcode  

<table><tr><td>31</td><td>30</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td>0</td><td></td><td></td><td></td><td></td><td></td><td>zimm[10:0]</td><td></td><td></td><td></td><td></td><td></td><td>rs1</td><td></td><td></td><td>1</td><td>1</td><td>1</td><td></td><td></td><td>rd</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="30">vsetvli</td></tr><tr><td>31</td><td>30</td><td>29</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td>1</td><td>1</td><td></td><td></td><td></td><td></td><td>zimm[9:0]</td><td></td><td></td><td></td><td></td><td></td><td>uimm[4:0]</td><td></td><td></td><td>1</td><td>1</td><td>1</td><td></td><td></td><td>rd</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="30">vsetivli</td></tr><tr><td>31</td><td>30</td><td></td><td></td><td></td><td>25</td><td>24</td><td></td><td></td><td></td><td>20</td><td>19</td><td></td><td></td><td>15</td><td>14</td><td></td><td>12</td><td>11</td><td></td><td></td><td>7</td><td>6</td><td></td><td></td><td></td><td></td><td></td><td>0</td><td></td></tr><tr><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td></td><td>rs2</td><td></td><td></td><td>rs1</td><td></td><td></td><td>1</td><td>1</td><td>1</td><td></td><td></td><td>rd</td><td></td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td colspan="30">vsetvi</td></tr></table>

Category operation  
Operands  
Type of scalar operand  

<table><tr><td>OPI-VV</td><td>integer operation</td><td>vector-vector</td><td>N/A</td></tr><tr><td>OPF-VV</td><td>floating-point operation</td><td>vector-vector</td><td>N/A</td></tr><tr><td>OPM-VV</td><td>mask operation</td><td>vector-vector</td><td>N/A</td></tr><tr><td>OPI-VI</td><td>integer operation</td><td>vector-immediate</td><td>imm[4:0]</td></tr><tr><td>Category</td><td>operation</td><td>Operands</td><td>Type of scalar operand</td></tr><tr><td>OPI-VX</td><td>integer operation</td><td>vector-scalar</td><td>GPR(general purpose registers)× register rs1</td></tr><tr><td>OPI-VF</td><td>integer operation</td><td>vector-scalar</td><td>FP f register rs1</td></tr><tr><td>OPM-VX</td><td>mask operation</td><td>vector-scalar</td><td>GPR × register rs1</td></tr></table>

In this lab, we only focus on the OPIV, OPIVI and OPIX.

# Lab Goals

The purpose of the lab is to design a two-issue RISC-V processor, which can perform Matrix Multiplication operation and Softmax operation.

# Matrix Multiplication

The dimensions of all matrices are 8 by 8 and each element in the matrices is a 32-bit data. Matrix-A is the weight matrix. Matrix-B is the input matrix. Matrix-C is the bias matrix. Matrix-D is the output matrix.

For simplicity, we assume no data overflow will occur.

![](images/49ea697289db42bffcf801afe06fe0b5cb56917c6dfaa78904589d35088637ba.jpg)

$$
D = A \cdot B + C
$$

A: Weights. B: Input Matrix. C: Bias. D: Output Matrix

$$
D = \left( \begin{array}{c c c c} A _ {1, 1} & A _ {1, 2} & \dots & A _ {1, 8} \\ A _ {2, 1} & A _ {2, 2} & \dots & A _ {2, 8} \\ \vdots & \vdots & \ddots & \vdots \\ A _ {8, 1} & A _ {8, 2} & \dots & A _ {8, 8} \end{array} \right) \cdot \left( \begin{array}{c c c c} B _ {1, 1} & B _ {1, 2} & \dots & B _ {1, 8} \\ B _ {2, 1} & B _ {2, 2} & \dots & B _ {2, 8} \\ \vdots & \vdots & \ddots & \vdots \\ B _ {8, 1} & B _ {8, 2} & \dots & B _ {8, 8} \end{array} \right) + \left( \begin{array}{c c c c} C _ {1, 1} & C _ {1, 2} & \dots & C _ {1, 8} \\ C _ {2, 1} & C _ {2, 2} & \dots & C _ {2, 8} \\ \vdots & \vdots & \ddots & \vdots \\ C _ {8, 1} & C _ {8, 2} & \dots & C _ {8, 8} \end{array} \right)
$$

![](images/63b19909efaabb6d49a44bcc5b8ee4e9e1f8e09a4fb3c203328b15d308fd293d.jpg)

![](images/688a1927ce8c9b74bb09d6532e45cd1abed2aaa73dc9ff82f738755cecff3996.jpg)

![](images/ae384735263ee6d438b89cddd7f3fbc30ad600c584a540f2ee230ecbaeb1446d.jpg)

![](images/aaacaaebe06c90f19d766824e87ce0c186d59ebcef6d51272cc7cb0114e02e35.jpg)

# Softmax

The softmax (also called softmax) function is a smooth approximation to the argmax function. The softmax function  $\$ 0(x)$  operates on a vector of real-valued scores  $\$ x\_i$  and normalizes them into a probability distribution

$$
\$ \$ p (i) = \backslash s i g m a (x) _ {i} = \backslash c f r a c \{e ^ {\wedge} \{x _ {i} \} \} \{\backslash s u m _ {k} e ^ {\wedge} \{x _ {k} \} \} \$ \$
$$

where $p_i\geq 0$ and $\sum_{i} p_i = 1$ [1].

Float data occurs in div and exp operation. Since the hardware only supports 32-bit integers, we need to modify the way data is stored so that the hardware can handle fractional (decimal) values.

We can use the Q16.16 format to compute the softmax. In a 32-bit register, the highest bit represents the sign, bits 30-16 represent the integer part, and bits 15-0 represent the fractional part. Essentially, it is a 32-bit integer. If we interpret the value in the register directly as an int32, then the Q16.16 value is int32 >> 16.

For example, if we want to store a Q16.16 number $x$ in a processor that only supports int32, we simply multiply the number by $2^{\wedge}\{16\}$ ($x$ << 16) to convert it into an integer, which can then be stored in the processor. When we want to retrieve the actual value from the stored number, we just need to right-shift it by 16 bits.

# Q16.16 format

<table><tr><td>bit</td><td>31</td><td>30 - 16</td><td>15 - 0</td></tr><tr><td></td><td>sign</td><td>integer part</td><td>fractional part</td></tr></table>

# Softmax algorithm with Q16.16

The test script will randomly generate 8 Q16.16-format values in memory. You need to read them to your processor, compute the softmax according to the following algorithm, and then write the results back to the corresponding memory addresses. The exponential (exp) operation is implemented directly using a lookup table (LUT), which is also preloaded to memory by the test script.

During this Softmax, we do not calculate \(\$ e^{\wedge}\{x_i\}\) directly because it may lead to data overflow. Instead, we perform exp operation on the delta between max(\(x_i\)) and each \)x_i\$. The delta values are clipped to [-8, 0] because \)delta = x_i - max(x_i)\( will never be larger than 0, and \)e^{\wedge}\{-8\}\( is small enough to regard it as 0.

The 64-element LUT supports input value from -8 to 0, with the step of 8/63.

```txt
Q = 16  
Q_16 = 1 << Q  
LUT_SIZE = 64  
X_q16_16 = original Q16.16 values (8-element vector)  
x_max = max(X_q16_16[i])  
DELTA[i] = X_q16_16 - x_max  
DELTA = clip(DELTA, -8 * Q_16, 0)  
(change values in DELTA smaller than -8 * Q_16 to -8 * Q_16, larger than 0 to 0)  
get exp(DELTA) using LUT:
```

```c
idx_num[i] = (DELTA[i] + 8 * Q_16) * (LUT_SIZE - 1)
idx[i] = (idx_num[i] // (8 * Q_16))
idx[i] = clip(idx[i], 0, LUT_SIZE - 1)
exp(DELTA[i]) = LUT[ idx[i]]
```

HINT: If you get confused with this pseudocode, please refer to extract_golden() in projects/lab4/tools/test/Test3_2.py. But do not modify this file.

# !!! IMPORTANT !!!

You only need to store the Q16.16-format result to memory.

# Lab Setup

# Data Mapping

The mapping of the data of the matrix to the addresses of the memory are shown in the following figures. For all the matrixes involved, we provide two ways of data mapping.

One is to store the matrices in row-major.

![](images/de1a4268be97c7378c14fb6800a505a0f64ab5bc5d97db002749da8a7516363d.jpg)

![](images/3affa7d2bc9b76a75436077e6a7122c79f6e99694b30a2a81b688bbf4ad26310.jpg)

![](images/6d2c1d699d3b51e0fc622a758463576498d3343d329f8a63ad8309c5fc544449.jpg)

![](images/c5ac034570ab42fb0d1e3a818f3d7757ce3061a67661686b762561b5a0d2eb2a.jpg)

The other is to store the transposed matrices in column-major.

![](images/18d1390064b19723122ce7e779788adb5ca603e7fe5ad7eb4898a91eb22db5fa.jpg)

![](images/25a51f96a33789c5cc5970d25f16eddae77df147068780b060b33a6fc540a2fe.jpg)

![](images/3178d852613184f6fcec52fbc478bbf35acb4acdf0559df6ef06b92336876662.jpg)

![](images/e56f98d365e469523f13a28c69c01beda3438b44002473379d067ae876dabf5f.jpg)

The base address is the first element's address of the data block. The detailed base address information is shown below.

<table><tr><td>Data</td><td>Base Address</td></tr><tr><td>Instructions</td><td>0x80000000</td></tr><tr><td>Matrix-A</td><td>0x80100000</td></tr><tr><td>Matrix-A.T</td><td>0x80200000</td></tr><tr><td>Matrix-B</td><td>0x80300000</td></tr><tr><td>Matrix-B.T</td><td>0x80400000</td></tr><tr><td>Matrix-C</td><td>0x80500000</td></tr><tr><td>Matrix-C.T</td><td>0x80600000</td></tr><tr><td>Matrix-D</td><td>0x80700000</td></tr><tr><td>Matrix-D.T</td><td>0x80800000</td></tr><tr><td>original value for Softmax</td><td>0x81000000</td></tr><tr><td>LUT for Softmax exp operation</td><td>0x81000000 + 32</td></tr></table>

# Data

# Base Address

result of Softmax

0x81100000

# Memory Interface

To reduce the challenge of accessing memory, we use 3 simplified memory access models.

Caution: The memory interface has been greatly simplified here to reduce the difficulty of the experiment. The memory can read out data in one clock cycle (processor clock) during simulation. But in practice, the memory's read and write speed are much lower than the processor's main frequency. In addition, the memory and the processor are often connected through the bus, and instruction storage and data storage do not necessarily use two sets of read-out interfaces. Thus, the processor needs to arbitrate whether to read data or the instructions.

In this lab, the INST_RAMHelper is used to get the instructions from the memory.

```txt
module INST_RAMHelper( input clk, input ren, // read enable input [31:0] raddr, // read address output [31:0] rdata // read data );
```

The SCALAR_RAMHelper is used to get the scalar data from the memory.

```verilog
module SCALAR_RAMHelper(  
input clk,  
input ren, // read enable  
input [31:0] rdrd, // read address  
output [31:0] rdata, // read data  
input wen, // write enable  
input [31:0] waddr, // write address  
input [31:0] wdata, // write data  
input [31:0] wmask // write mask);
```

The Vector_RAMHelper is used to get the vector data from the memory.

```verilog
module VECTOR_RAMHelper(  
input clk,  
input ren, // read enable  
input [31:0] rdrd, // read address  
output [255:0] rdata, // read data  
input wen, // write enable  
input [31:0] waddr, // write address  
input [255:0] wdata, // write data
```

```txt
input [255:0] wmask // write mask);
```

# Data Access

The minimum stride of access memory is 1 Byte (8 bits).

For 32-bit data access, the access address's step is 4. For example, data A and data B are both 32-bit data and stored in adjacent locations. Data A's address is 0x0000_1000 and data B's address is 0x0000_1004.

RISC-V Vector Extension

# Vector Data Width Setting

The vector extended RISCV processor contains 32 vector registers,  $x0 - x31$ .

VLEN: represents a fixed bit width for each vector register.

SEW: represents the bit width of the selected vector element. It is controlled by register vsew[2:0].

Taking VLEN=128bits as an example, the correspondence between the number of elements contained in each vector register and SEW is shown in the following table.

<table><tr><td>VELN</td><td>SEW</td><td>Elements Per Vector Register</td></tr><tr><td>128</td><td>64</td><td>2</td></tr><tr><td>128</td><td>32</td><td>4</td></tr><tr><td>128</td><td>16</td><td>8</td></tr><tr><td>128</td><td>8</td><td>16</td></tr></table>

LMUL: represents the vector length multiplier. If greater than 1, it represents the default number of vector registers.

VLMAX: represents the maximum number of vector elements that a vector instruction can operate on.  
VLMAX=LMUL*VLEN/SEW.

# !!! IMPORTANT !!!

To simplify the experiment, we set all the above parameters to fixed values.

<table><tr><td colspan="8">vector register&lt;n&gt; VLEN = 256-bits SEW=32-bits</td></tr><tr><td>32-bits</td><td>32-bits</td><td>32-bits</td><td>32-bits</td><td>32-bits</td><td>32-bits</td><td>32-bits</td><td>32-bits</td></tr><tr><td>seg&lt;0&gt;</td><td>seg&lt;1&gt;</td><td>seg&lt;2&gt;</td><td>seg&lt;3&gt;</td><td>seg&lt;4&gt;</td><td>seg&lt;5&gt;</td><td>seg&lt;6&gt;</td><td>seg&lt;7&gt;</td></tr></table>

# Mask Setting

In the arithmetic and access instructions, you can choose whether to use the mask or not, and whether to enable the function is controlled by the vm bit in the instruction.

# !!! IMPORTANT !!!

In this experiment, the mask is not used by default. And vm is set to 1 in all instructions to disable the mask function.

# Load/Store Setting

There are 3 different access modes for RISC-V vector extensions: unit-stride, strided, indexed.

The unit-stride operation accesses consecutive elements stored in memory starting at the base effective address.

The strided operation accesses the first memory element at the base effective address, and then accesses subsequent elements at the address increment given by the byte offset contained in the x register specified by rs2.

The indexed operation adds the vector offset specified by vs2 to the base effective address to obtain the effective address of each element.

The following images describe the format of the access command and its specific meaning.

Format for Vector Load Instructions under LOAD-FP major opcode

![](images/1d4de4952b96abb8e879e402f1d61f5a48223c068fbcaa9f98a9f028967d2bfa.jpg)

![](images/5b628b1fd30e8ab69a474396bebc638aeca0cab85a2e64791e0aa99711dbda92.jpg)

![](images/2b5e6de565d4e842a4103124961bd4d13c7985cb7682fdcb14c07f14b739b87a.jpg)

![](images/0c8318f633b725a451b038fd47546a074fcf9f19f7a9abe1e1e14c741448d53b.jpg)

![](images/57b4323d04448ad6653ec79bb96e358e6188bde3d521b874fefe7e04301414e3.jpg)

Format for Vector Store Instructions under STORE-FP major opcode

![](images/87a3661bb71a1d64475700a403978ef5ba5a3a103c079bd03bbbedaff3375637.jpg)

![](images/c98d36af6e3642df4962b9a544fb9296fc1007fad94957c0f14d4239cc409593.jpg)

![](images/ee0e3ae60875e5c5265e69ea734208b60c61ae6bf17b875186f441e5e8a24b8f.jpg)

![](images/13b2599bc10b5b8a5e9b94aa6efac1dd1b43417b113470c2938c63aefae61448.jpg)

![](images/158a203b0a679c79288aa4f45bc1da6e9abac5cc3817a5b464d61ad50a0a01b1.jpg)  
address offsets base address store data VSX\* indexed

<table><tr><td>Field</td><td>Description</td></tr><tr><td>rs1[4:0]</td><td>specifies x register holding base address</td></tr><tr><td>rs2[4:0]</td><td>specifies x register holding stride</td></tr><tr><td>vs2[4:0]</td><td>specifies v register holding address offsets</td></tr><tr><td>vs3[4:0]</td><td>specifies v register holding store data</td></tr><tr><td>vd[4:0]</td><td>specifies v register destination of load</td></tr><tr><td>vm</td><td>specifies whether vector masking is enabled (0 = mask enabled, 1 = mask disabled)</td></tr><tr><td>width[2:0]</td><td>specifies size of memory elements, and distinguishes from FP scalar</td></tr><tr><td>mew</td><td>extended memory element width. See Vector Load/Store Width Encoding</td></tr><tr><td>mop[1:0]</td><td>specifies memory addressing mode</td></tr><tr><td>nf[2:0]</td><td>specifies the number of fields in each segment, for segment load/stores</td></tr><tr><td>lumop[4:0]/sumop[4:0]</td><td>are additional fields encoding variants of unit-stride instructions</td></tr></table>

# !!! IMPORTANT !!!

In order to simplify the experiment, this experiment only needs to support the access mode of unit-stripe. In addition, the nf, mew, mop, and lumop bits of the access instruction can be set to the default value of 0.

# Assembly Code

The assembly code needs to be translated into machine code by the translator in order to be recognized and executed by the processor. A simple translator is provided in the experimental environment to translate the assembly code into a bin file. Using this translator requires the assembly code to be written in the specified format.

If you are not familiar with the assembly code, you can try to use the RISC-V assembler and runtime simulator.

TheThirdOne/rars: RARS -- RISC-V Assembler and Runtime Simulator (github.com)

The file demo . asm in the asm folder shows an example code that calls some of the used instructions in the lab. Please refer to the format of this code to write assembly code.

The scalar registers' names are x0-x31. And x0 can be replaced with zero.

The vector registers' names are vx0-vx31. And vx0 can be replaced with vzero.

```asm
;  
; scalar instructions  
;  
; load matrix_A base-address 0x80100000 to the register x5  
; ( the immediate number is already shifted 12-bits here )  
lui x5, 2148532224 ; x5 = 0x80100000  
addi x6, x5, 4 ; x6 = x5+4  
lw x7, 0(x5) ; x7 = A[0][0]  
lw x8, 4(x6) ; x18 = A[0][2]  
slt i x9, x7, 64 ; x9 = (x7<64)  
addi x9, zero, 1 ; x9 = 1  
add and x11, x10, x9 ; x10 = x9+x9  
mul x12, x10, x10 ; x12 = x10 * x10  
sll x13, x9, 1 ; x13 = x9<<1  
addi x9, zero, 4 ; assign 4 to x9  
addi x10, zero, 0 ; assign 0 to x10  
loop:  
addi x10, x10, 1 ; assign x10+1 to x10  
mul x12, x12, x10 ; assign x12*x10 to x12  
blt x10, x9, loop ; if( x10<x9 ) jump to loop  
sw x12, 0(x5) ; A[0][0] = x12
```

```asm
jal x1, label ; jump to label   
;   
; vector instructions   
;   
label:   
vle32.v vx2, x5, 1 ; vx2 = A[0][0:7]   
vadd.vi vx3, vzero, 1 , 1 ; vx3[i] = 1   
vadd.vx vx3, vzero, x12 , 1 ; vx3[i] = x12   
vmul.vv vx4, vx2, vx3 , 1 ; vx4[i] = vx2[i] * vx3[i]   
vsub.vv vx4, vx2, vx3 , 1 ; vx4[i] = vx2[i] - vx3[i]   
vdiv.vx vx4, vx2, x3 , 1 ; vx4[i] = vx2[i] / x3[i]   
vmin.vv vx4, vx2, vx3, 1 ; vx4[i] = min(vx2[i], vx3[i])   
vmax.vv vx4, vx2, vx3, 1 ; vx4[i] = max(vx2[i], vx3[i])   
vsra.vx vx4, vx2, x3, 1 ; vx4[i] = vx2[i] >> x3[i]   
vredsum.vs vx3, vx2, vx0, 1 ; vx3[0] = sum( vx0[0], vx2[0], vx2[1],   
vx2[2]..., vx2[-1])   
vredmax.vs vx3, vx2, vx0, 1 ; vx3[0] = max( vx0[0], vx2[0], vx2[1],   
vx2[2]..., vx2[-1])   
vmv.x.s x8, vx2, vx0, 1 ; x8 = vx2[0]   
vmv.v.x vx3, vx31, x8, 1 ; vx3[i] = x8   
vse32.v vx4, x5, 1 ; A[0][0:7] = vx4
```

# Detailed Tasks

# Task1.1 - Single-Issue Processor

You need to complete the instructions in the table below and pass the test. Here sext means signed extended and Mem means Memory. The logic operation (and) is bit-wise.

<table><tr><td>Type</td><td>Instruction</td><td>Format</td><td>Implementation</td></tr><tr><td>R</td><td>mul</td><td>mul rd, rs1, rs2</td><td>x[rd] = x[rs1] * x[rs2]</td></tr><tr><td>R</td><td>add</td><td>add rd, rs1, rs2</td><td>x[rd] = x[rs1] + x[rs2]</td></tr><tr><td>R</td><td>and</td><td>and rd, rs1, rs2</td><td>x[rd] = x[rs1] &amp; x[rs2]</td></tr><tr><td>R</td><td>sll</td><td>sll rd, rs1, rs2</td><td>x[rd] = x[rs1] &lt;&lt; x[rs2]</td></tr><tr><td>I</td><td>addi</td><td>addi rd, rs1, imm</td><td>x[rd] = x[rs1] + sext(immediate)</td></tr><tr><td>I</td><td>slti</td><td>slti rd, rs1, imm</td><td>x[rd] = x[rs1] &lt; sext(immediate)</td></tr><tr><td>I</td><td>lw</td><td>lw rd, offset(rs1)</td><td>x[rd] = Mem[x[rs1] + sext(offset)]</td></tr><tr><td>S</td><td>sw</td><td>sw rs2, offset(rs1)</td><td>Mem[x[rs1] + sext(offset)] = x[rs2]</td></tr><tr><td>B</td><td>blt</td><td>blt rs1, rs2, offset</td><td>if (x[rs1] &lt; x[rs2]) pc += sext(offset)</td></tr><tr><td>U</td><td>lui</td><td>lui rd,imm</td><td>x[rd] = immediate[31:12] &lt;&lt; 12</td></tr><tr><td>J</td><td>jal</td><td>jal rd,offset</td><td>x[rd] = pc+4; pc += sext(offset)</td></tr></table>

To better understand the meaning of these instructions, you can refer to this.  
To get the corresponding instruction format, you can refer to this.  
The code templates have been provided for the implementation of several instructions.  
HINT: Finish the modules under /src/vsrc/components/single-issue. Top module (/src/vsrc/rvcpu/rvcpu_single-issue.v) is already given.

# Task1.2 - Assembly Code 1

After completing Task 1.1, we have been able to use these instructions to perform some useful computation. Now, use these instructions to complete the MAC Operation.

HINT: Finish the code in /src/asm/task1_2.asm.

# Task2.1 - Two-Issue Processor

Extend the scalar processor to a two-issue processor, with one datapath supporting scalar and another supporting vector operations. For vector operations, you need to complete the instructions in the table below and pass the test.

<table><tr><td>Type</td><td>Instruction</td><td>func6</td><td>width</td><td>Format</td><td>Implementation</td></tr><tr><td>LOAD</td><td>vle32.v</td><td>000000</td><td>110</td><td>vle32.v vd, rs1, vm</td><td>vd = Mem[x[rs1]]</td></tr><tr><td>STORE</td><td>vse32.v</td><td>000000</td><td>110</td><td>vse32.v vs3, rs1, vm</td><td>Mem[x[rs1]] = vs3</td></tr><tr><td>Type</td><td>Instruction</td><td>func6</td><td>func3</td><td>Format</td><td>Implementation</td></tr><tr><td>OPIVV</td><td>vadd.vv</td><td>000000</td><td>000</td><td>vadd.vv vd, vs2, vs1, vm</td><td>vd[i] = vs2[i] + vs1[i]</td></tr><tr><td>OPIVX</td><td>vadd.vx</td><td>000000</td><td>100</td><td>vadd.vx vd, vs2, rs1, vm</td><td>vd[i] = vs2[i] + x[rs1]</td></tr><tr><td>OPIVI</td><td>vadd.vi</td><td>000000</td><td>011</td><td>vadd.vi vd, vs2, imm, vm</td><td>vd[i] = vs2[i] + imm</td></tr><tr><td>OPIVV</td><td>vsub.vv</td><td>000010</td><td>000</td><td>vsub.vv vd, vs2, vs1, vm</td><td>vd[i] = vs2[i] - vs1[i]</td></tr><tr><td>OPIVX</td><td>vsub.vx</td><td>000010</td><td>100</td><td>vsub.vx vd, vs2, rs1, vm</td><td>vd[i] = vs2[i] - x[rs1]</td></tr><tr><td>OPMVV</td><td>vmul.vv</td><td>100101</td><td>010</td><td>vmul.vv vd, vs2, vs1, vm</td><td>vd[i] = vs2[i] * vs1[i]</td></tr><tr><td>OPMVX</td><td>vmul.vx</td><td>100101</td><td>110</td><td>vmul.vx vd, vs2, rs1, vm</td><td>vd[i] = vs2[i] * x[rs1]</td></tr><tr><td>OPMVV</td><td>vdiv.vv</td><td>100001</td><td>010</td><td>vdiv.vv vd, vs2, vs1, vm</td><td>vd[i] = vs2[i] / vs1[i]</td></tr><tr><td>Type</td><td>Instruction</td><td>func6</td><td>func3</td><td>Format</td><td>Implementation</td></tr><tr><td>OPMVX</td><td>vdiv.vx</td><td>100001</td><td>110</td><td>vdiv.vx vd, vs2, rs1, vm</td><td>vd[i] = vs2[i] / x[rs1]</td></tr><tr><td>OPMVV</td><td>vmv.x.s</td><td>010000</td><td>010</td><td>vmv.x.s rd, vs2, vx0, vm</td><td>x[rd] = vs2[0] (vs1=0)</td></tr><tr><td>OPIVX</td><td>vmv.v.x</td><td>010111</td><td>100</td><td>vmv.v.x vd, rs1, vm</td><td>vd[i] = x[rs1]</td></tr></table>

NOTICE: Please set vs1 to vx0 when using vmv.x.s. You can set vs2 to any vx when using vmv.v.x.

The formats of relative instructions are shown below.

Instruction Format  

<table><tr><td rowspan="3">vle32.v</td><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>vm</td><td>0</td><td>0</td><td>0</td><td>0</td><td>rs1</td><td></td><td>1</td><td>1</td><td>0</td><td></td><td>vd</td><td></td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td>nf</td><td colspan="6">mew mop</td><td colspan="6">lumop</td><td colspan="6">width</td><td colspan="7">opcode (VL* unit-stride)</td></tr><tr><td rowspan="3">vse32.v</td><td>31</td><td>29</td><td>28</td><td>27</td><td>26</td><td>25</td><td>24</td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>vm</td><td>0</td><td>0</td><td>0</td><td>0</td><td>rs1</td><td></td><td>1</td><td>1</td><td>0</td><td></td><td>vs3</td><td></td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td></tr><tr><td>nf</td><td colspan="6">mew mop</td><td colspan="6">sumop</td><td colspan="6">width</td><td colspan="7">opcode (VS* unit-stride)</td></tr><tr><td rowspan="16">Arithmetic Inst</td><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>vs1</td><td></td><td></td><td></td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>vs1</td><td></td><td>0</td><td>0</td><td>1</td><td></td><td>vd / rd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>vs1</td><td></td><td>0</td><td>1</td><td>0</td><td></td><td>vd / rd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>imm[4:0]</td><td></td><td>0</td><td>1</td><td>1</td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td>1</td><td>0</td><td>0</td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td>1</td><td>0</td><td>1</td><td></td><td>vd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>31</td><td></td><td></td><td>26</td><td>25</td><td>24</td><td></td><td></td><td>20</td><td>19</td><td></td><td>15</td><td>14</td><td>12</td><td>11</td><td></td><td>7</td><td>6</td><td></td><td></td><td>0</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>funct6</td><td></td><td></td><td>vm</td><td></td><td>vs2</td><td></td><td></td><td></td><td>rs1</td><td></td><td>1</td><td>0</td><td>0</td><td></td><td>vd / rd</td><td></td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td></td></tr><tr><td>OPVX</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>OPFV</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></table>

HINT: Finish the modules under /src/vsrc/components/two-issue and /src/vsrc/rvcpu/rvcpu_two-issue.v.

Set vid wb_from_rs1 to 1 when vmv.v.x inst is activated.

# Task2.2 - Assembly Code 2

Vector processors can handle MAC operations more efficiently. Now, use the vector instructions to do this operation.

Note that you are only allowed to use instructions from the instruction set RV-32I and RV-32V. In other words, you are not allowed to use the mul instruction from RV-32M.

HINT: Finish the code in /src/asm/task2_2.asm.

# Task3.1 - Two-Issue Processor with more instructions

Enable the processor to support more instructions required for Softmax operation in the table below and pass the test.

<table><tr><td>Type</td><td>Instruction</td><td>func6</td><td>func3</td><td>Format</td><td>Implementation</td></tr><tr><td>OPIVV</td><td>vmin.vv</td><td>000101</td><td>000</td><td>vmin.vv vd, vs2, vs1, vm</td><td>vd[i] = min(vs2[i], vs1[i])</td></tr><tr><td>OPIVX</td><td>vmin.vx</td><td>000101</td><td>100</td><td>vmin.vx vd, vs2, rs1, vm</td><td>vd[i] = min(vs2[i], x[rs1])</td></tr><tr><td>OPIVV</td><td>vmax.vv</td><td>000111</td><td>000</td><td>vmax.vv vd, vs2, vs1, vm</td><td>vd[i] = max(vs2[i], vs1[i])</td></tr><tr><td>OPIVX</td><td>vmax.vx</td><td>000111</td><td>100</td><td>vmax.vx vd, vs2, rs1, vm</td><td>vd[i] = max(vs2[i], x[rs1])</td></tr><tr><td>OPIVV</td><td>vsra.vv</td><td>101001</td><td>000</td><td>vsra.vv vd, vs2, vs1, vm</td><td>vd[i] = vs2[i] &gt;&gt;&gt; vs1[i]</td></tr><tr><td>OPIVX</td><td>vsra.vx</td><td>101001</td><td>100</td><td>vsra.vx vd, vs2, rs1, vm</td><td>vd[i] = vs2[i] &gt;&gt;&gt; x[rs1]</td></tr><tr><td>OPIVI</td><td>vsra.vi</td><td>101001</td><td>011</td><td>vsra.vi vd, vs2, uimm, vm</td><td>vd[i] = vs2[i] &gt;&gt;&gt; uimm</td></tr><tr><td>OPMVV</td><td>vredsum.vs</td><td>000000</td><td>010</td><td>vredsum.vs vd, vs2, vs1, vm</td><td>vd[0] = sum(vs1[0], vs2[0], vs2[1], vs2[2]..., vs2[-1])</td></tr><tr><td>OPMVV</td><td>vredmax.vs</td><td>000111</td><td>010</td><td>vredmax.vs vd, vs2, vs1, vm</td><td>vd[0] = max(vs1[0], vs2[0], vs2[1], vs2[2]..., vs2[-1])</td></tr></table>

HINT: Modify the modules under /src/vsrc/components/two-issue and /src/vsrc/rvcpu/rvcpu_two-issue.v (based on task 2).

# Task3.2 - Assembly Code 2

Implement the softmax computation using RV32I/V and RVV instructions.

HINT: Finish the code in /src/asm/task3_2.asm.

NOTICE: If you'd like to add some other instructions for softmax computation, please check if it exists in InstructionSet[] of projects/lab4/tools/compile/Instruction.py. If it doesn't exist, please add it in this file manually.

# Simulation Environment

# File Tree

```txt
-- build
-- src
-- asm
-- csrc
-- vsrc
```

```powershell
--tools --Folder containing the python scripts   
1-- Makefile
```

# Make Command

Run the simulation and test. IMG is your assembly code name. ISSUE_NUM determines the issue number (1 for Task1.x, 2 for Task2.x and Task3.x). You can modify the parameter IMG and ISSUE_NUM in Makefile or in your commands.

```batch
make run IMG=task1_2 ISSUE_NUM=1  
make run IMG=task2_2 ISSUE_NUM=2
```

HINT: You can check your answer of Task x.1 by setting IMG = Taskx_1 and Task x.2 by setting IMG = Taskx_2.

There are 2 ways to clean the built files.

```txt
make clean # reserve the vcd file  
make clear # clean all files in build_test
```

You can run all the testbench at one time by answer.sh.

# Grading

The total score (100%) is the sum of code (100%).

# Code (100%)

- Task1 (30%)

- Task1.1 (15%)  
- Task1.2 (15%)

Task2 (30%)

- Task2.1 (20%)  
- Task2.2 (10%)

Task3 (40%)

- Task3.1 (20%)  
- Task3.2 (20%)

# Submission

Pack your lab4 folder and rename it with a new name StudentNumber_StudentName_Lab4.zip, and submit to Gradescope. The file structure should be like this.

```txt
12345678_张三_Lab4.zip
```

# Reference Link

1. M. Dukhan and A. Ablavatski, "Two-Pass Softmax Algorithm," 2020 IEEE International Parallel and Distributed Processing Symposium Workshops (IPDPSW), New Orleans, LA, USA, 2020, pp. 386-395, doi: 10.1109/IPDPSW50202.2020.00074  
2. RISC-V 手册一本开源指令集的指南  
3. The RISC-V Instruction Set Manual Volume I: Unprivileged ISA  
4. riscv-card  
5. RV32I, RV64I Instructions  
6. RISCV-V Online Doc  
7. RISCV-V Document PDF