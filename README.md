# Overview
- What is a Bootloader?
- What is x86 and x86_64?
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
