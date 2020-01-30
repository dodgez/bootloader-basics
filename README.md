# Overview
- What is a Bootloader?
- What is x86 and x86_64?
- How Many Bits?
- 16-bit Real Mode
- A20 Line
- Global Descriptor Table
- 32-bit Protected Mode
- Excep-Interrupts-tions (Interrupt Descriptor Table)
- Paging
- 64-bit Long Mode
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

![A Lot of Bits](https://raw.githubusercontent.com/sumitc91/data/master/askgif-blog/e00afba1-4221-4b3d-b9af-3a69ac35f84d_bits-bytes.jpg)

# 16-bit Real Mode
When *all* x86 processors (including any from the present day) start up, they are in what is called _16-bit Real Mode_.
This means that numbers can only be stored in 2 bytes, so 65536 is the maximum number.
This also means that less than 1 MB of RAM can be used!

This mode is the starting point for any bootloader so in order to boot a modern operating system (if you are using BIOS and not UEFI), the very first piece of code that gets run needs to be running in 16-bit.

One of the major benefits of this mode is the BIOS, standing for Basic Input/Output System and which is firmware built into the motherboard controlling the process of booting up and more, provides many useful utilities.
For example, by calling an interrupt, `int 0x10`, you can print characters out to the screen.
We will talk more about interrupts later, but you can think of them as a generic way to call a system-provided function.
So here, we are calling a predefined function that the BIOS provides to print a character out to the screen.
Later, we will use write the handling code for exception interrupts like dividing by zero.

Fun fact: MS-DOS was 16-bit real mode all the way up to the time of Windows 9x!

As mentioned above, every bootloader starts in this mode and its job is to get out of real mode and into 32-bit protected mode.
It does this in a series of steps.

1. Disable interrupts
2. Enable the A20 Line
3. Load the Global Descriptor Table
4. Set the PE (Protection Enable) bit in CR0 (Control Register 0)

At this point 16-bit code is still being run so we need to perform a far jump (which will be explained in GDT) to 32-bit code.

# A20 Line
# Global Descriptor Table
# Excep-Interrupts-tions (Interrupt Descriptor Table)
# Paging
# 64-bit Long Mode
# Build System
