
# Instruction Set Architecture Design

## 1. Introduction

- In this lecture, we are going to look at the principles and issues behind the design of instruction set architectures (ISAs).
- Then, we will explore the advantages and disadvantages of the two main ISA design philosophies: RISC and CISC.
- Finally, we will look in detail at one example ISA which we will use for the rest of the subject: the MIPS architecture.
- At the end of the journey, you will probably find it beneficial to go back over the material, to see how the MIPS architecture reflects the principles and issues of ISA design.

## 2. Designing an Instruction Set and CPU

- Our goal, in designing an ISA is to make a CPU which has enough functionality so that programmers can write programs for it.
- As we saw last week, we need instructions to do basic maths, data comparisons, deal with data of different sizes, and instructions to branch and jump (so we can implement decisions, loops and functions).
- There are an infinite number of ways we can build instruction sets to achieve the above, so we will also need to make some other design decisions along the way.
- However, any design decision which we make then becomes a constraint which limits all our other design decisions.
- In effect, any complete design is a collection of compromises which in total come as close as possible to achieving our original goals.
- So, we need to keep in mind what are our most important design goals, and which ones we can compromise on.

## 3. Basic ISA Issues

### 3.1 Registers

- We probably want to have some registers, so that we don't have to keep going out to slow memory, but how many registers do we want or need?
- More registers is probably better, but this will make implementing the CPU more complicated, i.e. we tradeoff ease of programming against ease of building the CPU.
	- and we need bits in each instruction to encore the register number: 8 registers = 3 bits etc.
- It also means we have to save out more registers when we switch between programs, and we do want to support a system where multiple programs can be running at the same time.
- On the other hand, too few registers means that we will have to go out to main memory more often.
- Another issue is how we deal with the special registers like the Program Counter (PC): do we want these registers visible and accessible to the programmer?
	- this would allow the programmer to change the PC's value by hand, which could be useful.
	- or do we provide special instructions to manipulate these in only limited ways?
- Most modern ISAs have 8 to 32 registers.

### 3.2 Bus Sizes

- What size data bus do we want? This effectively sets the natural word size of the ISA.
- And because the CPU will want to fetch in word units, this also influences the size of our instructions.
	- generally, each instruction will be 1 word at minimum, or a multiple of the word size.
- What size address bus do we want? This determines how much memory the CPU can address.
- But then we need the ability to express every address in some way.
- If the address size is too big, it can make the instruction size too big.
- For example, if the address bus is 64 bits, how will we encode the operation: load a word from address X into register R3?
	- The instruction will be at least 64 bits long to hold the address, plus bits to describe the load operation and the register which is the destination.

### 3.3 Operations

- How many operations do we want to have?
- If we have lots, this provides a rich set of operations that the programmer can perform, but it will make implementing them in silicon much more difficult.
- We will also need more bits in each instruction to encode the operation:
	- 5 bits => 32 operations, 6 bits => 64 operations etc.
- On the other hand, we might choose to have only a small set of simple instructions. This may force the programmer to have to combine 2 or more instructions to get something done, but the advantage is fewer bits required to encode an operation, and a simpler CPU design.

### 3.4 How Many Operands?

- With the number of instructions decided, how many operands will each operation require?
- If the CPU gets most of its data from registers, then we probably want to have some 3-operand instructions like
	- ADD R1, R2, R3, i.e. Add R2 and R3, and save the result into R1
so the instruction format needs to have bits set aside to identify each of the three registers.
- Other 3-operand instructions include instructions which compare and then divert the PC to a new instruction, e.g.
	- BGT R1, R2, 100, i.e. if R1 > R2, branch the CPU to instruction at current PC + 100
- Not all instructions have 3 operations. Examples of 2-operand instructions include:
	- LOAD R3, 4000, i.e. get the value from memory location 4000 and load it into register R3.
	- SAVE R4, 5000, i.e. write R4's value out to memory location 5000.
	- SET R6, 23, i.e. set R6 to the literal value 23 (not the value at location 23).
- And, of course, a CPU designer may think of 1-operand instructions, such as:
	- INCR R4, i.e. increment the value in R4. The Java equivalent is R4++.
- Later on, we will talk about the various addressing modes which a CPU designer might wish to use.

### 3.5 Literal Values

- Many instructions require literal values.
	- e.g. in Java when we write for (i=0; i<100; i++), there are two literal values: 0 and 100.
- Are we going to be able to find space in each instruction to put in literal values?
- If so, that will be great, but it will be wasted space if programs don't have many literal values.
- If we can't find space, then each time there is a literal, it is going to have to be stored in a register, or we are going to have to go out to memory to fetch the literal value.

### 3.6  Instruction Format

- Each different instruction has to be encoded differently: operation, operands, literal values, where to place the result, size of the data being operated on etc.
- What is the instruction format going to look like?
- Can we make each instruction the same size, or are some instructions going to be different sizes?
- Is there going to be a single format, which makes the decoding in silicon easy, or are we going to have several different instruction types, each with a different format?




