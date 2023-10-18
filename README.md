# BUILDING A RISC-V CORE - MYTH

This repository contains all the information regarding the 5-day RISC-V based CPU Core Design MYTH (Microprocessor for You in Thirty Hours) Workshop, offered by for VLSI System Design (VSD) and Redwood EDA. In a short span of 5-days, the basic RISC-V ISA was studied & a simple RISC-V core with base instruction set was implemented. The RISC-V CPU Core has been designed with the help of Transaction Level Verilog(TL-Verilog) in addition with the Makerchip IDE Platform. Find below the accompanying details.

# Contents of The workshop

Check the individual day folders for the documentation, source codes and assignments of the respective days of the workshop.

This Course mainly covers Day 3 to 5. For details of Day1 and Day2

<details>
<summary>DAY 1: Introduction to RISCV ISA and GNU Compiler Toolchain</summary>
<br>

## Introduction to Risc-v Basic Keywords
- **Instruction Set Architecture(ISA)**
  - An Instruction Set Architecture (ISA) refers to the set of instructions that a computer's central processing unit (CPU) can understand and execute. It defines the interface between software and hardware, specifying the operations that a CPU can perform, the data types it can manipulate, and the memory addressing modes it supports.

- **Risc-V ISA**
  - Risc-V ISA is an open-source ISA that has simpler and fixed length instructions that allows us to create custom processors for specific needs without being tied to proprietary architectures
 
- **Tools Used for the flow**
  - As we are aware of the flow, we will be using Risc-v ISA ALP and the RTL used will be picorv32a (We will be using rv64i during initial stages)

# Goal : Any High level Program that is written should be able to get executed in our CHIP

### List of well-known extensions present in Risc-V ISA

``` rv32i``` ``` rv64i``` ```rv32imc``` ```rv64imc``` ```rv32imafdc``` ```rv64imafdc``` ```rv32imcb``` ```rv64imcb``` ```rv32imc_sv32``` ```rv64gcv```

### Extensions and their Applications

- **I (Integer)** :The I set includes the base integer instruction set for RISC-V. It provides fundamental integer arithmetic and logical operations, data movement, and control flow instructions.
  - ADD, SUB, AND, OR, XOR, ADDI, SLTI, JAL, BEQ, LW

- **M (Multiply and Divide)** : The M set adds integer multiplication and division instructions to the base integer set. These instructions are particularly useful for arithmetic-heavy computations.
  - MUL, MULH, DIV, REM
  
- **A (Atomic)** : The A set introduces atomic memory access instructions. These instructions enable multiple operations on memory locations to be performed atomically, ensuring that other processors or threads cannot observe intermediate states.
  - LR (Load-Reserved), SC (Store-Conditional), AMO (Atomic Memory Operation)
  
- **F (Single-Precision Floating-Point)**: The F set adds single-precision floating-point instructions. These instructions enable arithmetic operations on 32-bit floating-point numbers.
  - FADD.S, FSUB.S, FMUL.S, FDIV.S, FCVT.W.S, FCVT.S.W

- **D (Double-Precision Floating-Point)** : The D set includes double-precision floating-point instructions. These instructions allow arithmetic operations on 64-bit floating-point numbers.
  - FADD.D, FSUB.D, FMUL.D, FDIV.D, FCVT.W.D, FCVT.D.W

- **C (Compressed)** : The C set introduces a compressed instruction format that reduces the size of code. Compressed instructions maintain the same functionality as their non-compressed counterparts but use shorter encodings.
  - C.ADDI4SPN, C.LWSP, C.ADDI, C.SW, C.JALR, C.BEQZ

- **G (Atomic and Lock-Free Operations)** : The G set, also known as the "GAS Set," is an alternative to the A set. It focuses on providing atomic and lock-free instructions to simplify hardware implementation.
  - LRV (Load-Reserved Variant), SCV (Store-Conditional Variant), AMO (Atomic Memory Operation Variants)

- **V (Vector)** :The V set adds vector instructions to the ISA, enabling Single Instruction, Multiple Data (SIMD) operations. These instructions allow efficient parallel processing of data elements in vectors.
  - VADD, VMUL, VFMADD, VLW, VSW

- **S (Supervisor)** : The S set, often used in privileged modes, includes instructions for managing and interacting with the supervisor-level operations of the system, such as handling exceptions and interrupts.
  - ECALL, EBREAK, SRET, MRET, WFI

- **B (Bit Manipulation)** : The B set introduces instructions for bit manipulation operations, allowing efficient manipulation of individual bits in registers and memory.
  - ANDI, ORI, XORI, SLLI, SRLI, SRAI

## 1. Create a simple C program That calculates sum from 1 to N -> sum_1_to_N.c

_____Compile it using C compiler_____
```
gcc sum_1_to_N.c -o 1_to_N.o
./1_to_N.o
```
-o allows you to name your output file

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/305639cd-3672-477e-a0a5-026735fa6e22)


_____compile using riscv compiler and view the output_____
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o 1_to_N.o sum_1_to_N.c
spike pk 1_to_N.o
```

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/db7508ad-c758-40bb-b6c9-1e783a613919)


- ```-O<number>``` : level of optimisation required
- ```-mabi``` : specifies the ABI (Application Binary Interface) to be used during code generation according to the requirements
- ```-march``` : specifies target architecture

_______We can check the different options available for all these fields using the commands_______ 
go to the directory where riscv64-unkonwn-elf is present
- -O1 : ``` riscv64-unkonwn-elf --help=optimizer```
- -mabi : ```riscv64-unknown-elf-gcc --target-help```
- -march : ```riscv64-unknown-elf-gcc --target-help```

_____To view the disassembled ALP code_____
```
riscv64-unknown-elf-objdump -d 1_to_N.o
```

_____To debug the ALP generated by the compiler_____
```
spike -d pk 1_to_N.o
```

- press ENTER : shows the first line and successive ENTER shows successive lines
- reg 0 a2 : checks content of register a2 0th core
- q : quit the debug process

##### Difference between the ALP commands when used different optimizers
- use the command ```riscv64-unknown-elf-objdump -d 1_to_N.o | less```
- use ``` /instance``` to search for an instance 
- press ENTER
- press ```n``` to search next occurance
- press ```N``` to search for previous occurance. 
- use ```esc :q``` to quit

_____Contents of main when used -O1 optimizer_____
![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/9bcf10f7-fe33-45f3-8867-4656119cb86a)


_____contents of main when used -Ofast optimizer_____
![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/0c05586b-08a3-452f-b242-6b97aa07acfb)


## Integer number Representation (n-bit)
- Range of Unsigned numbers : [0, (2^n)-1 ]
* Range of signed numbes : Positive : [0 , 2^(n-1)-1]
                         Negative : [-1 to 2^(n-1)]

## 2. create a C program that shows the maximum and minimum values of 64bit unsigend and signed numbers

```
sign_unsign.c
```
![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/b3360c42-659a-421a-8425-f8aabd68a7e1)




</details>
<details>
<summary>DAY 2 : Introduction to ABI and Basic Verification Flow </summary>
<br>

## BASICS :

Instructions that act on signed or unsigned integers are called Base Integer Instructions
There are 47 Base Integer Instructions present in RISC-V ISA

### Types of Instruction based on encoding format

1. **R-Type (Register-Type):**
   - These instructions operate on registers and have a fixed format for their operands.
   - Examples: ADD, SUB, AND, OR, XOR, SLL, SRL, SRA, SLT, SLTU

2. **I-Type (Immediate-Type):**
   - These instructions have an immediate operand and one register operand.
   - Examples: ADDI, SLTI, SLTIU, XORI, ORI, ANDI, SLLI, SRLI, SRAI, LB, LH, LW, LBU, LHU, JALR

3. **S-Type (Store-Type):**
   - These instructions are used for storing values from registers to memory.
   - Examples: SB, SH, SW

4. **B-Type (Branch-Type):**
   - These instructions perform conditional branching based on comparisons.
   - Examples: BEQ, BNE, BLT, BGE, BLTU, BGEU

5. **U-Type (Upper Immediate-Type):**
   - These instructions have a larger immediate field for encoding larger constants.
   - Examples: LUI, AUIPC

6. **J-Type (Jump-Type):**
   - These instructions are used for unconditional jumps and function calls.
   - Examples: JAL

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/be9b07f3-4a07-4f3c-bd23-495bc86ed0e4)


**[number]** represents number of bits occupied by that field

1. **Opcode [7] :** The opcode is a field within a machine language instruction that indicates the operation to be performed by the instruction. It defines the type of operation, such as arithmetic, logic, memory access, or control flow. Opcodes are used by the CPU to determine how to execute the instruction.

2. **rd (Destination Register) [5]:** The "rd" field represents the destination register in an assembly language instruction. It indicates the register where the result of the operation will be stored. After executing the instruction, the computed value will be placed in this register.

3. **rs1 (Source Register 1) [5]:** The "rs1" field represents the first source register in an assembly language instruction. It indicates the register that holds the value used in the operation. For instructions that involve two operands, "rs1" typically corresponds to the first operand.

4. **rs2 (Source Register 2) [5]:** The "rs2" field represents the second source register in an assembly language instruction. It indicates the register that holds the value used in the operation. For instructions that involve three operands, "rs2" typically corresponds to the second operand.

5. **func7 and func3 (Function Fields)[7] [3]:** These fields further refine the operation specified by the opcode. The "func7" field is used to distinguish different variations of instructions within the same opcode category. The "func3" field is used to specify a more specific operation within the opcode category. Together, these fields allow for a finer level of instruction differentiation.

6. **imm (Immediate Value):** The "imm" field represents an immediate value that is part of the instruction. Immediate values are constants that are embedded within the instruction itself. They can be used for various purposes, such as specifying offsets, constants, or small data values directly within the instruction.


#### ABI : Application Binary Interface

The instructions generated by compiler using a target ISA can be accessed by OS and User directly
- The parts of ISA accessible to User : User ISA
- The parts of ISA accessible to OS : system ISA
The access is done using Sysytem calls with the help of ABI

==> If we want to access hardware resources of processor, it has to be done via registers using ABI(names)

### ABI Names : 
- ABI names for registers serve as a standardized way to designate the purpose and usage of specific registers within a software ecosystem. These names play a critical role in maintaining compatibility, optimizing code generation, and facilitating communication between different software components.

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/43c5d23d-3f71-47a6-b957-bcdb8a2f35fe)


#### Data can be stored in register by 2 methods
1. Directly store in registers
2. Store into registers from memory

To store 64 bits of data from mem to reg, we use 8*8bit stores ie., m[0],m[1]......m[7]. 

- ___RISC-V uses Little Endian format to store the data ie., Least significant Byte is stored in m[0]___

## Simulate a C program using ABI function call (using registers) and execute 

The required program files are under day 2 folder

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/56e45ab1-5b8c-49ed-bf43-249b00acfb40)



Here we can observe that at 5th line, inorder to comute the result ,its going to the "load"  function

### Further we will see how to run a C program on on RISC-V CPU

- Input : C Program loaded into memory to RISC-V CPU in Hex format

CPU processes the contents of the memory and provides with output using iverilog 

- Risc-V CPU : ```Picorv32.v```
- Testbench for verification : ```testbench.v```
- Tool : ```iverilog```
- script : ```rv32im.sh``` : has the commands to get the c-program, ALP, converts into hex format, loads into memory of riscv cpu, passes it iverilog and provides the output



</details>


<details>
<summary>DAY 3 : Digital Logic With TL Verilog in Makerchip IDE</summary>
<br>

# Example 
Navigate to [makerchip](https://makerchip.com)

### Inverter
- Learn -> Examples -> Makerchip Default Template

#### A) Inverter in TLV using command

- under TLV Section type ```$out = ! $in1```
- Now compile ``` press E -> compile```

#### B) Xor gate using operators

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/d199c126-9816-4db8-ab5e-37789f7a3fd9)


#### C) Vectors

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/3a463788-447f-4538-b601-2aa135316f57)


#### D) Mux (with and without vectors)

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/b1ca531e-29d8-497b-afe2-a17ab964027b)


#### E) Simple Claculator

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/0046c0e8-771c-431c-9566-4f224c386e4d)


### Sequential Logic

- Sequential logic is sequenced by a clock signal
- A D-flip-flop transitions next state to current state on a rising clock edge
- Reset signal helps the circuit come to a known state

#### F) Fibonacci series

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/659e9c36-dacc-4e99-a935-04e0796fff13)


#### G) Up-Counter

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/dbcdf4cb-801d-4274-8694-ac5b6ffb7712)


#### H) Sequential Calculator 
  Input val1 is the previous output of calculator

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/a4f05719-a50d-486e-a4ea-5152e52da6fb)


#### I) A simple pipeline through Pythagorean example

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/dee7893f-3e06-4792-b984-26204ae030a9)


#### J) Pipeline Implementation example

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/f6528c79-d607-4eb2-9b1d-3996ebdaaba8)



## Validity 
- Easier debug
- cleaner design
- Better error checking
- Automated clock gating

#### K) 2 cycle calculator with validity

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/432b9610-3e58-4ef1-a7f4-08ca07cb54fa)



#### L) Distance Calculator

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/a0173020-4ae5-4b89-bbe1-d3e318b4dfff)


#### M) Calulator_memory

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/a054a0ea-b822-4308-b764-078de6da17fe)



</details>

<details>
<summary>DAY 4 : Basic RISC-V CPU Micro Architecture</summary>
<br>

# RISC-V Implementation Blueprint

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/396a4ef1-bcf6-4dc7-9604-1c7971d50418)


Link for the starter code [starter code](https://myth.makerchip.com/sandbox?code_url=https:%2F%2Fraw.githubusercontent.com%2Fstevehoover%2FRISC-V_MYTH_Workshop%2Fmaster%2Frisc-v_shell.tlv#) 


## 1. Program Counter

- Program Counter is a register that contains the address of the next instruction to be executed. It is a pointer into the instruction memory, for the instruction that we are going to execute next. Since the memory is byte addressable and each instruction length is 32 bits, the Program Counter adder adds 4 bytes to the address to point to the next address.

- For the initial state, before fetching the first ever instruction, there is a presence of a reset signal that will reset the PC value to 0.

- For branch instructions, we will have immediate instructions, for which we have to add an offset value to the PC. So for branch instructions, NextPC = Incremented PC + Offset value.

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/9c6ef09c-8061-406f-9757-fb82e684e698)


## 2. Instruction Fetch

- Here the instruction memory is added to the program. In the Instruction Fetch logic, the instructions are fetched from the instruction memory amd passed to the Decode logic for computation. 
- The instruction memory read address pointer is computed from the program counter and it outputs a 32 bit instruction. (instr[31:0]) .
- In our case, the Makerchip shell provides us an instantiation to the instruction memory, which contains a test program to compute the sum of numbers from 1 to 9.

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/9bb97751-ee83-41e6-a1f7-9974d24c3070)



## 3.Instruction Decode

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/9aa2c24b-4b09-47d3-acaa-80b1149f6719)


## 4. Instruction Decode with validity

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/73c57c8c-9c51-4129-bd9d-712f9f5a6ac8)


## 5. Individual Instruction decode

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/8ee5983d-a03c-41ea-ab41-07b73854e988)


## 6. Register file read

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/ab93aeab-41cf-4488-8ed4-ecd1d07c3f36)


## 7. ALU

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/ceffb1eb-788b-423c-b4b8-4d6557008e66)


## 8. Register File Write

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/4fede29d-ae90-4f70-939b-65b4719d0887)


## 9. Branch Instructions

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/3c11a7fe-a90f-482c-8665-b6b8811de68e)


## 10. Testbench to check functionality

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/2019d27e-f993-45fa-b0d2-a99100ea938f)


</details>

<details>
<summary>DAY 5 : Complete Pipelined RISC-V Micro Architecture </summary>
<br>

- Pipelining helps improve the operating frequency by breaking down the micro-arch ito substages that consume lesser time.But this process intriduces some hazards and dependencies such as
  - Data Hazards
  - Structural Hazards
  - Control Hazards
  - Name Dependence
  - Anti Dependence
  - Output Dependence

 ## 3-cycle RISC-V

 - A simple pipeline approach where we divide the arch into 3 stages ie., PC, Decode to ALU, Reg write.
 - This requires a valid signal that is generated every 3 clock cylces

### Generation of 3 cycle valid signal

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/98f5d27a-8e39-467c-a54f-5e3bf9acd941)

- There might be some invalid cycles (invalid operation when valid is on) being encountered in this proccess. SO we have to take care of them.

### 3-cycle RISC-V To take care of invalid signals

- Avoid writing into register file for invalid operations
- Avoid redirecting PC for nvalid instructions(branch)

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/9fafb6ff-355f-41a9-a71c-b64d2a73a9eb)


### Introduce 5 stage pipeline 

- 5 stage pipeline : PC, Decode, Reg Rd, ALU, Reg write

## Solutions to Pipeline Hazards

1. Register file bypass

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/6e2878b0-7d24-43ce-9626-eeeba39871c9)


2. Correct the branch target path

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/f0e0649b-7b97-4344-8bfe-08f1d537d876)


3. Complete Instruction Decode

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/7750d359-f072-4252-bafd-1f011f295977)


4. Complete ALU

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/afa5c168-2e19-424a-9ff9-3b69b522fc6e)


## Completing RISC-V CPU with final touch of Load/Store Instructions

1. Redirecting Loads

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/0113b1d1-59b4-486a-a16b-9b17963ec307)


2. Load Data from Memory to register file

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/9ace0132-6cb3-4b8d-b9c4-3efc19ca4d38)


3. Instantiate Data memory to CPU

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/92b515f7-723a-43fd-b9e9-efe03fbef0df)


4. Add loads and stores to test the program

```
m4_asm(SW r0, r10, 100)
m4_asm(LW r15, r0, 100)
```
5. Lab for jump instructions

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/f9e03d46-ac2f-42cb-9f7b-b643185ee223)


</details>

# Code Comparision

The code required for the RISC-V Core written in TL-Verilog and System Verilog can be compared by selecting the "Show Verilog" on the makerchip platform under the "E" tab. Upon visualization, a significant code reduction can be seen in the comparision chart.

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/0b7da2c9-5cfe-48d1-876a-6dc93c0ba14b)



# Final RISC-V core block Diagram

![image](https://github.com/S-Vighnesh/RISC-V/assets/137196908/91195cad-3f51-47e0-b6ea-9c9d24432f4c)


# References

You can follow the below mentioned sites for more information regarding the particular topics.

- **RISC-V:** https://riscv.org/
- **Makerchip Platform:** https://makerchip.com/
- **TL-Verilog:** https://www.redwoodeda.com/tl-verilog  or http://tl-x.org/
- **Redwood EDA:** https://www.redwoodeda.com/
- **VLSI System Design:** https://www.vlsisystemdesign.com/

# Acknowlegedgements

- [Dr Madhura Purnaprajna](https://in.linkedin.com/in/madhurapurnaprajna) Professor, PES University
- [Prof Mahesh Awati](https://in.linkedin.com/in/mahesh-awati-4423538b) Associate professor, PES University
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Steve Hoover](https://github.com/stevehoover), Founder, Redwood EDA

# Contact Information

- Kunal Ghosh, Director, VSD Corp. Pvt. Ltd. (Email: kunalghosh@gmail.com)
- Steve Hoover, Founder, Redwood EDA (Email: steve.hoover@redwoodeda.com)
- Yagna Vivek B ,Student, PES University (Email : pesug20ec109@pesu.pes.edu)

