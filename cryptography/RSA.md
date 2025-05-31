#TODO

The RSA algorithm is a public-key cryptosystem that relies on number-theoretic principles for secure communication. 
---

### **1. Key Generation**
1. **Choose two large primes `p` and `q`:**
   - These primes are kept secret.
2. **Compute `n = p * q`:**
   - `n` is the modulus used for encryption and decryption.
3. **Compute Euler’s totient function `ϕ(n)`:**
   - `ϕ(n) = (p-1)*(q-1)`.
   - **Proof:** For distinct primes `p` and `q`, `ϕ(n) = ϕ(pq) = ϕ(p)*ϕ(q) = (p-1)*(q-1)`.
4. **Choose public exponent `e`:**
   - `1 < e < ϕ(n)`, and `gcd(e, ϕ(n)) = 1`.
   - **Proof:** `e` must be invertible modulo `ϕ(n)`, which requires `e` and `ϕ(n)` to be coprime (Bézout’s identity).
5. **Compute private exponent `d`:**
   - `d ≡ e^(-1) mod ϕ(n)`, i.e., `e*d ≡ 1 mod ϕ(n)`.
   - **Proof:** Use the Extended Euclidean Algorithm to solve `e*d + k*ϕ(n) = 1` for integers `d` and `k`.

---

### **2. Encryption**
- **Public key:** `(n, e)`.
- **Encrypt message `m`:**
  - Convert `m` to an integer `0 ≤ m < n`.
  - Compute ciphertext `c ≡ m^e mod n`.

---

### **3. Decryption**
- **Private key:** `(d, p, q)`.
- **Decrypt ciphertext `c`:**
  - Compute `m ≡ c^d mod n`.

---

### **4. Proof of Correctness**
We prove `c^d ≡ m mod n`.

#### **Case 1: `gcd(m, n) = 1`**
- By Euler’s theorem: `m^ϕ(n) ≡ 1 mod n`.
- Since `e*d ≡ 1 mod ϕ(n)`, write `e*d = 1 + k*ϕ(n)`.
- Then:
  ```
  c^d ≡ (m^e)^d ≡ m^(e*d) ≡ m^(1 + k*ϕ(n)) ≡ m * (m^ϕ(n))^k ≡ m * 1^k ≡ m mod n.
  ```

#### **Case 2: `gcd(m, n) ≠ 1`**
- Since `n = p*q`, `m` is a multiple of `p` or `q`. Assume `m ≡ 0 mod p`.
- By the Chinese Remainder Theorem (CRT):
  - **Modulo `p`:** `m^(e*d) ≡ 0 ≡ m mod p`.
  - **Modulo `q`:** Since `e*d ≡ 1 mod ϕ(n)`, `e*d = 1 + t*(q-1)`.
    ```
    m^(e*d) ≡ m^(1 + t*(q-1)) ≡ m * (m^(q-1))^t ≡ m * 1^t ≡ m mod q` (by Fermat’s Little Theorem).
    ```
- Combining via CRT: `m^(e*d) ≡ m mod n`.

---

### **5. Algorithms Used in RSA**
1. **Extended Euclidean Algorithm:**
   - Finds `d = e^(-1) mod ϕ(n)`.
   - Solves `a*x + b*y = gcd(a, b)`. For `a = e`, `b = ϕ(n)`, it gives `d`.

2. **Modular Exponentiation:**
   - Efficiently computes `m^e mod n` using **exponentiation by squaring** (time `O(log e)`).

3. **Primality Testing (e.g., Miller-Rabin):**
   - Probabilistic test to generate large primes `p` and `q`.

---

### **6. Security of RSA**
- Relies on the **hardness of factoring `n` into `p` and `q`**.
- Best-known factoring algorithm: **General Number Field Sieve** (sub-exponential complexity).
- If `n` is factored, `ϕ(n)` and `d` can be computed, breaking RSA.

---

### **Summary**
- **Key Insight:** RSA uses Euler’s theorem and CRT to ensure `m^(e*d) ≡ m mod n`.
- **Algorithms:** Extended Euclidean, modular exponentiation, primality testing.
- **Security:** Based on the difficulty of factoring large integers.

Padding schemes (e.g., OAEP) are required in practice to prevent attacks like chosen-ciphertext attacks.

---
