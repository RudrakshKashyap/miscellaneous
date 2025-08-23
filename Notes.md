A **nibble** is of 4 bits, and can be represented as 1 hexadecimal digit.


## [Big-Endian vs. Little-Endian](https://youtu.be/CounrFEsOeA?si=G2pnKXjh-bN4enIA&t=376)

- **Big-Endian**: The **most significant byte (MSB)** is stored at the **lowest memory address**.
- **Little-Endian**: The **least significant byte (LSB)** is stored at the **lowest memory address**. Memonic -> LLL => Little Least Lowest

Assume a 32-bit integer `0x12345678` is stored at memory address `0x100`.

| Memory Address | Big-Endian | Little-Endian |
|----------------|------------|---------------|
| **0x100**      | `0x12` (MSB) | `0x78` (LSB)  |
| **0x101**      | `0x34`      | `0x56`        |
| **0x102**      | `0x56`      | `0x34`        |
| **0x103**      | `0x78` (LSB) | `0x12` (MSB)  |
| **Common Usage** | Network protocols (e.g., TCP/IP), Motorola processors, Standard for network byte order (e.g., IPv4 headers). | x86, ARM, most modern CPUs, Some formats (e.g., JPEG) use big-endian. | 
---
