| Feature     | Local Storage                                     | Session Storage                                        |
| ----------- | ------------------------------------------------- | ------------------------------------------------------ |
| Persistence | Persists across browser sessions (until cleared)  | Cleared when the browser tab/window is closed          |
| Scope       | Accessible across all tabs/windows of same origin | Isolated to the specific tab/window where created      |
| Use Cases   | Long-term data, user preferences, cached data     | Temporary, session-specific data (e.g., shopping cart) |
| Capacity    | Typically around 5-10 MB                          | Typically around 5 MB                                  |

### Types of Cookies

Cookies are small text files used to store data. They are generally categorized by their **lifespan** and **attributes**.

```http
HTTP/1.1 200 OK
Set-Cookie: theme=dark
Set-Cookie: user_id=12345; Secure; HttpOnly
```

- **Session Cookies:**
  - **Lifespan:** Temporary. They are deleted immediately when you close your browser.
  - **Use Case:** Shopping carts, keeping you logged in while navigating between pages.
  - **Storage:** Typically stored in the computer's **RAM** (memory), not written to the hard drive.
- **Persistent Cookies:**
  - **Lifespan:** Long-term. They have a set expiration date (e.g., "Expires in 30 days"). They survive browser restarts.
  - **Use Case:** "Remember me" login features, language preferences, tracking user behavior.
  - **Storage:** Written to the **Hard Drive/SSD**.
- **Secure Cookies:** Can only be transmitted over encrypted (HTTPS) connections.
- **HttpOnly Cookies:** **Crucial for security.** These cookies _cannot_ be accessed by JavaScript (e.g., `document.cookie`). They are only sent to the server. **If you try to set the cookie from the frontend using JavaScript, you cannot apply the HttpOnly flag.**
- **SameSite Cookies:** Controls whether cookies are sent with cross-site requests (e.g., clicking a link from Facebook to your site). This prevents CSRF (Cross-Site Request Forgery) attacks.
  - `Strict`: Sent only for first-party requests. Cookies are only sent with requests originating from the same site, providing the strongest protection.
  - `Lax`(relaxed ): is fundamentally designed to block cookies on cross-site requests that use "unsafe" methods like POST, meaning it primarily works on "safe" methods like GET. The request must result in a change of the URL in the browser's address bar (e.g., clicking a link).
  - `None`: Sent with all requests (requires `Secure` flag). Avoid it.
- **`Partitioned` flag (CHIPS):** This flag prevents cross-site tracking while still allowing legitimate third-party services (like embedded chat widgets or payment gateways) to function using cookies.
  - **The Problem:** Traditionally, if Site A embedded an "Ad" widget, the "Ad" site could set a cookie (`SameSite=None`). If you then visited Site B, which also embedded the "Ad" widget, the browser would send that *same* cookie to the "Ad" site. This allowed the "Ad" site to track you across both websites. Because of this privacy violation, modern browsers are blocking unpartitioned third-party cookies.
  - **The Solution:** When a cookie is marked as `Partitioned`, its storage is "sandboxed" or tied to the top-level site you are currently visiting.
  - **How it works (Example):**
    1. You visit Site A. Site A embeds an "Ad" widget. The "Ad" site sets a `Partitioned` cookie. The browser stores this in a specific partition: `(Site A + Ad Site)`.
    2. You then visit Site B. Site B also embeds the "Ad" widget.
    3. The browser will **not** send the cookie created on Site A to the "Ad" site, because the browser looks in the `(Site B + Ad Site)` partition, which is currently empty.
    4. The "Ad" widget on Site B gets a fresh state and cannot link your activity between Site A and Site B.

---

### What Cookies Can a Site Access?

A website **cannot** access all cookies on your computer. Access is strictly controlled by the **Same-Origin Policy** and cookie attributes.

1.  **The "Same-Origin" Rule:** A site (e.g., `google.com`) can only read cookies that were set by `google.com`. It cannot read cookies from `facebook.com`.
2.  **Domain Scope:** A cookie set for `.example.com` can be read by `subdomain.example.com`. However, a cookie set strictly for `app.example.com` cannot be read by `example.com`.
3.  **Path Scope:** If a cookie is set with `Path=/admin`, it cannot be read by pages at `/home`.
4.  **The HttpOnly Barrier:** Even if a cookie belongs to the _same_ site, if it is marked **HttpOnly**, the JavaScript running on that site **cannot see it**.

---

### 4. Refresh Token vs. Access Token & Local Storage

This is a critical security architecture question.

#### Is Local Storage unsafe?

**Yes, for sensitive data.**

- **Vulnerability:** Local Storage is accessible by _any_ JavaScript running on your page.
- **The Threat (XSS):** If your site has a Cross-Site Scripting (XSS) vulnerability (e.g., a hacker injects a malicious script via a comment section or a compromised third-party library), that script can read `localStorage` and send all your tokens to the hacker.
- **Can _every_ site see it?** **No.** Only code running on _your_ specific domain (origin) can see your Local Storage. `evil-site.com` cannot directly read `your-bank.com`'s local storage. But if `evil-site.com` manages to run a script _inside_ `your-bank.com` (XSS), then they can.

#### Are Refresh Tokens kept at a "safer place"?

**Ideally, Yes.**
The "Standard Safe Pattern" for storing tokens is:

1.  **Access Token:** Short-lived (e.g., 15 mins). Kept in **Application Memory (JavaScript variable)**.
    - _Why?_ If it's in memory, it vanishes if the user closes the tab. It is not written to disk.
2.  **Refresh Token:** Long-lived (e.g., 7 days). Kept in an **HttpOnly, Secure Cookie**.
    - _Why is this safer?_ Because it is **HttpOnly**, JavaScript cannot read it. Even if a hacker successfully runs a script on your page (XSS), they cannot "steal" the refresh token because the browser hides it from the code. The browser simply handles sending it to the API automatically.

#### Summary Comparison

| Feature                   | Local Storage                  | HttpOnly Cookie                  |
| :------------------------ | :----------------------------- | :------------------------------- |
| **Accessible by JS?**     | Yes (Vulnerable to XSS)        | **No** (Safe from XSS)           |
| **Accessible by Server?** | No (Must be manually sent)     | Yes (Sent automatically)         |
| **Main Risk**             | **XSS** (Theft of token)       | **CSRF** (Unwanted actions)\*    |
| **Best For**              | User preferences (theme, etc.) | **Refresh Tokens** / Session IDs |

_> **Note on CSRF:** HttpOnly cookies protect against theft, but because they are sent automatically, they can be abused for Cross-Site Request Forgery (CSRF). This is easily solved by using `SameSite=Strict` cookies or anti-CSRF tokens._

## Auth Token in Header(bearer) vs Cookies

If your frontend is at `myapp.com` and your API is at `api.com`, cookies can be tricky to configure (requiring `CORS` and `SameSite` tweaks). Headers work everywhere regardless of domain.

#### The "Best of Both Worlds" Strategy

Many modern apps use **Cookies for the Web** (to prevent theft) and **Bearer Tokens for Mobile** (for ease of use). NestJS makes this easy because you can write a "multi-extractor" strategy that checks for both:

```typescript
jwtFromRequest: ExtractJwt.fromExtractors([
  (req) => req?.cookies?.jwt, // Check cookies first
  ExtractJwt.fromAuthHeaderAsBearerToken(), // Fallback to header for mobile
]);
```
