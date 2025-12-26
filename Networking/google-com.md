## World wide web

Historically, the `www` in a web address (like `www.google.com`) was a **subdomain** used to tell the server which service you wanted. Just as `mail.google.com` takes you to email and `ftp.google.com` might have been used for file transfers, `www` was the standard way to say "I want the website."

In the 90s and early 2000s, www literally stood for the "World Wide Web server," while the naked domain was often used to point to the "Main Server" (which might have been used for email, FTP, or shell access instead of just web pages).

Even though it's optional for users, engineers still use it for a few technical reasons:

* **Cookie Control:** If a site uses `www.example.com`, it can keep its cookies restricted to just the website. If it uses the "naked" domain (`example.com`), those cookies might be sent to every other subdomain (like `mail.example.com`), which can cause security or performance issues.
* **Large-Scale Traffic (DNS):** For massive websites (like Facebook or Amazon), using `www` makes it easier to use a **CNAME record**. This allows them to quickly shift traffic to different servers if one goes down. It’s much harder to do this with a "naked" domain. <!-- TODO --> TODO - didnt quite understand this
* **Branding:** In some parts of the world or on physical advertisements (billboards), `www` is still a universal "symbol" that tells people "this is a website address."

<br />
<br />
<br />

---

When you type **google.com** into your browser's URL bar and press Enter, a complex sequence of events unfolds to deliver the webpage to your screen. Here's a detailed breakdown:

### 1. **URL Parsing & Scheme Resolution**
   - The browser parses the input to determine if it's a URL or a search term. Since "google.com" is a valid domain, the browser assumes HTTPS (default for most modern browsers).

### 2. **Decides HTTP/HTTPS protocol**
   - Checks the **HSTS(HTTP strict transport security) Preload List** to enforce HTTPS (Google is on this list, so HTTP is bypassed). (You need to manually apply for your domain to be eligible).

### 3. **DNS Lookup**
   - **Browser Cache**: Checks if the IP for google.com is cached.
   - **OS Cache**: If not, the OS checks its DNS cache.
   - **DNS Resolver** (e.g., ISP/Public DNS like 8.8.8.8):
     - Queries **Root DNS Servers** → directs to **.com TLD Servers** → directs to **Google’s Authoritative Name Servers**.
     - Returns the IP address(es) (e.g., 142.250.190.78 for IPv4 or 2607:f8b0:4005:80a::200e for IPv6).

### 4. **TCP Connection**
   - Initiates a **TCP 3-Way Handshake** (SYN → SYN-ACK → ACK) with the server’s IP on port 443 (HTTPS).

### 5. **TLS Handshake**
   - Negotiates encryption using **TLS 1.3** (or 1.2):
     - **ClientHello**: Browser sends supported cipher suites.
     - **ServerHello**: Server selects cipher suite and sends its SSL certificate (issued by a trusted CA).
     - **Certificate Validation**: Browser verifies the certificate’s authenticity and checks for revocation (OCSP/CRL).
     - **Key Exchange**: Session keys are generated for symmetric encryption.

### 6. **HTTP Request/Response**
   - **HTTP/2 or HTTP/3 (QUIC)** is negotiated for efficient resource loading.
   - Browser sends an encrypted **HTTP GET** request for the root path (`/`).
   - Google’s infrastructure (load balancers, CDNs) routes the request to the nearest server.
   - Server responds with **HTTP 200 OK** (or a redirect, e.g., to `www.google.com`), headers, and HTML content.

### 7. **Redirect Handling (Optional)**
   - If redirected (e.g., 301/302), the browser repeats steps 2–5 for the new URL.

### 8. **Resource Processing**
   - **Parsing HTML**: Browser constructs the **DOM** (Document Object Model).
   - **CSSOM**: Processes CSS to build the **CSS Object Model**.
   - **Render Tree**: Combines DOM and CSSOM to layout elements.
   - **JavaScript Execution**: Blocks parsing unless marked `async`/`defer`. May modify DOM/CSSOM.
   - **Additional Requests**: Fetches images, fonts, scripts, etc., via parallel connections (HTTP/2 multiplexing).

### 9. **Rendering**
   - **Layout (Reflow)**: Calculates position/size of elements.
   - **Paint**: Converts elements to pixels.
   - **Compositing**: Layers are combined for final display.

### 10. **Post-Load Actions**
   - **Analytics/Telemetry**: Scripts send usage data to Google.
   - **Service Workers**: Cache static assets for offline use (if registered).
   - **WebSocket/HTTP/2 Push**: Optional real-time updates or preloaded content.

### 11. **Network Optimization**
   - **TCP Fast Open**: Reuse prior TLS sessions for faster subsequent connections.
   - **QUIC (HTTP/3)**: Uses UDP to reduce latency if supported.

### 12. **Security & Privacy**
   - **Cookie Handling**: Existing cookies are sent; new ones may be set (e.g., preferences, session ID).
   - **Content Security Policy (CSP)**: Enforces allowed resource sources.
   - **Cross-Origin Checks**: Verify CORS headers for external resources.

### 13. **Final Page Load**
   - `DOMContentLoaded` event fires when HTML/CSS are parsed.
   - `load` event fires when all resources (images, scripts) finish loading.
   - Page becomes interactive (**TTI: Time to Interactive**).

### Underlying Network Layers
- **ARP/Routing**: Local network finds MAC addresses for routers (if needed).
- **OS Routing Table**: Determines the path to send packets (IPv4/IPv6).
- **Firewalls/Proxies**: Enterprise networks may intercept and inspect traffic.

### Server-Side Infrastructure
- **Load Balancers**: Distribute traffic across global servers.
- **CDNs**: Cache static content (e.g., images, scripts) closer to users.
- **Edge Computing**: Execute logic at the edge (e.g., Cloudflare Workers).

### Advanced Protocols
- **HTTP/3 (QUIC)**: Uses UDP for faster connection setup, avoiding TCP head-of-line blocking.
- **Brotli Compression**: Compresses text resources (HTML/CSS/JS) for faster transfer.

### Visual Flowchart
```
User Input → URL Parsing → HSTS Check → DNS → TCP → TLS → HTTP → Server → Response → Parse → Render → Load → Interactive
```

Each step involves intricate systems working in milliseconds to deliver the seamless experience of accessing Google.com.