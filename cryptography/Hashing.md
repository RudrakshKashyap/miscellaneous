## [Types of security of Hash functions](https://en.wikipedia.org/wiki/Security_of_cryptographic_hash_functions#Types_of_security_of_hash_functions)


## [MAC/HMAC](https://www.youtube.com/watch?v=fzMIjWFYQl0)

A **Message Authentication Code (MAC)** is a cryptographic technique used to verify both the **integrity** and **authenticity** of a message. It combines a *message* with a *secret key* to produce a short piece of data (the MAC). It ensures:
1. **Data Integrity** – The message has not been altered in transit.
2. **Authenticity** – The message comes from the expected sender (and not an impostor).

Unlike digital signatures, MACs are faster but require a pre-shared key.

### **Types of MAC Algorithms**
#### **1. HMAC (Hash-based MAC)**
- Uses a **cryptographic hash function (e.g., SHA-256)** + a **secret key**.
- Example: `MAC = HMAC-SHA256(key, message)`
- Secure against length-extension attacks.
- Used in:
  - API authentication (e.g., AWS signatures).
  - JWT (JSON Web Tokens).
  - TLS/SSL.

#### **2. CMAC (Cipher-based MAC)**
- Uses a **block cipher (e.g., AES)** instead of a hash function.
- Example: `AES-CMAC`
- Used in network security protocols like **IPSec**.

#### **3. Poly1305 (Fast MAC for Modern Cryptography)**
- Often paired with **ChaCha20** (e.g., `ChaCha20-Poly1305`).
- Used in TLS 1.3 for high-speed encryption.

### **Real-World Uses of MAC**
**API Security** – HMAC ensures that API requests are from trusted clients.<br/>
**Banking Transactions** – Prevents tampering in payment messages.<br/>
**Secure Messaging (WhatsApp, Signal)** – Uses MAC to verify message integrity.<br/>
**TLS/HTTPS** – Combines MAC with encryption for secure web traffic.


# [MD5](https://youtu.be/5MiMK45gkTY?si=goovwf1542LUJ4Q8&t=353)

The 128-bit (16-byte) MD5 hashes (also termed message digests) are typically represented as a sequence of 32 hexadecimal digits. 

![](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a5/MD5_algorithm.svg/330px-MD5_algorithm.svg.png)

Above figure shows one MD5 operation. MD5 consists of 64 of these operations, grouped in four rounds of 16 operations. $F$ is a nonlinear function; one function is used in each round. $M_i$ denotes a 32-bit block of the message input, and $K_i$ denotes a 32-bit constant, different for each operation. $<<<_S$ denotes a left bit rotation by $s$ places; $s$ varies for each operation. 
${\displaystyle \boxplus }$ denotes addition modulo 232.

### [Algorithm, Pseudocode & More info](https://en.wikipedia.org/wiki/MD5#Algorithm)

Despite being cryptographically broken due to collision vulnerabilities, **MD5** is still used in specific non-security-critical applications where speed and simplicity outweigh the need for collision resistance. Here are the primary areas where MD5 persists today:

### **1. Data Integrity Verification (Checksums)**
- **File Downloads**: Websites provide MD5 hashes alongside downloadable files to verify they weren’t corrupted during transfer. Users compare the generated hash with the provided one to ensure integrity .
- **Backup Validation**: MD5 checksums help confirm backup files match the originals, detecting accidental corruption (e.g., storage errors or transmission issues) .
- **Software Distribution**: Developers publish MD5 checksums with releases to ensure users install unmodified binaries .

### **2. Non-Cryptographic Database Operations**
- **Partitioning/Indexing**: Databases use MD5 to quickly generate unique keys for partitioning or indexing data, leveraging its speed and fixed output size .
- **Deduplication**: Systems compare MD5 hashes to identify duplicate files or records without storing full content .

### **3. Legacy Systems and Embedded Devices**
- **Low-Resource Environments**: IoT devices or embedded systems with limited computational power use MD5 for lightweight tasks like firmware checksums .
- **Legacy Software**: Older applications or protocols (e.g., some third-party APIs) still rely on MD5 for backward compatibility .

### **4. Debugging and Logging**
- **Unique Identifiers**: MD5 generates short, consistent hashes for log entries or cache keys where security isn’t a concern (e.g., tracking API calls) .


### **Key Limitations and Risks**
- **Collision Attacks**: MD5 is vulnerable to intentional tampering (e.g., forged certificates or malware like **Flame**) .
- **No Security Use**: Avoid MD5 for passwords, digital signatures, or any scenario where collision resistance matters .

<br/>

# [SHA(Secure Hash Algorithm)-256](https://youtu.be/f9EbD6iY9zI?si=rmUhe1WaytjDRwqo&t=507)

As of 2011, the best public attacks break preimage resistance for 52 out of 64 rounds of SHA-256 or 57 out of 80 rounds of SHA-512, and collision resistance for 46 out of 64 rounds of SHA-256.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7d/SHA-2.svg/500px-SHA-2.svg.png)

**Ch(E, F, G)** (Choose function): If bit of E is one choose bit from F otherwise G.

**Ma(A, B, C)** (Majority function): Choose majority bit from A,B & C.

**Σ₀(A)** (Sigma-0 rotation): $
   \Sigma_{0}(A) = (A \ggg 2) \oplus (A \ggg 13) \oplus (A \ggg 22)  
   $

**Σ₁(E)** (Sigma-1 rotation):  $
   \Sigma_{1}(E) = (E \ggg 6) \oplus (E \ggg 11) \oplus (E \ggg 25)
   $

**Notes:**  
- The bitwise rotation uses different constants $K_i$ for **SHA-512**. The given numbers apply to **SHA-256**.  
- The red ${\displaystyle \boxplus }$ denotes **addition modulo** $2^{32}$ for SHA-256 or $2^{64}$ for SHA-512.  
### [Pseudocode & More info](https://en.wikipedia.org/wiki/SHA-2#Pseudocode)

## [Comparison of SHA functions](https://en.wikipedia.org/wiki/SHA-2#Comparison_of_SHA_functions)
| Feature      | SHA-0 (1993) | SHA-1 (1995) | SHA-2 (2001) | SHA-3 (2015) |
|-------------|-------------|-------------|-------------|-------------|
| **Hash Size** | 160-bit | 160-bit | 224/256/384/512-bit | 224/256/384/512-bit |
| **Structure** | Merkle-Damgård | Merkle-Damgård | Merkle-Damgård | **Keccak Sponge** |
| **Security Status** | Broken | Broken (2017) | Secure | Secure |
| **Collision Resistance** | Weak | Broken | Strong | Strong |
| **Usage** | Never adopted | Legacy (Git, old certs) | **Most common (SSL, Bitcoin, TLS)** | Emerging (IoT, backup) |


The **Merkle–Damgård construction** is a method for building cryptographic hash functions from compression functions. <br/> The construction processes an input message in blocks, using a compression function repeatedly to produce a fixed-size hash. 

[![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Merkle-Damgard_hash_big.svg/500px-Merkle-Damgard_hash_big.svg.png)](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction)

---

# TODO SHA-3 + Sponge Construction

---

### **Salt and Pepper in Password Security**  
| Feature      | **Salt** | **Pepper** |  
|-------------|---------|-----------|  
| **Uniqueness** | Different per user | Same for all users |  
| **Storage** | Stored in DB | Kept separately (e.g., env vars) |  
| **Purpose** | Prevents rainbow tables(Precomputed table for caching the outputs of a cryptographic hash function) | Adds secret key protection | 

## Why bcrypt is Preferred Over SHA for Password Hashing

- A **BCrypt hash** typically looks like this:  
```
$2a$10$N9qo8uLOickgx2ZMRZoMy.E3B7FxtgTH9In0uWb3M..lZGdC5V6W.
```
Breaking it down:  
- **`$2a$`** → Algorithm identifier (BCrypt version).  
- **`10$`** → Cost factor (2^10 rounds).  
- **`N9qo8uLOickgx2ZMRZoMy.`** → **The 22-character salt** (before the last `$`).  
- **`E3B7FxtgTH9In0uWb3M..lZGdC5V6W.`** → The actual hashed password. 

Based on the **Blowfish cipher**(Feistel Network (16 Rounds)) (BCrypt derive its key from `password + salt`), **bcrypt** is generally preferred over SHA (Secure Hash Algorithm) variants for password storage because it was specifically designed for password hashing, while SHA was designed for general cryptographic purposes. Here are the key reasons:

### 1. Built-in Work Factor (Cost Factor)
- bcrypt has an adjustable work factor that allows you to increase the computational cost as hardware improves
- This makes it resistant to brute force attacks over time
- SHA has no built-in way to adjust its computational complexity

### 2. Salt Handling
- bcrypt automatically generates and manages salts for each password
- With SHA, you must manually implement salt generation and storage

### 3. Resistance to GPU/ASIC Attacks
- bcrypt's memory-intensive algorithm makes it harder to parallelize on GPUs or custom hardware
- SHA algorithms (especially SHA-256/SHA-512) are easily accelerated with modern hardware


### 4. Slower by Design
- bcrypt is intentionally slow (a feature for password hashing)
- SHA is designed to be fast, which is bad for password hashing


For these reasons, security experts recommend **bcrypt** (or other password-specific hashing algorithms like **Argon2** or **PBKDF2**) over SHA variants for password storage.