# Instruction set architecture (ISA)



# [Reduced Instruction Set Computer vs Complex Instruction Set Computer(RISC vs CISC)](https://youtu.be/6Rxade2nEjk?si=3eLW8NJ_LSvNFM3b)
![](https://media.geeksforgeeks.org/wp-content/uploads/20240313171140/RISC-and-CISC.png)

| **Aspect**               | **RISC**                                                                 | **CISC**                                                                 |
|--------------------------|--------------------------------------------------------------------------|--------------------------------------------------------------------------|
| **Instruction Size**     | Fixed-length (e.g., 32 bits)                                | Variable-length                                             |
| **Clock Cycles**         | 1 cycle per instruction                                     | Multiple cycles per instruction                             |
| **Registers**            | Large number of general-purpose registers                   | Fewer registers                                             |
| **Addressing Modes**     | Simple (e.g., register-to-register)                         | Complex (e.g., memory-to-memory)                            |
| **Power Consumption**    | Lower                                           | Higher                                          |
| **Code Size**            | Larger (more instructions needed)                           | Smaller (fewer instructions)                                |
| **Pipelining**           | Highly efficient                                            | Less efficient        


- **RISC Architectures**:
  - **Mobile Devices**: ARM processors (e.g., smartphones, tablets) due to power efficiency .
  - **Embedded Systems**: IoT devices, automotive systems (e.g., ECUs) .
  - **High-Performance Computing**: ARM-based servers (e.g., AWS Graviton) and supercomputers .
  - **Examples**: ARM, RISC-V, MIPS, SPARC .

- **CISC Architectures**:
  - **Desktop and Laptops**: x86 processors (Intel Core, AMD Ryzen) for compatibility and performance .
  - **Servers and Workstations**: Enterprise systems handling complex workloads .
  - **Examples**: Intel x86, AMD64, VAX .

---


## Addressing Modes in Computer Architecture

| Addressing Mode | Description | Effective Address (EA) Calculation | Example & Explanation | Key Advantage |
| :--- | :--- | :--- | :--- | :--- |
| **Implied/Implicit** | The operand is specified implicitly by the instruction. | None. | `CLC` (Clear Carry Flag). The instruction itself defines the operand (the flag). | Very compact instruction. |
| **Immediate** | The operand is a constant value contained within the instruction. | **EA = Next byte(s) of the instruction.** The value is immediately available. | `MOV R1, #42`<br>Loads the **value** `42` directly into register R1. | Very fast; no memory reference for data. |
| **Register Direct** | The operand is located in a CPU register. | **EA = Register ID.** The value is in the named register. | `ADD R3, R2`<br>Adds the **contents of register R2** to register R3. | Fastest; operands are in the CPU. |
| **Register Indirect** | The specified register contains the memory address of the operand. | **EA = [Register]**<br>The value inside the register is the address to fetch from. | `LOAD R1, (R2)`<br>If R2 contains value `2000`, it loads R1 with the data from **memory address 2000**. | Provides indirection; efficient for pointers. |
| **Direct (Absolute)** | The instruction contains the direct memory address of the operand. | **EA = Address Field**<br>The address in the instruction is used directly. | `LOAD R1, [2048]`<br>Loads R1 with the contents of **memory address 2048**. | Simple; requires only one memory lookup for data. |
| **Memory Indirect** | The instruction contains a memory address which in turn holds the address of the operand. | **EA = [[Address]]**<br>Go to the address in the instruction, then use the value found there as the real address. | `LOAD R1, @2048`<br>1. Go to address `2048`.<br>2. Find the value at `2048` (e.g., `3500`).<br>3. Go to address `3500` to get the operand for R1. | Highly flexible (e.g., pointer to a pointer). |
| **Displacement** | Combines a direct address with the contents of a register. Has three primary subtypes: | **EA = Register + Address Field** | | Powerful; basis for advanced data structures. |
| ↳ **Relative** | Uses the **Program Counter (PC)** as the register. | **EA = PC + Address Field** | `JUMP +50`<br>Jump to an address 50 bytes ahead of the current instruction. | Position-independent code. |
| ↳ **Base-Register** | Uses a dedicated **Base Register** (holds start of a segment/array). | **EA = Base Reg + Address Field** | `LOAD R1, 100(BaseReg)`<br>If `BaseReg` holds `2000`, EA is `2000 + 100 = 2100`. | Memory relocation & protection. |
| ↳ **Indexed** | Uses an **Index Register** (holds an offset/position). | **EA = Address Field + Index Reg** | `LOAD R1, Array(Rindex)`<br>If `Array` is at `3000` and `Rindex=5`, EA is `3005`. | Efficient array processing. |
| **Stack** | The operand is implicitly at the top of the stack, managed by the Stack Pointer (SP). | **EA = [SP]**<br>The SP register contains the memory address. | `PUSH R1` (Decrement SP, then store R1 at new [SP])<br>`POP R2` (Load R2 from [SP], then increment SP) | Simplifies subroutine calls and expression evaluation. |

---



## What is [Pipelining](https://www.youtube.com/watch?v=BSLLDXQTqmM&list=PLTd6ceoshprfg23JMtwGysCm4tlc0I1ou&index=5&pp=iAQB)?

In simple terms, **pipelining** is an implementation technique where multiple instructions are overlapped in execution. It's like an assembly line in a factory: while one stage of a product is being worked on, the next stage can simultaneously work on a different product.

Without pipelining, a processor would finish executing one entire instruction before starting the next one. This is inefficient, as parts of the processor sit idle while a single instruction is being processed.

Pipelining increases the **throughput** of the system—the number of instructions completed per unit of time—even though it does not reduce the execution time of a single instruction (called **latency**).

![](https://upload.wikimedia.org/wikipedia/commons/a/a7/Exception_et_pipeline.png?20130309134922)

---

### The CPU Instruction Pipeline

A classic RISC pipeline breaks down instruction execution into five distinct stages:

1.  **IF (Instruction Fetch):** Fetch the instruction from memory.
2.  **ID (Instruction Decode):** Decode the instruction and read registers.
3.  **EX (Execute):** Perform the arithmetic or logic operation.
4.  **MEM (Memory Access):** Access memory if needed (e.g., load or store data).
5.  **WB (Write Back):** Write the result back to a register.

---

### Why is this Crucial for RISC vs. CISC?

This is a fundamental reason why RISC and CISC architectures are designed differently.

*   **RISC (e.g., ARM, RISC-V):** Designed with pipelining in mind from the start.
    *   **Fixed-length instructions** make the "IF" and "ID" stages simple and fast. The decoder knows exactly where each instruction begins and ends.
    *   **Simple, single-cycle instructions** make the "EX" stage predictable. This allows the pipeline to run smoothly without frequent stalls.
    *   **Load-Store Architecture** (where only specific instructions access memory) simplifies the "MEM" stage.

*   **CISC (e.g., x86):** Has instructions that are complex and take varying, often multiple, clock cycles to execute.
    *   **Variable-length instructions** make the "IF" and "ID" stages complex. The CPU must decode an instruction to even know how long the next one is, which can stall the pipeline.
    *   **Complex instructions** (e.g., `ADD` that can work directly on memory) can have wildly different execution times, causing pipeline "bubbles" or stalls as the CPU waits for one long instruction to finish.

**Modern CISC processors (like Intel x86) get around this** by translating their complex CISC instructions into simpler, RISC-like internal micro-operations (µops). These µops are then executed by a high-performance internal RISC-style pipeline. So, a modern x86 CPU is essentially a CISC processor on the outside (for compatibility) but a RISC machine on the inside (for performance).

---

### The Big Challenge: Pipeline Hazards

Pipelining isn't perfect. Problems called **hazards** can force the pipeline to stall, reducing efficiency. The main types are:

*   **Structural Hazard:** A hardware resource conflict (e.g., if the CPU only has one memory port but needs to fetch an instruction and data at the same time).
*   **Data Hazard:** An instruction depends on the result of a previous instruction that hasn't finished yet (e.g., `ADD R1, R2, R3` followed by `SUB R4, R1, R5`). Solved by **forwarding** (bypassing results early) or stalling.
*   **Control Hazard (Branch Hazard):** Caused by branch instructions (like `if` statements). The CPU doesn't know which instruction to fetch next until the branch is resolved, potentially wasting cycles. Solved by **branch prediction**.
