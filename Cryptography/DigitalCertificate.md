
![](https://cheapsslsecurity.com/blog/wp-content/uploads/2018/09/how-do-digital-signatures-and-digital-certificates-work-together-in-ssl.png)


## DIGITAL SIGNATURE = Priv_key_en(HASH(Data)) [in case of RSA algo]
## GET SENDER's PUBLIC KEY FROM CERTIFICATE TO VERIFY THE SIGNATURE
## GET CA's PUBLIC KEY FROM BROWSER TO VERIFY THE SIGNATURE OF CERTIFICATE
## DIGITAL CERTIFICATE = $Signature_{cert}$ + $Sender\_Pub_k$ + Data(CN of CA, Sender and other stuff)

### **Verify the Certificate Chain (Chain of Trust)**
The verifier checks if the certificate chains back to a **trusted root CA**:
1. **Root CA Trust Check**  
   - The verifier checks if the **root CA** is in its **trust store** (e.g., Windows Certificate Store, Firefox Trusted Roots).
   - Example trusted CAs: DigiCert, Let’s Encrypt, GlobalSign.

2. **Intermediate CA Validation**  
   - Each intermediate certificate must be **signed by the next higher CA**.
   - Example chain:  
     ```
     End-entity Cert → Signed by → Intermediate CA → Signed by → Root CA
     ```
   - Each link in the chain must be **valid and properly signed**.



### Example SSL Certificate Details for `cheapsslsecurity.com`

Purpose of the SHA-256 Fingerprint 
- Quick Identification:
- Allows humans or systems to uniquely identify a certificate without parsing its full contents.


| **Field**     | **Value**  | **Description** |
|-------------------------------|-------------------|----------------------------------------------------------------------------------------------------------------|
| **Signature**                 | `30 45 02 20 38 06 CD B4 A1 E6 5C E4 30 80 DB 00 1A 10 B3 EF E3 19 34 FD BB E6 DB 5A 29 5D 50 54 03 4E 6D B8 02 21 00 98 49 DB 55 85 BE C9 5B AD E9 D5 2A 5A C8 46 BB F4 59 25 3D EA A8 AF 5D B2 A2 39 C4 98 D2 06 32` | Cryptographic signature by issuer                                                                              |
| Signature Algorithm           | `SHA256 with ECDSA`                                                       | Hash + signing scheme used                                                                                     |
| **Subject**                   |       |     |
| Common Name (CN)              | `cheapsslsecurity.com`                                                    | Primary domain name covered by the certificate                                                                 |
| **Issuer**                    |                                                                           |                                                                                                                |
| Country (C)                   | `US`                                                                      | Issuing country                                                                                                |
| Organization (O)              | `Google Trust Services`                                                   | Certificate Authority (CA)                                                                                     |
| Common Name (CN)              | `WE1`                                                                     | Specific issuing entity (Google's Web Environment 1)                                                           |
| **Validity Period**           |                                                                           |                                                                                                                |
| Not Valid Before              | `2025-04-04`                                                              | Certificate activation date                                                                                    |
| Not Valid After               | `2025-07-03` (Expires: 03/07/25)                                          | Expiration date                                                                                                |
| **Technical Details**         |                                                                           |                                                                                                                |
| Serial Number                 | `1C:2F:17:BE:19:F2:14:D3:13:0B:27:18:0C:14:81:79`                        | Unique identifier for this certificate                                                                          |
| Version                       | `3`                                                                       | X.509 v3 certificate (supports extensions)                                                                     |
| Public Key Algorithm          | `Elliptic Curve`                                                          | Uses ECC (Elliptic Curve Cryptography)                                                                         |
| Key Parameters (OID)          | `1.2.840.10045.3.1.7` (secp256r1)                                        | NIST P-256 curve parameters                                                                                    |
| Key Size                      | `256 bits`                                                                | Equivalent security to 3072-bit RSA                                                                            |
| Public Key (Subject's)      | `04 49 20 5C CD 2B 11 60 70 CA C3 31 03 43 88 4D 06 7B D1 9E 9A F4 27 77 01 7A 0F A9 33 5D 79 D7 1B 74 38 C6 B4 B6 E2 1A 8B 49 2A 4E EA 56 EA 78 8C AC 84 8F A3 4B E0 1E 52 EA 54 57 7C 1E F4 E4 E0`                | Uncompressed ECC public key   (Q = (x, y) coordinates on the curve)         |
**Fingerprints**              |                                                                           |                                                                                                                |
| SHA-1                         | `EF:F1:7C:3A:5A:00:D8:23:5D:7B:DD:F4:56:C6:DE:E6:58:7B:C5:27`             | Unique certificate hash (legacy, insecure)                                                                     |
| MD5                           | `52:F3:64:EE:D2:9F:47:AC:E3:03:5C:8A:A1:38:76:8E`                         | Deprecated hash (insecure)                                                                                     |
| Public Key SHA-1              | `C7:6F:35:63:B2:AF:8A:B0:76:77:CF:5D:4D:6C:B6:6D:8A:8C:5C:1B`             | Fingerprint of public key only                                                                                 |
| **Key Usage**                 |                                                                           |                                                                                                                |
| Key Usage                     | `Digital Signature`                                                       | Permitted use: Signing data (Critical)                                                                         |
| Extended Key Usage (EKU)      | `Server Authentication` (TLS Web Server)                                  | Allows use for HTTPS servers (Non-critical)                                                                    |
| **Constraints**               |                                                                           |                                                                                                                |
| Basic Constraints             | `CA: FALSE`, `Max Path Length: Unlimited`                                 | Not a Certificate Authority (Critical)                                                                         |
| **Extensions**                |                                                                           |                                                                                                                |
| Subject Alternative Names     | `cheapsslsecurity.com`, `*.cheapsslsecurity.com`                          | Covers base domain + all subdomains (Non-critical)                                                             |
| Subject Key Identifier        | `F1:B2:A8:40:77:7C:A8:AF:A3:C2:E5:51:A5:60:E7:AB:8C:76:A8:9E`             | Unique ID for key (Non-critical)                                                                               |
| Authority Key Identifier      | `90:77:92:35:67:C4:FF:A8:CC:A9:E6:7B:D9:80:79:7B:CC:93:F9:38`             | Issuer's key ID (Non-critical)                                                                                 |
| Authority Information Access  | OCSP: `http://o.pki.goog/s/we1/HC8`<br>CA Issuers: `http://i.pki.goog/we1.crt` | OCSP(Online Certificate Status Protocol) responder + CA certificate URLs (Non-critical)                                              |
| CRL Distribution Points       | `http://c.pki.goog/we1/pVHkAbbILwY.crl`                                   | URL for revocation list (Non-critical)                                                                         |
| Certificate Policies          | OID `1.3.159.1.17.1.2.1` (Google Trust Services)                          | Policy identifier (Non-critical)                                                                               |
| Signed Certificate Timestamps | CT Log Data (Embedded SCTs)                                               | Proof of certificate transparency (Non-critical)                                                               |


# ECDSA ([Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm#Security)) Explained

ECDSA is a cryptographic algorithm used for creating digital signatures. It's a variant of the Digital Signature Algorithm (DSA) that uses elliptic curve cryptography (ECC) instead of the more traditional modular arithmetic used in DSA. In ECDSA (Elliptic Curve Digital Signature Algorithm), there is **no encryption or decryption** involved. ECDSA is a signature scheme, not an encryption scheme. Unlike **ECIES (Elliptic Curve Integrated Encryption Scheme)**, which does perform encryption and decryption, ECDSA is **only for signing and verification**.

## Key Components

1. **Elliptic Curve Cryptography (ECC)**: 
   - Based on the algebraic structure of elliptic curves over finite fields
   - Provides equivalent security to RSA with much smaller key sizes (e.g., 256-bit ECC ≈ 3072-bit RSA)

2. **Digital Signatures**:
   - Provides authenticity (proves who created the message)
   - Provides integrity (proves message wasn't altered)
   - Provides non-repudiation (signer can't deny signing)

## [How ECDSA Works](https://youtu.be/U2bw_N6kQL8?si=W50wC8KoiDa1BG9W&t=591)

### Key Generation
1. Select an elliptic curve and a base point $G$ (generator) on that curve
2. Choose a private key $d$ (a random integer)
3. Compute public key $Q = d × G$ (elliptic curve point multiplication)

### Signing a Message
1. Hash the message to create a fixed-size digest: $h = hash(message)$
2. Select a random integer $k$ (called a nonce)
3. Compute point $(x₁, y₁) = k × G$
4. Compute $r = x₁\ mod\ n$ (where $n$ is the order of the curve)
5. Compute $s = k⁻¹(h + r·d) mod\ n$
6. The signature is the pair $(r, s)$

### Verifying a Signature
1. Hash the original message to get $h$
2. Compute $w = s⁻¹\ mod\ n$
3. Compute $u₁ = h·w\ mod\ n$ and $u₂ = r·w\ mod\ n$
4. Compute point $(x₂, y₂) = u₁×G + u₂×Q$
5. The signature is valid if $x₂ ≡ r\ mod\ n$

## Security Considerations

- **[Nonce (k) must be random](https://www.youtube.com/watch?v=-UcCMjQab4w)**: If k is predictable or reused, the private key can be recovered
- **Key size**: 256-bit ECDSA provides 128-bit security (resistant to brute force)
- **[Curve selection](https://www.youtube.com/watch?v=wpLQZhqdPaA&list=PLmL13yqb6OxdEgSoua2WuqHKBuIqvll0x&index=11)**: NIST curves like secp256k1 (used in Bitcoin) are commonly used

## Advantages over RSA

- Smaller key sizes for equivalent security
- Faster computations
- Smaller signature sizes

## Common Uses

- Bitcoin and other cryptocurrencies
- TLS/SSL certificates
- Secure messaging protocols
- Digital identity systems


# Resources
- [An Introduction to the Arithmetic of Elliptic Curves](https://youtube.com/playlist?list=PLYpVTXjEi1oe1OeAllJpNhFoI4B7Ws8Yl&si=jZS4_Sd605tXomgE)
- [Random number generators using ECC](https://youtu.be/yBr3Q6xiTw4?si=BRwmTvWb1kn7UR76&t=1202)

## **RSA-based** vs **ECDSA-based** signature verification

### **1. RSA Signatures: Direct "Encryption/Decryption" of the Hash**
- **Signing**:  
  - The hash of the message ($H$) is **padded** (e.g., PKCS#1) and then transformed using the **private key** ($d$):  
    $$S = \text{Pad}(H)^d \mod n$$
    - This is mathematically equivalent to **"encrypting" the hash with the private key** (though we avoid calling it "encryption" to prevent confusion with actual RSA encryption).

- **Verification**:  
  - The signature ($S$) is transformed back using the **public key** ($e$):  
    $$H' = S^e \mod n$$
    - This reverses the signing operation (like "decrypting" with the public key).  
  - The verifier compares $H'$ with their own hash of the message ($H$).  

#### **Key Point**:  
RSA signature verification **does involve modular exponentiation** (similar to RSA encryption/decryption) applied to the **hash**.

---

### **2. ECDSA Signatures: No "Encryption/Decryption" of the Hash**
- **Signing**:  
  - The hash ($H$) is combined with a **random nonce** ($k$) and the private key ($d$) in a **elliptic curve operation** to produce a signature ($r, s$).  
  - No step resembles "encrypting" the hash.  

- **Verification**:  
  - The verifier uses the public key ($Q$), the signature ($r, s$), and the hash ($H$) to compute elliptic curve points and checks if they align.  
  - **No "decryption" occurs**—just mathematical validation of curve operations.  

#### **Key Point**:  
ECDSA relies entirely on **elliptic curve algebra**, not modular exponentiation. There's no "undoing" an encryption step.

---
✅ **RSA** signature verification involves **direct modular exponentiation on the hash** (resembling "decryption"), while **ECDSA** does not—it's purely elliptic curve arithmetic.  
