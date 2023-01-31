# TOC
- [TOC](#toc)
- [Introduction](#introduction)
- [ISA Requirements](#isa-requirements)
  - [Some Things to Think About](#some-things-to-think-about)
  - [Architecture Limitations and Requirements](#architecture-limitations-and-requirements)
  - [Some Things to Think About](#some-things-to-think-about-1)
  - [Top-Level Interface](#top-level-interface)
  - [What must the processor do?](#what-must-the-processor-do)
    - [Program 1 (forward error correction block coder/transmitter)](#program-1-forward-error-correction-block-codertransmitter)
    - [Program 2 (forward error correction block decoder/receiver)](#program-2-forward-error-correction-block-decoderreceiver)
    - [Program 3 (pattern search)](#program-3-pattern-search)
  - [What to Submit?](#what-to-submit)
- [Milestone 1 — The ISA](#milestone-1--the-isa)
  - [Milestone 1 Objectives](#milestone-1-objectives)
  - [Milestone 1 Components](#milestone-1-components)
    - [What to Submit?](#what-to-submit-1)

# Introduction
Your task this quarter is to design a custom processor that supports specific Forward Error 
Correction (FEC) tasks. FEC is commonly used in radio communications with lossy links where 
retransmission is expensive, challenging, or impractical (as well as other domains). As an 
interesting and motivating example, consider one of the longer-lived computational systems 
humanity has ever built, the Voyager Probe, which passed out of the heliosphere last year but is 
still sending us data! As an example closer to home, satellite radio streams allow for 4-5s of 
signal loss (e.g. driving under a highway underpass) without any interruption in the audio 
stream. Sirius receivers use a custom chip, historically the STA210/240, which is in many ways 
simply an advanced form of what you are designing and implementing in this course.

# ISA Requirements
Your instruction set architecture shall feature fixed-length instructions (machine code) 9 bits 
wide.

*Given the tight limit on instruction bits, you need to consider the target programs and their needs carefully.
The best design will come from an iterative process of designing an ISA, then coding the programs, redesigning the 
ISA, etc.*

Your ISA specification should describe:
- What operations it supports and what their respective opcodes are.
  - For ideas, see the MIPS, ARM, RISC-V, and/or SPARC instruction lists  
- How many instruction formats it supports and what they are
  - In detail! How many bits for each field, where they are found in the instruction.
  - Your instruction format description should be detailed enough that someone 
other than you could write an assembler (a program that creates machine code 
from assembly code) for it. (Again, refer to ARM or MIPS.)
- Number of registers, and how many general-purpose or specialized.
- All internal data paths and storage will be 8 bits wide.
- Addressing modes supported
  - This applies to both memory instructions and branch instructions.
  - How are addresses constructed or calculated? Lookup tables? Sign extension? 
Direct addressing? Indirect? Immediates? 

*The more time and care you put into your specification, the easier the rest of the project will be. This is the design 
element, and it harder than it seems (you have a lot of options!).*

## Some Things to Think About
For instructions to fit in a 9-bit field, the memory demands of these programs will have to be 
small.  For example, you will have to be clever to support a conventional main memory of 256 
bytes (8-bit address pointer). You should consider how much data space you will need before 
you finalize your instruction format. Your instructions are stored in a separate memory, so that 
your data addresses need be only big enough to hold data. Your data memory is byte-wide, i.e., 
loads and stores read and write exactly 8 bits (one byte). Your instruction memory is 9 bits 
wide, to hold your 9-bit machine code. 

You will write and run three programs on your ISA. You may assume that each program starts at 
address 0, and that you will reload the instruction memory specifically for each program. The 
specification of your branch instructions may depend on where your programs reside in 
memory, so you should make sure they still work if the starting address changes a little (e.g., if 
you have to rewrite one of the programs and it causes the others to also shift). This approach 
will allow you to put all three programs in the same instruction memory later on in the quarter.

## Architecture Limitations and Requirements
We shall impose the following constraints on your design, which will make the design a bit 
simpler:

1. Your core should have separate instruction memory and data memory.
2. You should assume single-ported data memory (a maximum of one read or one write 
per instruction, not both — Your data memory will have only one address pointer input 
port, for both input and output).
3. Your instruction memory should not exceed 2^10 entries; it must not exceed 2^11 
entries. If you need the larger number of instruction entries, your writeup must explain 
how these extra entries improve some other performance element.
4. Your data memory should not exceed 2^8 entries; it must not exceed 2^9 entries. If you 
need the larger number of data entries, your writeup must explain how these extra 
entries improve some other performance element.
5. You should also assume a register file (or whatever internal storage you support) that 
can write to only one register per instruction.
   - The sole exception to this rule is that you may have a multibit ALU condition/flag 
register (e.g., carry out, or shift out, sign result, zero bit, etc., like ARM's Z, N, C, 
and V status bits) that can be written at the same time as an 8-bit data register, if 
you want.
    - You may read up to two data registers per cycle.
   - Your register file will have no more than two output ports and one input port.
   - You may use separate pointers for reads and writes, if you wish.
    - Please restrict register file size to no more than 16 registers.
1. Manual loop unrolling of your code is not allowed – use at least some branch or jump 
instructions.
1. Your ALU instructions will be a subset of those in ARMsim, or of comparable complexity.
2. You may use lookup tables / decoders, but these are limited to 32 elements each (i.e., 
pointer width up to 5 bits).
    - You may not, for example, build a big 512-element, 32-bit LUT to map your 9-bit 
machine codes into ARM- or MIPS-like wider microcode. (It was amusing the first 
time a team tried it, but it got old.)

## Some Things to Think About
In addition to these constraints, the following suggestions will either improve your performance 
or greatly simplify your design effort:
1. In optimizing for performance, distinguish between what must be done in series vs. 
what can be done in parallel.
    - E.g. An instruction that does an add and a subtract (but neither depends on the 
output of the other) takes no longer than a simple add instruction.
    - Similarly, a branch instruction where the branch condition or target depends on 
a memory operation will make things more difficult later on.
2. Your primary goal is to execute the assigned programs accurately. Secondary goals are:
    -  Minimize clock cycle count.
    - Minimize cycle time (short critical paths).
    - Simplify your processor hardware design.

Generic, general-purpose ISAs (that is, those that will execute other programs just as efficiently 
as those shown here) will be seriously frowned upon. We really want you to optimize a creative 
special purpose design for these programs only.

## Top-Level Interface
Your microprocessor needs only three one-bit I/O ports: clock input and start input from the 
testbench and done output back to the testbench.

We will use the start and done signals to drive your processor. During final testing, the 
sequence will be as follows:
1. The testbench will set the start bit high.
    - Your processor must not write to data memory while the start bit is asserted.
2. The testbench will load operands into specified location in the data memory.
3. The testbench will lower the start bit.
    - This should cause your processor to begin executing the first program.
4. When your program has run and your device has stored the result into the specified 
location(s) in data memory, your device should bring the done flag high.
5. The testbench will respond by reading and verifying your results.
6. You may then set up your second set of instruction 9-bit words.
7. The testbench will assert the start bit
    - Your processor should deassert the done flag in response
8. The testbench will load the next set of operands into the specified locations in data 
memory while the start bit is high.
9. The testbench will lower the start bit.
    - Your device should start running the second program.
10. When the second program completes, your processor should assert done.
11. The testbench will read and verify your results from the second program, then issue the 
final start command while loading the third set of operands into data memory. Your 
done flag at the end of this program will terminate simulation after the testbench reads 
and verifies your results. 

If you cannot get all three programs to run, separate testbenches for individual programs will 
also be provided, with correspondingly lower course grades awarded.

## What must the processor do?
Your processor must be able execute the following three programs.
### Program 1 (forward error correction block coder/transmitter)
Given a series of fifteen 11-bit message blocks in `data mem[0:29]`, generate the corresponding 
16-bit encoded versions and store these in `data mem[30:59]`.
Input and output formats are as follows: 
```
input MSW =    0   0   0   0   0 b11 b10 b09  
LSW =   b8  b7  b6  b5  b4  b3  b2  b1, where bx denotes a data bit

output MSW =  b11 b10  b9  b8  b7  b6  b5  p8 
LSW =   b4  b3  b2  p4  b1  p2  p1  p0, where px denotes a parity bit

Example, to clarify “endianness”:  binary data value = 101_0101_0101
mem[1] = 00000101  -- 5 bits zero pad followed by b11:b9 = 00000_101
mem[0] = 01010101  -- lower 8 data bits b8:b1
You would generate and store:
mem[31] = 10101010 -- b11:b5, p8                 = 1010101_0 
mem[30] = 01011010 --  b4:b2, p4, b1, p2:p1, p0 = 010_1_1_01_0 
p8 = ^(b11:b5) = 0; 
p4 = ^(b11:b8,b4,b3,b2) = 1; 
p2 = ^(b11,b10,b7,b6,b4,b3,b1) = 0;  
p1 = ^(b11,b9,b7,b5,b4,b2,b1) = 1;        
p0 = ^(b11:1,p8,p4,p2,p1) = 0;
```

### Program 2 (forward error correction block decoder/receiver)

Given a series of 15 two-byte encoded data values – possibly corrupted – in `data mem[30:59]`, recover the original message and write into `data mem[0:29]`.

This is just (sort of..) the inverse problem. However, in Program 1 there was only 1 unique 
output for each unique input. Now there are several (how many?) possible inputs for each of 
the 2**11 possible outputs.

This coding scheme can correct any single-bit error and detect any two-bit error.

The testbench will generate parity bits for various random data sequences, occasionally flip 
one, occasionally two, of the parity or data bits, and then ask you to figure out the original 
message.

### Program 3 (pattern search)
Given a continuous message string in `data mem[0:31]` and a 5-bit pattern in bits `[7:3]` of `data 
mem[32]`:
- Enter the total number of occurrences of the given 5-bit pattern in any byte into `data 
mem[33]`. Do not cross byte boundaries for this count.
- Write the number of bytes within which the pattern occurs into `data mem[34]`.
- Write the total number of times it occurs anywhere in the string into `data mem[35]`. For 
this total count, consider the 32 bytes to comprise one continuous 256-bit message, 
such that the 5-bit pattern could span adjacent portions of two consecutive bytes.

```
Note the order ("Endian-ness") of the sequence:
data mem[0] = first byte in message string;
bit [7] is the first bit in the entire string;
data mem[1] = second byte;
bit [7] of [1] comes right after bit [0] of [0]
Example:
data_mem[0:31] = 8'h0,8'h0..  Memory to search is all zeros          
data_mem[32] = 8'b00000_000 The pattern to search for is ‘00000’
data_mem[33] = 4*32 = 128   because the string can show up in any
of 4 different locations in each byte
data_mem[34] = 32           because every byte contains at least
one copy of the pattern
data_mem[35] = 252          because pattern can start at bit 0, 1, 2, .., 251
```

  *It cannot start at bit 252, because one bit of the pattern would fall outside the string; likewise, 253, 254, and 255 are "nonstarters," as well*

## What to Submit?
You will turn in milestone reports and (eventually) all your code.

Reports will address questions for each milestone. In describing your architecture, keep in mind 
that the person grading it has much less experience with your ISA than you do. It is your 
responsibility to make everything clear. One objective of this course is to help you improve your 
technical writing and reporting skills, which will benefit you richly in your career. 

For each milestone, there will be a set of requirements and questions that direct the format of 
the writeup and make it easier to grade, but strive to create a report you can be proud of.

# Milestone 1 — The ISA
For the first milestone, you will design the instruction set architecture (ISA) for your processor. 
A quick reminder that an ISA is more than just an instruction set. It describes a fair bit about 
how the machine will work [at least from the programmer’s perspective]. It specifies how many 
registers are available, how memory operates, how addressing works, etc. Your ISA design will 
dictate your implementation — plan ahead!

## Milestone 1 Objectives
For this milestone, you will design the instruction set and instruction formats for your 
processor. You will then write code for the three programs to run on your instruction set.

## Milestone 1 Components
0. Team.
    - List the names of all members of your team, but only one copy of the report 
should be submitted.
1. Introduction.
    - This should include the name of your architecture (have fun with this), overall 
philosophy, specific goals strived for and achieved.
    - Can you classify your machine in any of the classical ways (e.g., stack machine, 
accumulator, register, load-store)? If so, which? If not, devise a name for your 
class of machine.
2. Architectural Overview. This must be in picture form.
    - What are the major building blocks you expect your processor to be made up of?
NOTE: This is not your final processor design, rather an early rough draft of the 
major elements. Missing details and imprecision are okay at this stage, but you 
should continue to refine this picture as your design evolves. You will submit an 
updated diagram with every milestone.
3. Machine Specification
    - Instruction formats.  
        - List all formats and an example of each. (ARM has R, I, and B type 
instructions, for example.)
    - Operations.  
        - List all instructions supported and their opcodes/formats.
    - Internal operands.
        - How many registers are supported?
        - Is there anything special about any of the registers, or all of them general 
purpose?
    - Control flow (branches).
        - What types of branches are supported?
        - How are the target addresses calculated?
        - What is the maximum branch distance supported?
    - Addressing modes.
      - What memory addressing modes are supported, e.g. direct, indirect?
      - How are addresses calculated?
      - Give examples.
4. Programmer’s Model [Lite]
   - How should a programmer think about how your machine operates?
   - Give an example of an “assembly language” instruction in your machine, then 
translate it into machine code.
5. Program Implementations

    - For each program, give assembly instructions that will implement the program correctly. 
Make sure your assembly format is either very obvious or well described, and that the code is 
(very) well commented. If you also want to include machine code, the effort will not be wasted, 
since you will need it later. We shall not correct/grade the machine code. State any assumptions 
you make.   
        - Program 1
        - Program 2
        - Program 3
### What to Submit?
You will submit a written report that contains all of the required components of Milestone 1. It 
is your responsibility to make this report clear and well-organized.

Your report should be a single document, in PDF form.
- Exception: You may include your program implementations as separate “source code” files 
if you wish.
