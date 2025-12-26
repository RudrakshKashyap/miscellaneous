# [DNS](https://youtu.be/27r4Bzuj5NQ?si=7rKZOof-6e1YfFuR&t=17)(Domain Name Server)

1. **Recursive Resolver (DNS Resolver)**  
   - Queries authoritative servers on behalf of the client.
   - Caches responses to reduce lookup time.
   - Example: ISP’s DNS server, Google DNS (`8.8.8.8`), Cloudflare (`1.1.1.1`).

2. **Root Name Server**  
   - Directs queries to TLD servers.
   - There are **13 root server clusters** worldwide.

3. **TLD(Top Level Domain) Name Server**  
   - Manages domain extensions (e.g., `.com`, `.org`).
   - Directs queries to authoritative name servers.

4. **Authoritative Name Server**  
   - Contains the actual DNS records for a domain.
   - Responds with the final IP address.(www.example.com → 192.0.2.1)

```bash
Your App → `127.0.0.53` (local resolver) → `8.8.8.8` (Google DNS or your local DNS server) → Root Servers → TLD → Authoritative DNS
```

![](https://www.indusface.com/wp-content/uploads/2024/10/DNS-lookup-process-.png)



```bash
rudraksh @🔥:~$nslookup iitbhilai.ac.in
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	iitbhilai.ac.in
Address: 103.147.138.100



rudraksh @🔥:~$dig +trace google.com

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> +trace google.com
;; global options: +cmd
.			4451	IN	NS	i.root-servers.net.
.			4451	IN	NS	g.root-servers.net.
.			4451	IN	NS	d.root-servers.net.
.			4451	IN	NS	b.root-servers.net.
.			4451	IN	NS	h.root-servers.net.
.			4451	IN	NS	c.root-servers.net.
.			4451	IN	NS	f.root-servers.net.
.			4451	IN	NS	k.root-servers.net.
.			4451	IN	NS	e.root-servers.net.
.			4451	IN	NS	j.root-servers.net.
.			4451	IN	NS	a.root-servers.net.
.			4451	IN	NS	l.root-servers.net.
.			4451	IN	NS	m.root-servers.net.
;; Received 239 bytes from 127.0.0.53#53(127.0.0.53) in 1 ms

com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			86400	IN	DS	19718 13 2 8ACBB0CD28F41250A80A491389424D341522D946B0DA0C0291F2D3D7 71D7805A
com.			86400	IN	RRSIG	DS 8 1 86400 20250621050000 20250608040000 53148 . OOfgwxobZgAijnmOUJ9RpfdRyl5HTlrKcn/bj6SR1ebAYL9bSxvShbjE 4GBxIX5QR/eASwkp8hXJFDHu9R+V0IEiqJ7PpRJ4Q/Vbs8TC4BPyaY/i WVGQ4ndtYKzm8UVOrrqSZy3Xlz1xBT1hV0grHZDNFWFBLF9OPcS7yXUt KJeRu0qB8LL7w99os1+/iAQVlIWSF7mcfSWgmQyoNfEubGzXhaEi6NgY mOBpu6c+eTotenv2O/sDLrrb7DKhCkOVHJSzFkVgqbMc190Rn2L6NdDY XKr6Qa7sysHhyhI2YIzNJxZmyNqu4+pZP5J6I13Www93paR7XWQODels DvRVwA==
;; Received 1198 bytes from 192.36.148.17#53(i.root-servers.net) in 40 ms

google.com.		172800	IN	NS	ns2.google.com.
google.com.		172800	IN	NS	ns1.google.com.
google.com.		172800	IN	NS	ns3.google.com.
google.com.		172800	IN	NS	ns4.google.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN NSEC3 1 1 0 - CK0Q3UDG8CEKKAE7RUKPGCT1DVSSH8LL NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN RRSIG NSEC3 13 2 900 20250612002557 20250604231557 40097 com. aQvdbofGBXZnILmqyixzz6sdMQYlY2FVg26Rwq5LXhTk8r4DOqZ7t/Cj AKVDYCGLTJA5MHTIuQ4MNJ7A91rwTA==
S84BOR4DK28HNHPLC218O483VOOOD5D8.com. 900 IN NSEC3 1 1 0 - S84BR9CIB2A20L3ETR1M2415ENPP99L8 NS DS RRSIG
S84BOR4DK28HNHPLC218O483VOOOD5D8.com. 900 IN RRSIG NSEC3 13 2 900 20250613013039 20250606002039 40097 com. Ho5HztEGQJ7K7y3JRPxSaZpUI1FeeIi3Uh1YoQn5MettNffL/EN/xUud g08cjRvXdG45cEZDn6DDjVHkTwjVGQ==
;; Received 644 bytes from 2001:502:1ca1::30#53(e.gtld-servers.net) in 178 ms

google.com.		300	IN	A	142.251.42.110
;; Received 55 bytes from 2001:4860:4802:36::a#53(ns3.google.com) in 140 ms

```

`dig +trace google.com` # only use to see the path (root → TLD → authoritative)

Suppose:  
- The `.com` TLD server (`a.gtld-servers.net`) has `google.com`’s nameservers cached.  
- Normally, it could return the `A` record directly (if asked recursively).  

But with `dig +trace`:  
1. The query is **non-recursive** (`RD` flag disabled).  
2. The TLD server **must return `NS` records** (not the `A` record).  
3. So the trace **continues until the authoritative server** (`ns1.google.com`).  

### **Why Does `127.0.0.53` Respond with Root Hints?**
This is a special case
   - `systemd-resolved` (listening on `127.0.0.53`) **includes a built-in list of root server IPs** (called "root hints").
   - When it sees a non-recursive query for the root zone (`.`), it returns this **preconfigured list** instead of forwarding the query upstream(Local DNS).
     - This is part of the DNS protocol: all resolvers (even local ones) must know the root servers to bootstrap resolution.
   - The response you see is **not coming from an upstream server**—it’s hardcoded in `systemd-resolved`.
    
## Use `nmcli` (NetworkManager on Linux)
Since **dig +trace** starts by querying **127.0.0.53** (your system’s stub resolver), it won’t reveal the **real upstream DNS server** (e.g., 8.8.8.8, 1.1.1.1, or your ISP/router’s DNS). 

Here’s how to find it:
```bash
rudraksh @🔥:~$nmcli dev show | grep DNS
IP4.DNS[1]:   192.168.29.1
IP6.DNS[1]:   2405:201:303e:d849::c0a8:1d01
```

Lists DNS servers assigned via DHCP or manual config.

## **DNS-over-HTTPS (DoH)**
**DOH** is a security protocol that encrypts DNS queries using **HTTPS** (the same protocol used for secure web browsing). It prevents eavesdropping and manipulation of DNS traffic by third parties (e.g., ISPs, hackers, governments).

- Resistant to **DNS spoofing, censorship, and surveillance**.
- Supported by major browsers (Firefox, Chrome, Edge) and DNS providers (Cloudflare, Google).

### **DoH Lookup Process:**
1. **Client (Browser/OS)** sends a DNS query (e.g., `example.com`).
2. Instead of using standard **DNS (UDP 53)**, the request is sent via **HTTPS (port 443)** to a **DoH-compatible resolver** (e.g., Cloudflare `1.1.1.1`).
3. The DoH resolver processes the query and returns the encrypted response.
4. The client decrypts the response and connects to the resolved IP.


## **How to Enable DoH?**
### **A. In Web Browsers**
#### **Firefox**
1. Go to `about:config`.
2. Search for `network.trr.mode`.
3. Set to `2` (enable DoH by default).
4. Set `network.trr.uri` to a DoH provider (e.g., `https://cloudflare-dns.com/dns-query`).

#### **Chrome/Edge**
1. Go to `Settings > Privacy and Security > Security`.
2. Enable **"Use secure DNS"** and select a provider (e.g., Cloudflare, Google).

### **B. At OS Level (Windows 11)**
1. Go to **Settings > Network & Internet > Ethernet/Wi-Fi**.
2. Click **"Edit DNS Settings"** → **"Encrypted Only (DNS-over-HTTPS)"**.
3. Enter a DoH server (e.g., `https://dns.google/dns-query`).

### **C. Using Public DoH Resolvers(free of cost)**
| Provider | DoH Server URL |
|----------|----------------|
| **Cloudflare** | `https://cloudflare-dns.com/dns-query` |
| **Google** | `https://dns.google/dns-query` |
| **Quad9** | `https://dns.quad9.net/dns-query` |
| **NextDNS** | `https://dns.nextdns.io/your-id` |
---

## **DNS-over-TLS (DoT)**
| Factor | DoT (DNS-over-TLS) | DoH (DNS-over-HTTPS) |
|--------|---------------------|-----------------------|
| **Port** | Dedicated (853) | Shared (443) |
| **Traffic Pattern** | Clearly DNS | Looks like normal HTTPS |
| **Blocking Method** | Simple port block | Needs DPI(Deep Packet Inspection)/ML |

---


<br />
<br />
<br />


## [CNAME](https://en.wikipedia.org/wiki/CNAME_record) - [The canonical (true) name](https://www.youtube.com/watch?v=ZXCQwdVgDno)