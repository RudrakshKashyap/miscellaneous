### **[Diffie-Hellman Key Exchange (DHKE)](https://www.youtube.com/watch?v=Yjrfm_oRO0w)**  

- Allows two parties to securely establish a shared secret key over an insecure channel.  
- Used in protocols like SSL/TLS, SSH, and IPsec.  

#### **Key Concepts**:  
We know that $g^{ab} = (g^a)^b = (g^b)^a$ and goal is to derive $g^{ab}$, we ask other person to calcuate $g^b$ and give it to us so that we can then do the rest of the calcuation and raise it to power $a$. Idea is that it's hard to get $b$ from $g^b mod\ p$. Ofcourse every thing is taken $mod\ p$ here.

**TODO** - why modulo needs to be prime, -> some Chinese remainder theorm shit.

1. **Public Parameters**:  
   - Large prime number $p$ (modulus).  
   - Generator $g$ (primitive root modulo $p$).  

2. **Private & Public Keys**:  
   - Each party selects a **private key** ($a$ for Alice, $b$ for Bob).  
   - Computes **public key**:  
     - Alice: $A = g^a \mod p$  
     - Bob: $B = g^b \mod p$  

3. **Shared Secret Calculation**:  
   - Alice computes: $S = B^a \mod p$  
   - Bob computes: $S = A^b \mod p$  
   - Both arrive at the same **shared secret** $S = g^{ab} \mod p$.  

#### **Security**:  
- Relies on the **Discrete Logarithm Problem (DLP)** (hard to find $a$ from $A = g^a \mod p$).  
- Vulnerable to **Man-in-the-Middle (MITM) attacks** if not authenticated. You still needs authentication to verify the sender's identity (Parties exchange public keys beforehand or get from a trusted source / Digital signatures/ Certificates).

### **[Elliptic Curve Diffie-Hellman (ECDH)](https://www.youtube.com/watch?v=NF1pwjL9-DE)**
Uses elliptic curves for better efficiency. [Backdoor.. ? ?](https://youtu.be/gAtBM06xwaw?si=EHyGRPuZRtm8DhLK&t=367) [Backdoor.. ? ?](https://youtu.be/ulg_AHBOIQU?si=RGoVzXdO4-0JfkiI&t=337) [Backdoor.. ? ?](https://www.youtube.com/watch?v=nybVFJVXbww&t=630s)

ECC is considered asymmetric because it involves public-private key pairs, even though the shared secret is symmetric, but the key exchange mechanism is not.

#### **Key Concepts**  
- **Elliptic Curve (EC):** A mathematical structure defined by the equation:  
  $ y^2 = x^3 + ax + b $ (over a finite field).  
- **Private Key (d):** A randomly selected integer (kept secret).  
- **Public Key (Q):** Computed as $ Q = d \times G $, where $G $ is the base point (generator).  

#### **ECDH Key Exchange Steps**  
1. **Key Generation:**  
   - Alice & Bob generate their private & public keys:  
     - Alice: $ (d_A, Q_A = d_A \times G) $  
     - Bob: $ (d_B, Q_B = d_B \times G) $  
2. **Public Key Exchange:**  
   - Alice sends $ Q_A $ to Bob, Bob sends $ Q_B $ to Alice.  
3. **Shared Secret Computation:**  
   - Alice computes: $ S = d_A \times Q_B $  
   - Bob computes: $ S = d_B \times Q_A $  
   - Both arrive at the same point $ S = d_A \times d_B \times G $.  
4. **Derive Symmetric Key:**  
   - Use the x-coordinate (or hash) of $ S $ as the shared key.  

#### **Security**  
- Relies on the **Elliptic Curve Discrete Logarithm Problem (ECDLP)**.  
- Eavesdroppers cannot compute $ d_A $ or $ d_B $ from $ Q_A, Q_B $.  
- Provides **forward secrecy** if ephemeral keys are used (ECDHE).  

#### **Advantages over Classic DH**  
- **Shorter key sizes** for equivalent security (e.g., 256-bit ECC ≈ 3072-bit RSA/DH).  
- **Faster computations**, lower resource usage.  
