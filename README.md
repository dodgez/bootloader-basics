# Overview
- What is a Bootloader?
- What is x86 and x86_64?
- How Many Bits?
- 16-bit Real Mode
- Global Descriptor Table
- A20 Line
- 32-bit Protected Mode
- Paging
- Excep-Interrupts-tions (Interrupt Descriptor Table)
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
Intel has remained very backwards compatible, so modern processors still have a lot of the same instructions as the Intel 8086 microprocessor.

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
In this example, we are calling a predefined function that the BIOS provides to print a character out to the screen.
Later, we will write the handling code for exception interrupts like dividing by zero.

Fun fact: MS-DOS was 16-bit real mode all the way up to the time of Windows 9x!

As mentioned above, every bootloader starts in this mode and its job is to get out of real mode and into 32-bit protected mode.
It does this in a series of steps.

1. Disable interrupts
2. Enable the A20 Line (for more memory)
3. Load the Global Descriptor Table (for protecting memory)
4. Set the PE (Protection Enable) bit in CR0 (Control Register 0)

At boot, 16-bit code is still being run so we need to perform a far jump (which will be explained in GDT) to 32-bit code.

# Global Descriptor Table
The Global Descriptor Table is a way of segmenting memory into chunks that have different attributes like read-only, write-only, priviledge level, size, etc.
Anymore the GDT is only used to describe the sections of memory for the kernel and the user and protects them using various flags so that user processes cannot overwrite kernel code or data.
Paging is used for further restriction of access and manipulation of memory and we will come back to this.

_One important note is 64-bit long mode deprecates a lot of GDT functionality and pretty much only uses it for determining code running in kernel or user space (has limited or full access to memory and such)._

A GDT is just a section of memory formatted as shown below, but it is used by the _Memory Management Unit_.
The purpose of the MMU is to take something called a virtual address, just a memory address, and maps it to a physical location in RAM.
This translation is important for a kernel to manage memory between applications without applications having to be writting based in different locations.
E.g. a program may expect important objects at a certain address and if the kernel decided to locate this program in different locations, the MMU makes it seem like these objects are at the expected address even if it is not the same location at all in physical memory.

![GDT Entry](https://wiki.osdev.org/images/f/f3/GDT_Entry.png)

A reference for this table is https://wiki.osdev.org/Global_descriptor_table.

Any read or write of memory is going to go through this table using things called segment registers.
- CS - Code Segment (offset of the entry in the GDT that points to the segment of memory containing code)
- DS - Data Segment (same idea as above but for data)
- SS - Stack Segment (usually set to the same as data)
- ES - Extra Segment (usually set to the same as data)
- FS, GS - General Purpose Segments (usually set to the same as data)

Remember, when we wanted to go from 16-bit Real Mode to 32-bit Protected Mode we needed to load a GDT.
But in order to actually execute any 32-bit code we need to setup the GDT structure in memory but also load the correct values in the segment registers.
It is straight forward to do this for DS, ES, FS, GS, SS, but in order to put the right value into CS, we need to do a _far jump_ because CS cannot be changed without doing a far jump.
A _far jump_ is just where you specify the location in memory you will start executing code and the code segment as well because potentially we may jump between code segments like from user to kernel space.

# A20 Line
The A20 Line is the 21st bit of any memory address.
When the Intel 286 was created, it allowed up to 16 MB of memory instead of the 1 MB of the 8086, but the 8086 had an odd feature.
The 8086 had address lines A0 through A19 that would go up to 1 MB but the segmented memory model allowed for above 1 MB, any address above 1 MB would wrap around to zero.

For the i286, they created address line A21, which is disabled by default to enable backwards compatibility with the 8086, but by enabling it, more than 1 MB could be addressed.

Fun fact: the keyboard controller had a spare pin with which they routed A20 through so the keyboard controller needed to be used to activate it.

![A20 Line](https://i.stack.imgur.com/OcwGJ.png)

# Paging
Paging is the modern replacement of Segmentation.
One of the main benefits of using paging versus segmentation is a particular problem of fragmentation.
Because the GDT described regions of various lengths, situations like this picture happen.
![Segment Fragmentation](https://os.phil-opp.com/paging-introduction/segmentation-fragmentation.svg)

Paging fixes this issue by using small fixed-size pages of data to get the following.
![Paging Memory Layout](https://os.phil-opp.com/paging-introduction/paging-fragmentation.svg)

We still have another type of fragmentation brought on by the fixed-size property.
This fragmentation isn't nearly as "bad" but it occurs when a program only needs to take a fraction of a page or a non-integer multiple of pages and so the rest of the page data is unused.
However, the amount of wasted space is not as bad compared to the performance issues of relocating chunks to fix segment fragmentation.

The way paging is setup is through a page table.
A page table is a list of addresses (one for each page) and the associated flags like permissions that go with it.

![Page Table Entry](https://wiki.osdev.org/images/9/9b/Page_table.png)

However, to describe all of the pages for a modern computer's amount of RAM would be wasteful.
For example, if only the first and last pages were needed, the table would still need all of the entries inbetween.

This is where multiple level page tables come in where instead of just one table pointing to each page, we may have a table which lists various tables that each list direct pages.
This allows the operating system to use less memory to describe the page system by not allocating memory for page entries describing things we don't need.

![Page Directory Entry](https://wiki.osdev.org/images/9/94/Page_dir.png)

Though this comes with a downside: if a program tries to access an address whose page isn't present, a _Page Fault_ occurs.
This will be important to handle because it happens all of the time as an operating system often uses more virtual memory than it has physically.

Paging may seem a little daunting because it is quite an advanced concept.
Operating systems need to constantly keep track and update these tables as programs request and free memory and this task is non-trivial.

The important part is Paging is _required_ in order to enter 64-bit long mode.
One could theoretically use 32-bit protected mode with just segmentation (though it would be awfully hard and run into issues discussed above), but in 64-bit long mode, the GDT purely describes kernel and user space segments.

# Excep-Interrupts-tions (Interrupt Descriptor Table)
Interrupts are signals to the CPU to halt what it is doing and deal with something else.
There are multiple types of interrupts:
- Exceptions - things like dividing by zero or a page being accessed isn't mapped into memory
- Hardware Interrupts or Interrupt Requests - things like keyboard input, timers, and devices needing attention
- Software Interrupts - kernel code or software code triggers an interrupt (like user code calling kernel code)

The main point of interrupts is that the CPU stops _everything_ in order to handle whatever is triggering the interrupt.
Funny enough, you can even have interrupts triggering while handling an interrupt! (Hence the title)

Interrupts are setup by creating a table pointing interrupt numbers and types to specific code that should run.
The hardware in charge of interrupts is the _Programmable Interrupt Controller_.
There are actually two PICs, a master and a slave, and depending on which sends the interrupt (specific interrupts go through one or the other), we need to send an acknowledgement that we got the interrupt and the PIC should continue sending interrupts.

An overview of the table structure is available at _osdev_ https://wiki.osdev.org/IDT#IDT_in_IA-32e_Mode_.2864-bit_IDT.29.

Interrupts are essential to how an operating system works and are necessary for many operations and since we want 64-bit long mode, which requires paging which may cause an exception, we need to set one up.

# 64-bit Long Mode
Congratulations, now wasn't that easy?
Here we are using the processor (almost) to its fullest and have access to a _lot_ of RAM (if installed) and large values for numbers.

# Build System
