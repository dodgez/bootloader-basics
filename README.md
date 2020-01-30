# Overview
- What is a Bootloader?
- What is x86 and x86_64?
- How Many Bits?
- 16-bit Real Mode
- Global Descriptor Table
- 32-bit Protected Mode
- Paging
- 64-bit Long Mode
- Excep-Interrupts-tions (Interrupt Descriptor Table)
- Build System

# What is a Bootloader?
A kernel is the core component of an operating system.
Its job is to manage memory, devices, and interfaces to those.
A bootloader is a piece of code which is responsible for loading the kernel from disk (or network), setting up the environment the kernel needs, give the kernel information about the system, and transferring control to the kernel.

The bootloader is one of the most challenging yet most rewarding pieces of an operating system because there are no protections and setting up the system is *extremely volatile* with little to *no helper routines*.

![Broken Hard Drive](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS7Pffjah6OpuHnde73ZCVVt1uRnDvzW6I28Tb7YR6GU5Oe4_wD7A&s)

# What is x86 and x86_64
The term _x86_ refers to a series of instruction set architectures developed by Intel.
Here, we use the phrase _instruction set architecture_ to mean set of instructions and their consequences.
Intel has remained very backwards compatible, so modern processors can still a lot of the same instructions as the Intel 8086 microprocessor.

The term _x86_64_ refers to the 64-bit version of the x86 instruction set (which is 32-bit).

![Intel 8086 Microprocessor](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRCca0gYFekjrqSLxhClefgyRtfQzW1upD53xNkrhjA7qLViJ1l&s)

# How Many Bits?
The term _bits_ and _bytes_ are used a lot in low level programming because the computer treats numbers as a series of bits, which are a placeholder for a 1 or a 0.
This means if a computer is 8-bit, it uses 8 placeholders for a 1 or 0 to represent numbers, so the highest possible number is 2^8 = 256.
A byte is just 8 bits.

Thus if we have a 16-bit, 32-bit, or 64-bit computer, numbers (and data) are stored in 2 bytes, 4 bytes, and 8 bytes respectively.
Also, the highest number for each type is 65536, ~4.9 billion, and a _very_ large number (2^64) for 64-bit.
For reference, 2^64 is approximately (2^3)^21 which has approximately 21 decimal places.
