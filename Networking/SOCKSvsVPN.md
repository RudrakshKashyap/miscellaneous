# Proxies vs VPN

# My understanding

> The main reason why socks need domain/ip in header is bc it operates at session layer and the packet doesn't have ip header already attached(so the proxy server has to build the L3 header on it's own end) while in vpn the ip address header is already attached(after decapsulation).

## Socks vs Other proxies

The fundamental difference lies in **where** they operate on the OSI model.

### 1. HTTP Proxies (Layer 7 - Application)

HTTP proxies are "high-level." They understand the protocol being used (HTTP/HTTPS). They can read the data, cache web pages, and even filter content (like blocking specific URLs).

- **Limitation:** They only work for web traffic. You can’t easily use an HTTP proxy for gaming, VOIP, or torrenting.

### 2. SOCKS Proxies (Layer 5 - Session)

SOCKS operates at a lower level than HTTP but higher than the raw "pipes" of Layer 4. Because it sits at the **Session Layer**, it acts as a circuit-level gateway.

- **Versatility:** It doesn't interpret the traffic. Whether it’s an email (SMTP), a file transfer (FTP), or a website (HTTP), SOCKS just passes the "stream" of data through.

---

## SOCKS5: The Modern Standard

When people talk about SOCKS today, they usually mean **SOCKS5**. It added three critical features that older versions (like SOCKS4) lacked:

- **UDP Support:** Unlike HTTP proxies which are TCP-only, SOCKS5 handles UDP. This is essential for **streaming, gaming, and DNS lookups**.
- **Authentication:** It allows you to require a username and password to use the proxy.
- **Remote DNS:** It can resolve domain names on the proxy server side, which prevents "DNS leaks" (where your ISP can see what sites you're visiting even if you're on a proxy).

---

## 1. The SOCKS5 Protocol Stack

In a SOCKS5 proxy connection, the "SOCKS" layer sits between the Transport (TCP) and the Application (HTTP) layers.

### The Handshake (The "Control" Phase)

Before data flows, a **TCP connection** is established between your Client and the VPS. The Client then sends a SOCKS5 request.
**Raw Data Example (Hex):** `05 01 00 03 0a 67 6f 6f 67 6c 65 2e 63 6f 6d 01 bb`

- `05`: SOCKS Version 5
- `01`: Connect command
- `03`: Address type (Domain Name)
- `0a`: Length of domain (10 characters)
- `67 6f 6f 67 6c 65 2e 63 6f 6d`: "google.com" in ASCII
- `01 bb`: Port 443 (HTTPS)

### The Data Phase (The "Tunnel" Phase)

Once the VPS connects to Google, the stack for a single packet looks like this:

| Layer        | Protocol | Content                                                   |
| ------------ | -------- | --------------------------------------------------------- |
| **Ethernet** | Layer 2  | MAC Addresses (Local Gateway)                             |
| **IP**       | Layer 3  | Source: Your IP, Destination: VPS IP                      |
| **TCP**      | Layer 4  | Source Port: 54321, Destination Port: 1080                |
| **SOCKS5**   | Layer 5  | _No header during data phase (it's a transparent stream)_ |
| **TLS/SSL**  | Layer 6  | Encrypted Handshake/Application Data                      |
| **HTTP**     | Layer 7  | `GET /search...` (Encrypted inside TLS)                   |

---

## 2. Is the SOCKS Header Plain Text?

**Yes.** Standard SOCKS5 is **not encrypted**.

- **Visibility:** Your ISP can see the SOCKS5 handshake. They can see the hex code for "google.com" and the destination port.
- **The "Wait":** While they see the _destination_ (Google), they cannot see the _content_ of the data if you are using HTTPS, because the TLS encryption happens at Layer 6, inside the SOCKS stream.

---

## 3. What is SOCKS5h? (DNS Leaks)

"SOCKS5h" is not a separate protocol, but a **configuration flag** (common in `curl` or browser settings) regarding **DNS Resolution**.

- **Standard SOCKS5:** Your computer asks your ISP's DNS: _"What is the IP for google.com?"_ Then it tells the proxy: _"Connect me to 142.250.190.46."_ \* **The Leak:** Your ISP sees you looking up Google.
- **SOCKS5h (Remote DNS):** Your computer does **not** do a DNS lookup. It tells the proxy: _"Connect me to the domain 'google.com'."_ \* **The Benefit:** The VPS does the DNS lookup. Your ISP never sees the DNS query.

### Is it default?

**No.** Most applications (like Firefox or Chrome) have a specific checkbox: _"Proxy DNS when using SOCKS v5"_. If you don't check it, you are leaking your browsing history to your ISP via DNS queries, even if the traffic goes through the VPS.

### How the Raw Data differs

When your client sends the request to the VPS, the SOCKS5 "Connect" command changes based on whether you've already done the DNS work or not:

**Option A: You send the IP (Client-side DNS)**
`05 01 00 01 [4 bytes of IP] [2 bytes of Port]`

- The `01` after the `00` tells the VPS: "I am giving you an **IPv4 address**."

**Option B: You send the Domain (SOCKS5h / Remote DNS)**
`05 01 00 03 [Length] [Domain String] [2 bytes of Port]`

- The `03` tells the VPS: "I am giving you a **Domain Name**. You resolve it."

---

# The VPN Protocol Stack

A VPN operates at a lower level (Layer 3). It creates a **Virtual Network Interface**. Instead of just wrapping the data, it wraps the **entire IP packet**.

| Layer          | Protocol | Content                                                                             |
| -------------- | -------- | ----------------------------------------------------------------------------------- |
| **Ethernet**   | Layer 2  | MAC Addresses                                                                       |
| **Outer IP**   | Layer 3  | Source: Your IP, Destination: VPN Server IP                                         |
| **UDP/TCP**    | Layer 4  | VPN Tunnel Protocol (e.g., OpenVPN Port 1194), Destinaton Port: 51820 for WireGuard |
| **VPN Header** | -        | Encryption/Authentication tags                                                      |
| **Inner IP**   | Layer 3  | Source: **VPN Internal IP**(10.8.0.2, assigned by VPN), Destination: Google IP      |
| **TCP**        | Layer 4  | Source Port: 54321, Destination Port: 443                                           |
| **TLS/HTTP**   | Layer 7  | Encrypted Web Data                                                                  |

**The Difference:** In SOCKS, the packet is addressed to the VPS. In a VPN, the packet is addressed to Google, but that entire packet is stuffed inside _another_ packet addressed to the VPN server.

---

In a VPN stack, there are actually **three** important IP addresses involved:

1. **The Outer IP (Public):** This is your real ISP IP and your VPS/VPN Public IP. This gets the packet from your house to the VPS.
2. **The Destination IP:** This is **Google.com’s IP** (e.g., `142.250.190.46`). This is sitting inside the encrypted tunnel.
3. **The Virtual/Internal IP:** This is the IP your VPN server assigned to you (usually something like `10.8.0.2`).

### 1. The "Inner" Packet (The Original Intent)

Before the VPN software touches anything, your OS generates a standard packet intended for Google.

- **Layer 7 (Application):** Your HTTPS data (Encrypted).
- **Layer 4 (Transport):** TCP Header (Source Port: 54321, Dest Port: 443).
- **Layer 3 (Network):** **Inner IP Header**.
- **Source IP:** Your **Virtual VPN IP** (e.g., `10.8.0.2`).
- **Destination IP:** **Google’s Real IP** (e.g., `142.250.190.46`).
When you toggle the VPN "On," your phone creates a virtual network interface (like a fake Wi-Fi card). The OS assigns that interface a private IP (e.g., `10.8.0.2`). When you try to go to Google, the OS routing table says, "Send this through the VPN interface," which triggers the client to build that inner packet with the `10.8.0.2` source address.

> **Note:** This packet is "complete," but if it were sent to the internet like this, it would fail because your ISP doesn't know how to route the `10.8.0.x` private range. Since many phones can connect to a single VPN server, the VPN server assigns a unique private IP to each connected device, and note it down in it's NAT table.

---

### 2. The VPN Transformation (The "Wrap")

The VPN client intercepts this "Inner" packet and treats the **entire thing** as raw data (payload). It then wraps it in the VPN Protocol's own headers.

- **VPN Header:** (WireGuard, OpenVPN, or IPsec). This contains a **Security Association (SA)** or Session ID and a **Message Authentication Code (MAC)** to ensure the packet hasn't been tampered with.
- **Encryption:** The entire "Inner" packet (including the Google IP header) is encrypted. To your ISP, it looks like high-entropy noise.

---

### 3. The "Outer" Packet (The Carrier)

To get this encrypted "blob" to your VPS, the VPN adds a final set of headers that are "routable" on the public internet.

- **Layer 4 (Transport):** Usually **UDP** (for speed). The destination port is your VPN server port (e.g., 51820 for WireGuard).
- **Layer 3 (Network):** **Outer IP Header**.
- **Source IP:** Your **Real ISP IP** (e.g., `203.0.113.5`).
- **Destination IP:** Your **VPS Public IP** (e.g., `45.33.22.11`).

<br />
<br />
<br />

# SSH Dynamic Port Forwarding (SOCKS Proxy)

Using SSH as a SOCKS proxy—often called **SSH Dynamic Port Forwarding**—is a clever, lightweight way to tunnel your web traffic through a secure server. It’s essentially a "poor man’s VPN" that encrypts your browser traffic and masks your IP address without needing extra software.

---

## How It Works

When you run the command, your local machine opens a port that acts as a SOCKS proxy server. Any application configured to use this port will send its data through the encrypted SSH "tunnel" to your remote server, which then fetches the data from the internet on your behalf.

### Command

Open your terminal and run:

```bash
ssh -D 8080 -f -C -q -N user@remote_host
ssh -D 12345 ubuntu@92.4.65.48 -i UbuntuMan-private-ssh-key-2026-02-21.key
```

**What these flags do:**

- **`-D 8080`**: Opens a "Dynamic" port(the destination port can be anything) forwarding on port `8080` (or you can pick any unused port).
- **`-f`**: Requests SSH to go to the background just before command execution.
- **`-C`**: Compresses the data (great for slower connections).
- **`-q`**: Quiet mode (suppresses warnings/messages).
- **`-N`**: Tells SSH not to execute a remote command (you only want the tunnel, not a shell).

---

## Setting Up Your Browser

Once the tunnel is active, your computer is ready, but your apps don't know it yet. You need to point them to `localhost:8080`.

### For Chrome/System-wide

Chrome usually uses your OS system settings.

- **macOS:** System Settings > Network > Wi-Fi/Ethernet > Details > Proxies > SOCKS Proxy.
- **Windows:** Settings > Network & Internet > Proxy > Manual proxy setup.

---

## Why Use This?

- **Security on Public Wi-Fi:** Encrypts your unencrypted traffic.
- **Bypass Firewalls:** If your work or school blocks a site, but allows SSH, you're in.
- **IP Masking:** Websites will see the IP address of your remote server, not your home/local IP.

> **Note:** This only proxies the traffic for the specific apps you configure or apps that use the system proxy settings. Other apps on your computer will still use your regular connection unless you use a tool like `proxychains`.

## [Wireguard](https://youtu.be/88GyLoZbDNw?si=K7GF045oSHosjgNF)
