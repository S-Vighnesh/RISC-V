# BUILDING A RISC-V CORE - MYTH

This repository contains all the information regarding the 5-day RISC-V based CPU Core Design MYTH (Microprocessor for You in Thirty Hours) Workshop, offered by for VLSI System Design (VSD) and Redwood EDA. In a short span of 5-days, the basic RISC-V ISA was studied & a simple RISC-V core with base instruction set was implemented. The RISC-V CPU Core has been designed with the help of Transaction Level Verilog(TL-Verilog) in addition with the Makerchip IDE Platform. Find below the accompanying details.

# Contents of The workshop

Check the individual day folders for the documentation, source codes and assignments of the respective days of the workshop.

This Course mainly covers Day 3 to 5. For details of Day1 and Day2 [click here](https://github.com/yagnavivek/PES_ASIC_CLASS/blob/main/README.md#course)

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

![compile_using_c_compiler](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/484cbdf8-07db-41b7-bc1b-6667825d580f)

_____compile using riscv compiler and view the output_____
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o 1_to_N.o sum_1_to_N.c
spike pk 1_to_N.o
```

![compile_using_riscv](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/adee1aab-d1da-44e3-8bad-babf43f869af)

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
![debug_using_spike](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/472dc533-af76-4b07-bb53-8fa8e2100785)

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
![ALP_used_O1](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/9c63218d-babc-45df-bb62-c894b27f13c5)

_____contents of main when used -Ofast optimizer_____
![ALP_Used_ofast](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/a89dff4b-30a7-46fe-bd7e-166ebe95f4e9)

## Integer number Representation (n-bit)
- Range of Unsigned numbers : [0, (2^n)-1 ]
* Range of signed numbes : Positive : [0 , 2^(n-1)-1]
                         Negative : [-1 to 2^(n-1)]

## 2. create a C program that shows the maximum and minimum values of 64bit unsigend and signed numbers

```
sign_unsign.c
```
![sign_snsign_compiled](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/99db1c71-f02f-472e-9185-4684a90551b8)

[Back to COURSE](https://github.com/yagnavivek/PES_ASIC_CLASS/tree/main#course)

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

<img width="1000" height="420" alt="image" src="https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/e69043fb-684e-42eb-9e21-fd51943c1ec1">

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

<img width="1000" height="600" src="https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/27d13974-1b70-4207-a2fb-05b232027323">

#### Data can be stored in register by 2 methods
1. Directly store in registers
2. Store into registers from memory

To store 64 bits of data from mem to reg, we use 8*8bit stores ie., m[0],m[1]......m[7]. 

- ___RISC-V uses Little Endian format to store the data ie., Least significant Byte is stored in m[0]___

## Simulate a C program using ABI function call (using registers) and execute 

The required program files are under day 2 folder

<img width="1150" height="150" src="https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/45c24570-4f59-4ddc-8a15-741bdd2d18c2">

![alp_onetonextern](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/c6e76147-0b45-4699-8f09-ea20e0364a47)

Here we can observe that at 5th line, inorder to comute the result ,its going to the "load"  function

### Further we will see how to run a C program on on RISC-V CPU

- Input : C Program loaded into memory to RISC-V CPU in Hex format

CPU processes the contents of the memory and provides with output using iverilog 

- Risc-V CPU : ```Picorv32.v```
- Testbench for verification : ```testbench.v```
- Tool : ```iverilog```
- script : ```rv32im.sh``` : has the commands to get the c-program, ALP, converts into hex format, loads into memory of riscv cpu, passes it iverilog and provides the output

![Screenshot from 2023-08-21 22-22-08](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/93475824/e2f64a4f-e83a-4c2c-a273-124f56b4da8b)


[Back to COURSE](https://github.com/yagnavivek/PES_ASIC_CLASS/tree/main#course)
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

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/8b29750b-b133-4ce8-8326-5ccfd7d87d5a)

#### C) Vectors

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/402438eb-b996-4782-945a-9df8a89030fa)

#### D) Mux (with and without vectors)

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/7795306d-10ba-48f4-a9e8-e490e5fcb102)

#### E) Simple Claculator

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/a09cf5b0-3202-42e0-8762-97cb3581bc61)

### Sequential Logic

- Sequential logic is sequenced by a clock signal
- A D-flip-flop transitions next state to current state on a rising clock edge
- Reset signal helps the circuit come to a known state

#### F) Fibonacci series

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/0e78f120-fb9f-4d24-ba6d-5b3e2485bd61)

#### G) Up-Counter

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/647f822d-008f-48ec-ae93-ade2da37db85)

#### H) Sequential Calculator 
  Input val1 is the previous output of calculator

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/9192605d-1221-46fd-bac3-c1f06352ac16)

#### I) A simple pipeline through Pythagorean example

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/923022a1-7bc4-4fd6-95e8-c0ea41405aba)

#### J) Pipeline Implementation example

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/2a58ccd8-e1ca-42bd-af79-2a277bc72265)


## Validity 
- Easier debug
- cleaner design
- Better error checking
- Automated clock gating

#### K) 2 cycle calculator with validity

![Screenshot from 2023-10-14 18-00-53](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/3e2e5fe1-2cbf-485d-a4e6-a6976e07e755)


#### L) Distance Calculator

![Screenshot from 2023-10-14 18-53-33](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/64701bdb-aa3e-4323-a2f4-2f2fa51a776c)

#### M) Calulator_memory

![Screenshot from 2023-10-14 18-58-48](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/bfc81566-6343-42e7-bfea-4ace15c2bfcf)


</details>

<details>
<summary>DAY 4 : Basic RISC-V CPU Micro Architecture</summary>
<br>

# RISC-V Implementation Blueprint

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/140a5803-8436-47a0-9e84-945284415482)

Link for the starter code [starter code](https://myth.makerchip.com/sandbox?code_url=https:%2F%2Fraw.githubusercontent.com%2Fstevehoover%2FRISC-V_MYTH_Workshop%2Fmaster%2Frisc-v_shell.tlv#) 

Codes for each step reported below can be found here [Day4 Codes](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/tree/main/DAY4)

## 1. Program Counter

- Program Counter is a register that contains the address of the next instruction to be executed. It is a pointer into the instruction memory, for the instruction that we are going to execute next. Since the memory is byte addressable and each instruction length is 32 bits, the Program Counter adder adds 4 bytes to the address to point to the next address.

- For the initial state, before fetching the first ever instruction, there is a presence of a reset signal that will reset the PC value to 0.

- For branch instructions, we will have immediate instructions, for which we have to add an offset value to the PC. So for branch instructions, NextPC = Incremented PC + Offset value.

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/86ebf67f-95ab-4a2b-8002-3eb1789966c6)

## 2. Instruction Fetch

- Here the instruction memory is added to the program. In the Instruction Fetch logic, the instructions are fetched from the instruction memory amd passed to the Decode logic for computation. 
- The instruction memory read address pointer is computed from the program counter and it outputs a 32 bit instruction. (instr[31:0]) .
- In our case, the Makerchip shell provides us an instantiation to the instruction memory, which contains a test program to compute the sum of numbers from 1 to 9.

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/3c9f61c7-124d-4dd7-b6d7-254da8ed16b3)


## 3.Instruction Decode

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/3464a32d-62c8-4a50-aef2-2c7e7000a179)

## 4. Instruction Decode with validity

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/06ea8486-6e91-4524-bff7-1b37cc0ff349)

## 5. Individual Instruction decode

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/e870f9db-de18-4052-b2d4-189dc0eddb92)

## 6. Register file read

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/a9c98216-a816-4fcf-974f-75fceeacbecd)

## 7. ALU

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/6a29d2b9-9354-48fa-8303-b09d90bf8b8c)

## 8. Register File Write

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/0c766dc9-3cd6-4387-bf9f-4167558ec240)

## 9. Branch Instructions

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/5e41c6a7-deac-411a-b439-05ff52215eb8)

## 10. Testbench to check functionality

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/857f16f1-e979-4e0f-a3f1-415392c6c85b)

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

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/cef37901-c4f9-4382-af6c-f778a1aaaa4a)

- There might be some invalid cycles (invalid operation when valid is on) being encountered in this proccess. SO we have to take care of them.

### 3-cycle RISC-V To take care of invalid signals

- Avoid writing into register file for invalid operations
- Avoid redirecting PC for nvalid instructions(branch)

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/1a2ec7de-4ab5-4e8c-bf8f-4024191dd641)

### Introduce 5 stage pipeline 

- 5 stage pipeline : PC, Decode, Reg Rd, ALU, Reg write

## Solutions to Pipeline Hazards

1. Register file bypass

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/9be47987-9e97-4eed-9c5e-8e65f0f158f2)

2. Correct the branch target path

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/2fffda72-1a4f-4b81-8643-dfa2a7fe8c7a)

3. Complete Instruction Decode

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/4f2127df-b28f-47e0-abac-37e7648f8ef2)

4. Complete ALU

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/dfb54415-4cda-434c-b175-11703f128ec8)

## Completing RISC-V CPU with final touch of Load/Store Instructions

1. Redirecting Loads

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/066cf305-0aa3-41fe-a937-f5ad39095d91)

2. Load Data from Memory to register file

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/f0fee7f9-9bf4-4a38-8386-275a52ebfecf)

3. Instantiate Data memory to CPU

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/03573672-9076-4c12-a8ac-c0fb43c2722d)

4. Add loads and stores to test the program

```
m4_asm(SW r0, r10, 100)
m4_asm(LW r15, r0, 100)
```
5. Lab for jump instructions

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/ec4d2fcf-143a-40e9-b991-912637bc4372)

</details>

# Code Comparision

The code required for the RISC-V Core written in TL-Verilog and System Verilog can be compared by selecting the "Show Verilog" on the makerchip platform under the "E" tab. Upon visualization, a significant code reduction can be seen in the comparision chart.

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/af4b9911-e56f-498a-9276-aceb17deb3c1)


# Final RISC-V core block Diagram

![image](https://github.com/yagnavivek/PES_YAGNAVIVEK_RISCV/assets/93475824/af260f0a-9f40-4d1d-9ab1-67dd7397a8fc)

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

