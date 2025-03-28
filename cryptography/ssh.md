# SSH (Secure Shell) - Complete Guide

SSH (Secure Shell) is a cryptographic network protocol that allows secure remote access and data communication between computers over an unsecured network (like the internet). It encrypts all traffic to prevent eavesdropping, connection hijacking, and other attacks.

---

## 1. How SSH Works
SSH works on a **client-server model**:
- **SSH Client**: Initiates the connection (e.g., `ssh` command, PuTTY, MobaXterm).
- **SSH Server**: Listens for incoming connections (usually on **port 22**).

### Key Concepts:
- **Encryption**: Uses strong cryptographic algorithms (AES, ChaCha20) to secure data.
- **Authentication**: Verifies the identity of the client and server.
- **Integrity**: Ensures data isn’t modified in transit (using HMAC-SHA2).

---

## 2. SSH Authentication Methods
### a) Password Authentication
- The client logs in with a username and password.
- **Weakness**: Vulnerable to brute-force attacks.

### b) Public Key Authentication (Recommended)
- Uses a **key pair**:
  - **Private Key** (stored on the client, must be kept secret).
  - **Public Key** (stored on the server in `~/.ssh/authorized_keys`).
- More secure than passwords (resistant to brute force).

#### How Key-Based Auth Works:
1. Client sends its public key to the server.
2. Server checks if the key is in `authorized_keys`.
3. Server sends a challenge encrypted with the public key.
4. Client decrypts it with the private key and responds.
5. If successful, access is granted.

---

## 3. SSH Key Generation
Generate a key pair using:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"  # Best for modern systems
```
- Keys are stored in ~/.ssh/id_algorithm (e.g., id_ed25519, id_rsa).
- The .pub file is the public key (can be shared).
- authorized_keys → Who can access the server? (Client keys)
- known_hosts → Is this the real server? (Server keys)


# Why `authorized_keys` Needs `chmod 600` Permissions

The `~/.ssh/authorized_keys` file must have strict permissions (`600`) for security reasons. Here's why:

---

## 1. What `chmod 600` Means
- **`6` (Owner)**: Read (`4`) + Write (`2`) = `6`  
- **`0` (Group)**: No permissions  
- **`0` (Others)**: No permissions  

This means:  
✅ **Only the file owner** (you) can read or modify it.  
❌ **No other user or process** (even in the same group) can access it.

---

## 2. Why Strict Permissions Are Required
### a) SSH Server Enforces Strict Checks
- The SSH daemon (`sshd`) **refuses to use `authorized_keys` if permissions are too loose** (e.g., `644`).  
- You'll see errors like:  
  `Permission denied (publickey).`  
  `Bad owner or permissions on ~/.ssh/authorized_keys.`

### b) Prevents Unauthorized Modifications
- If **group/others** have write (`622`, `666`, etc.), an attacker could:  
  - Add their own public key → Gain access to your account  
  - Delete existing keys → Lock you out  

### c) Blocks Information Leaks
- If readable by others (`604`, `644`), malicious users could:  
  - Steal the list of authorized public keys  
  - Use them for targeted attacks  

---

## 3. How to Fix Permissions
`chmod 600 ~/.ssh/authorized_keys`  
`chown $USER:$USER ~/.ssh/authorized_keys`  
*(Replace `$USER` with your username)*

---


## 4. What Happens If Permissions Are Wrong?
- SSH will ignore `authorized_keys`, falling back to:  
  - Password authentication (if enabled)  
  - Or deny access entirely  
- Example error in logs:  
  `Authentication refused: bad ownership or modes for authorized_keys`

### Key Takeaway  
`chmod 600` ensures **only you** can modify `authorized_keys`, preventing unauthorized SSH access. This is a critical security requirement enforced by SSH. 🔒
