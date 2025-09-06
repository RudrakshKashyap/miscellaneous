# [Program vs Process](https://www.youtube.com/watch?v=7ge7u5VUSbE&list=PL9vTTBa7QaQPdvEuMTqS9McY-ieaweU8M&index=1&pp=iAQB)

A **program** is a **static set of instructions**. It is a passive entity stored on a disk (e.g., an executable file like `notepad.exe` or `chrome.exe`).

A **process** is a **program in execution**. It is an active entity that has a life cycle—it is created, executes, and is terminated. A process requires resources like CPU time, memory, and I/O.

You can have one program file (e.g., `notepad.exe`), but you can run **multiple processes** of it (e.g., open three different Notepad windows to edit three different text files). Each open window is a separate process.

# [Process Control Block](https://youtu.be/LDhoD4IVElk?si=jb92_oWl68vjtNya&t=516)
The Process Control Block is a **data structure** inside the operating system kernel that stores all the crucial information the OS needs to manage a specific process.

# [Kernal mode vs User mode](https://www.youtube.com/watch?v=H4SDPLiUnv4&list=PL9vTTBa7QaQPdvEuMTqS9McY-ieaweU8M&index=3)

![](https://www.linux.it/~rubini/docs/ksys/ksys-figure1.png)

# [Interrupts](https://www.youtube.com/watch?v=1HHeyUVz43k)
```
Programmable Interrupt Controller (PIC) - a chip
The PIC forwards the highest-priority interrupt to the CPU's INTR (Interrupt Request) pin.
Interrupt Descriptor Table (IDT) is a special table in memory that is set up by the operating system during boot.
Interrupt Service Routine (ISR) is a small piece of kernel code designed to handle a specific event.
special return-from-interrupt instruction (e.g., IRET on x86)


+-------------------+     +------+     +---------------------+     +------+
| Hardware Device   |     | PIC  |     |       CPU           |     | OS   |
| (e.g., Keyboard)  |     |      |     |                     |     |      |
+-------------------+     +------+     +---------------------+     +------+
        |                    |                  |                    |
        | 1. Event occurs    |                  |                    |
        | (Key pressed)      |                  |                    |
        |------------------->|                  |                    |
        |                    |                  |                    |
        |                    | 2. PIC receives & prioritizes IRQ     |
        |                    | 3. PIC signals CPU on INTR pin        |
        |                    |-------------------------------------->|
        |                    |                  |                    |
        |                    |                  | 4. CPU finishes current |
        |                    |                  |    instruction         |
        |                    |                  | 5. Saves context (PC, regs)|
        |                    |                  | 6. Switches to Kernel Mode|
        |                    |                  | 7. Looks up ISR in IDT   |
        |                    |                  | 8. Jumps to ISR address  |
        |                    |                  |--------------------->|
        |                    |                  |                  | 9. ISR runs |
        |                    |                  |                  | (e.g., reads key)|
        |                    |                  |<--------------------|
        |                    |                  |                    |
        |                    |                  | 10. CPU executes IRET  |
        |                    |                  | 11. Restores context    |
        |                    |                  | 12. Resumes user code   |
        |                    |                  |                    |
```

## Types of Interrupts

1.  **Hardware Interrupts:** Caused by external hardware devices(keyboard, mouse, disk drive, **timer chip**). They are **asynchronous**, meaning they can happen at any time relative to the CPU's clock.
2.  **Software Interrupts (Traps):** Caused by the CPU itself executing a special instruction (e.g., `int 0x80` or `syscall`). This is how **system calls** are implemented. They are **synchronous**, meaning the program intentionally triggers them.
3.  **Exceptions:** Caused by the CPU detecting an error due to the executing program itself (e.g., division by zero, page fault, invalid memory access). The OS handles these, often by terminating the misbehaving program.


### What is a CPU Timer Interrupt?

It is a special type of **hardware interrupt** generated at fixed, regular intervals by a timer circuit (often part of the system's chipset, like the **Programmable interval timer PIT**). This interval is called the **tick rate** or **timer frequency**.

*   Without a timer interrupt, a process could run forever on the CPU, never giving other processes a chance. This is called **cooperative multitasking** and is unreliable (a misbehaving program can freeze the entire system).
*   The timer interrupt **preempts** the currently running process. It forces the CPU to switch into kernel mode, allowing the OS scheduler to decide:
*   *Should the current process keep running, or should we switch to a different one?*
*   This ensures every process gets a fair share of CPU time (a "time slice").

<br />

**Hardware Tick Rate (Common Value):** **100 Hz** (on many systems)
*   This means the timer interrupt fires **100 times per second**.
*   The time between interrupts is **10 milliseconds** (1000 ms / 100 = 10 ms).

**Typical Time Slice (or quantum):** **100 milliseconds**
*   This is a common default for interactive systems like Linux.

**The time slice (or quantum) assigned to a process is almost always significantly longer than the time between hardware timer interrupts (the tick period).**

```
CPU Executing Process A
        |
        |---> [Timer Interrupt Occurs] (e.g., every 10ms)
        |       |
        |       v
        |   CPU saves state of Process A, enters Kernel Mode
        |       |
        |       v
        |   Timer Interrupt Handler runs:
        |     - Updates time (jiffies++)
        |     - Accounts CPU time for Process A
        |     - Checks: Has time quantum expired? 
        |             : Is there a higher-priority process waiting to run?
        |           |
        |     No <--+--> Yes
        |       |         |
        |       |         v
        |       |     Set need_resched = TRUE
        |       |         |
        |       v         v
        |   Handler finishes
        |       |
        |       v
        |   CPU checks need_resched flag
        |       |
        |  FALSE |         TRUE
        |       |             |
        |       |             v
        |       |         Call schedule()
        |       |             |
        |       |             v
        |       |     Scheduler saves Process A's state to PCB,
        |       |     chooses Process B, loads Process B's state
        |       |             |
        |       v             v
        |   IRET instruction executed
        |       |             |
        |       |             |
        |       v             v
        |   Resume         Resume
        v   Process A      Process B
        |                   (Now running!)
        |
```

# [Problem -](https://www.youtube.com/watch?v=1HHeyUVz43k) To *context switch* CPU  has to save the state of current Process but even to do so it has to execute instructions, which will change the registers value...
The CPU needs to switch to a kernel stack, but it also needs to save the original user stack pointer (ESP). If it simply overwrote ESP first, the original value would be lost.


The short answer is: **The CPU uses a dedicated, hidden set of registers to execute the initial interrupt handling instructions without clobbering the main process's state.**

# In short Summary - Mordern process

Here's what happens during an interrupt that causes a privilege level change (e.g., from user to kernel mode):

**STEP -1. BEFORE RUNNING ANY PROCESS, THE KERNEL CREATES A SEPERATE STACKI(in kernel space ofcourse) FOR THIS
SEPECIFIC PROCESS AND STORES IT'S ADDRESS IN TSS.**


0. **Finish Current Instruction**
1. **Temporary ESP storage**: The CPU temporarily stores the current ESP value internally(in **hidden register**)
2. **Load new stack pointer**: CPU loads SS:ESP from the TSS (SS0:ESP0 for `ring 0`)
3. **Push old stack pointer**: CPU pushes the saved ESP value onto the new kernel stack(the one created for this specific process)
4. **Push other registers**: CPU continues pushing EFLAGS, CS, EIP, error code (if applicable)
5. **Update ESP**: ESP now points to the top of the kernel stack(the main kernel stack).
6. **Run the Interrupt Service routine code(in kernel)**: pushing general purporse registers and stuff.

During return -
`iret` **Atomic Return Magic**: single instruction for popping all user-mode `EIP`, `CS`, `EFLAGS`, `ESP`, `SS`.

### FAQ - Why we even need TSS, we can just store eip in temporary, and then change epi to jump to interrupt handler from there we can handle saving esp(like store esp) and other registers

The idea was to minimize the amount of "trusted" software code needed to handle a context switch.
Your Way (Software-Managed): The CPU would jump to a handler with a potentially corrupted user stack. The first few instructions of the interrupt handler (the software) would be responsible for checking the stack pointer, switching to a safe kernel stack, and then saving state. This creates a critical window of vulnerability where the kernel is running on a user-controlled stack.

The TSS mechanism actually simplifies the job of the OS kernel writer.

### Comparison: x86 vs. Your Proposed Method

| Feature | x86 (TSS Method) | Your Proposed Method (Common in RISC) |
| :--- | :--- | :--- |
| **Stack Switch** | **Hardware-enforced, automatic.** | Software-managed in the handler. |
| **Security** | **Higher.** No kernel code runs on user stack. | **Lower.** A few critical instructions must run on the user stack. |
| **Performance** | Potentially slower due to TSS access. | Potentially faster for simple interrupts. |
| **Flexibility** | Less flexible. Requires setting up a TSS. | More flexible. OS has full control over how state is saved. |
| **Critical Window** | **None.** Safe before handler runs. | **Exists.** Handler must immediately switch stacks. |
---
# More Details - 

## Method A: Dedicated Hardware Assistance (Common in RISC architectures like ARM, MIPS, RISC-V)

Many modern architectures provide alternative register banks for critical modes.

*   **Example: ARM's Exception Modes**
    *   ARM has distinct processor modes: `User`, `IRQ` (Interrupt), `FIQ` (Fast Interrupt), `Supervisor`, etc.
    *   Crucially, the register bank is *remapped* when the mode changes.
    *   Registers `R13` (SP - Stack Pointer) and `R14` (LR - Link Register) are *banked*. This means the `R13_irq` and `R14_irq` are physically different registers than `R13_user` and `R14_user`.
    *   When an IRQ interrupt occurs, the hardware automatically switches to the `IRQ` mode and uses `SP_irq` and `LR_irq`.
    *   The ISR, now executing in `IRQ` mode, can use `SP_irq` as its stack pointer and `LR_irq` to hold a return address **without ever touching the user process's `R13` and `R14` values.**
    *   The ISR's first instructions can then use these "safe" registers to save all the other general-purpose registers (R0-R12) onto the stack.

# Method B: Immediate Stack Switch (Common in x86 architecture)

The x86 architecture handles this differently but with the same goal.

1.  **Automatic Stack Switch:** As part of its hardware response, the x86 CPU often changes the stack pointer (`ESP`) to point to a kernel stack allocated for that **specific thread**. This is configured in the Task State Segment (TSS).
2.  **Hardware Saves Critical State:** It automatically pushes the essential state (the user `SS:ESP`, `EFLAGS`, `CS:EIP`) onto this **new** kernel stack.
3.  **ISR Uses New Stack:** The ISR code now begins execution. It's using the kernel stack(for that process), so any `PUSH` instructions it executes do not corrupt the user process's stack. The ISR prologue uses a series of `PUSH` or `MOV` instructions to save all the other general-purpose registers (`EAX`, `EBX`, `ECX`, etc.) onto this kernel stack.
	* Hardware Interrupt - **Save EVERYTHING**
	* Software Interrupt (`int 0x80`) - The The System Call Contract (The **ABI Application Binary Interface**) to know which user registers are volatile and which must be preserved.

Before `segment:offset` like `SS:ESP` used to describe a memory address but now a days esp is big enough that it can hold full address, so SS is obsolete.

## [Global Descriptor Table (GDT)](https://en.wikipedia.org/wiki/Global_Descriptor_Table)
The Global Descriptor Table (GDT) is a core part of Intel's x86 architecture that helps manage how memory is accessed and protected.
This is a table in memory that holds descriptors for segments of memory, including code, data, and system segments.
In 64-bit mode, segmentation is mostly disabled: all segment bases are treated as zero, and limits are ignored, creating a flat address space. However, GDT is still required to define system descriptors such as the **TSS**. 

## Task State Segment (TSS)
The TSS may reside anywhere in memory.

To use a TSS the following must be done by the operating system kernel:
* Create a TSS descriptor entry in the GDT
* Load the TR with the segment selector for that segment
* Add information to the TSS in memory as needed


### 1. The Original Design: Hardware Task Switching (Now Obsolete)

The x86 architecture was originally designed with a complex hardware-based task switching mechanism. The idea was:
*   The GDT would contain multiple TSS descriptors, each describing a TSS for a different task (process).
*   The `CALL` or `JMP` instruction could target a TSS descriptor.
*   The CPU would automatically perform a full context switch: saving all registers to the current TSS, loading a new TR, loading all registers from the new TSS, and changing privilege levels.
*   In this model, **the TR must be updated on every switch** because each process had its own TSS.

**This mechanism is slow and inflexible.** Modern operating systems (like Linux, Windows, macOS) **do not use this**. They handle context switching in software because it's much faster and offers more control.

### 2. The Modern Use: The TSS is for Stack Switching on Privilege Level Changes

Since hardware task switching isn't used, what is the TSS used for? Its critical, non-optional job in modern OSes is to **hold the kernel-mode stack pointers for when a privilege level change occurs.**

When an application (running in **ring 3, user mode**) makes a system call or triggers an interrupt, the CPU needs to switch to **ring 0 (kernel mode)**. For this switch to work, the CPU *must* have a known, valid stack to use for kernel mode. It cannot use the user-mode stack for security and stability reasons.

Where does the CPU get the new `SS:ESP` values for the kernel stack?
**It loads them from the TSS that the current core's TR points to.**

The TSS has fields like `ss0`, `esp0`, `ss1`, `esp1`, etc. On a privilege level change from ring 3 to ring 0, the CPU automatically loads the kernel stack pointer from `ss0` and `esp0` in the current TSS.

Each core has one **Task Register TR**, pointing to one TSS descriptor in the GDT. This TSS is typically a single, `static struct tss_struct` per CPU, allocated at boot.

### Summary

| | **Hardware Task Switching (Obsolete)** | **Software Task Switching (Modern OS)** |
| :--- | :--- | :--- |
| **# of TSSs** | Many (one per task/process) | **One per CPU core** |
| **TR Update** | **Updated on every task switch** | **Set once at boot/CPU init, never changed for process switches** |
| **TSS Purpose** | Store full hardware context of a task | Store **kernel stack pointers** (`esp0`, `ss0`) for privilege changes |
| **What Changes** | The entire TSS and the TR register | **Only the `esp0` field** inside the single, per-CPU TSS structure |

### User Stack vs Kernel Stack
* User Stack - The user process could be buggy or malicious. Its stack could be corrupted, too small, or even swapped out to disk.
* Kernel Stack - It's located in protected kernel memory, is always resident in RAM (never swapped out), and its integrity is paramount for system stability.


<br />
<br />
<br />
<br />
<br />
---


# todo https://www.youtube.com/watch?v=QatE61Ynwrw&pp=ygUYdXNlciBtb2RlIHZzIGtlcm5lbCBtb2Rl
https://kernelnewbies.org/Linux_Kernel_Newbies