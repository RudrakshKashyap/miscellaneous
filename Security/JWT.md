
<img src="https://cdn.sanity.io/images/3jwyzebk/production/c098fa07deca1062e013d92cabba4ba7ec7e7f19-1584x988.png?auto=format&fit=max&w=1920&q=75" width="50%" height="auto">

## Session vs JWT
- https://www.youtube.com/watch?v=GhrvZ5nUWNg
- https://www.youtube.com/watch?v=fyTxwIa-1U0

---

<br />
<br />
<br />

## Indrustry Standard

If you use a single **Long-Lived Access Token** (e.g., valid for 30 days), and a hacker steals it from your browser, they effectively **become you** for 30 days. You cannot stop them easily because the token is stateless—the server accepts it blindly until it expires.

Here is why the **Access + Refresh** combo is the industry standard:

---

### 1. The Security "Checkpoint"
The fundamental problem with JWT Access Tokens is that they are **stateless**. Once the server issues one, it cannot "cancel" it (without complex blacklisting).

* **Scenario A: Long-Lived Access Token (The Risk)**
    * You issue a token valid for 7 days.
    * A hacker steals it via an XSS attack.
    * **Result:** The hacker has full access to your bank account for 7 days. Even if you change your password, the hacker's token still works because the token itself says "I am valid for 7 days."

* **Scenario B: Short Access + Long Refresh (The Solution)**
    * **Access Token:** Valid for **15 minutes**.
    * **Refresh Token:** Valid for **7 days**.
    * **The Flow:** The user uses the Access Token. After 15 minutes, it dies. The app silently sends the **Refresh Token** to the server to ask: *"Can I have a new Access Token, please?"*
    * **The Checkpoint:** This moment—the "refresh" request—is the **only time** the server checks its database.
    * **Security Win:** If you banned the user or they changed their password 5 minutes ago, the server sees this during the refresh request and **denies** the new token. The hacker is locked out immediately after the 15-minute window closes.



### 2. Safer Storage Options
Because the Access Token is needed for every single API request, it is often kept in JavaScript memory (which is vulnerable to XSS) or a standard cookie.

However, the **Refresh Token** is only used once every 15–30 minutes. This allows you to store it more securely:
* You can put the Refresh Token in an **`HttpOnly` Cookie** with a specific path (`/refresh-token`).
* This means JavaScript cannot touch it. Even if a hacker injects code into your site, they can steal the Access Token (valid for only 15 mins) but they **cannot steal the Refresh Token**.
* Once the short Access Token expires, the hacker has nothing. They cannot renew it.

### 3. Refresh Token Rotation (Advanced Security)
To make this even tighter, banks and secure apps use **Token Rotation**(generally by changing the date field in token).
* Every time you use a Refresh Token to get a new Access Token, the server **also issues a new Refresh Token** and invalidates the old one.
* If a hacker manages to steal your Refresh Token and tries to use it, the server sees that an "old" Refresh Token is being used.
* The server concludes: *"This is theft."* It immediately invalidates **all** tokens for that user, forcing everyone to log in again.

### Summary Comparison

| Feature | **Long-Lived Access Token** | **Short Access + Refresh Token** |
| :--- | :--- | :--- |
| **If stolen...** | Hacker has access for days/weeks. | Hacker has access for minutes. |
| **Revoking access** | Very difficult (requires waiting for expiry). | Fast (happens at the next "refresh" interval). |
| **Server Load** | Very Low (No DB checks). | Low (DB check only occurs once every 15-30 mins). |
| **User Experience** | User stays logged in. | User stays logged in (refresh happens in background). |

---
<br/>

## Why not use single short lived access token as secure cookie

They can look similar, but they make different trade-offs. The two-token model (short-lived access token + long-lived refresh token) is the industry standard because it **cleanly separates scopes**, API access, transport semantics, revocation, and compatibility with resource servers.

In security, we want to limit where sensitive credentials are sent.

**Two-token**: recommended pattern is to keep refresh in HttpOnly cookie and put access token in memory and send it in Authorization header — this avoids CSRF on API calls because browsers don’t auto-send Authorization headers. But access token stored in JS is vulnerable to XSS unless you keep it in memory only and clear on navigation.

* **Standard Method:**
    * **Access Token:** Sent to *every* endpoint (Product Service, Comment Service, Image Service, Logging Service). If one of these services logs the header or is compromised, the attacker gets a token that only lasts 15 minutes.
    * **Refresh Token:** Sent **only** to the `/auth/refresh` endpoint. It is never seen by the other parts of your application.

* **Your Proposal:**
    * You are sending the token that has the power to "renew itself" (the equivalent of a Refresh Token) to *every single endpoint* in your system.
    * If your "Image Upload Service" has a vulnerability or aggressive logging, and it leaks your single token, the attacker can use that token to keep generating new tokens indefinitely (until your max session limit).

<br >
<br >
<br >

# [OAuth 2.0](https://www.youtube.com/watch?v=fX5U50VGxtg): Implicit, Authorization Code, and [PKCE](https://www.youtube.com/shorts/hRjAEblltjc)

Core Security Problem: [The Browser is a Leaky Place](https://www.youtube.com/watch?v=Gtbm5Fut-j8) (bold lines) ->

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*XVA6MV-nQ0pEeq6ceoA-XQ.png" width="60%" height="auto">

**OAuth 2.0** (Open Authorization) is an **authorization framework** that allows a third-party application (the **Client**) to obtain **limited access** to an HTTP service (the **Resource Server**) on behalf of a user (the **Resource Owner**), without the Client ever needing the user's credentials.

### Key OAuth 2.0 Roles

* **Resource Owner:** The user who owns the protected resources (data) and can grant access.
* **Client:** The application (e.g., a mobile app, web app) that wants to access the Resource Owner's protected resources. It must be registered with the Authorization Server.
* **Authorization Server:** The server that authenticates the Resource Owner and, upon consent, issues an **Access Token** to the Client.
* **Resource Server:** The server that hosts the protected resources (APIs) and accepts Access Tokens from the Client to grant access.

---

## 🔁 Authorization Code Flow (Recommended Flow)

The Authorization Code flow is considered the most secure and is the standard for confidential clients (like server-side web applications) and public clients (like Single-Page Applications and Mobile Apps) when paired with PKCE.



### Steps in the Authorization Code Flow

1.  **Client Redirects User:** The Client application directs the user's browser (User Agent) to the **Authorization Endpoint** of the Authorization Server. This request includes the `client_id`, `redirect_uri`, requested `scope`, `response_type=code`, and a random `state` parameter.
2.  **Resource Owner Authenticates/Consents:** The Authorization Server authenticates the Resource Owner (logs them in) and asks them to grant or deny the Client the requested access (`scope`).
3.  **Authorization Server Redirects with Code:** If consent is granted, the Authorization Server redirects the user's browser back to the Client's pre-registered **`redirect_uri`**, including a one-time-use **Authorization Code** and the original `state` parameter in the URL query string.
4.  **Client Exchanges Code for Token:** The Client application (from its backend server, securely) makes a direct **server-to-server** request to the Authorization Server's **Token Endpoint**. This request includes the Authorization Code, `client_id`, `redirect_uri`, and its **`client_secret`** (if it is a confidential client).
5.  **Authorization Server Issues Token:** The Authorization Server validates the code and the client credentials. If valid, it issues an **Access Token** (and optionally a **Refresh Token** and **ID Token** if using OpenID Connect) to the Client.
6.  **Client Accesses Resource:** The Client uses the Access Token to make requests to the **Resource Server** on behalf of the user.

---

## 🚫 Implicit Flow (Deprecated/Discouraged)

The Implicit flow was designed for browser-based, public clients (like SPAs) that couldn't securely store a client secret or perform a server-side code exchange. **It is now deprecated** due to inherent security risks. Modern SPAs and mobile apps use the Authorization Code flow with PKCE instead.

### Key Characteristics of Implicit Flow

* **Token in URL Fragment:** The Access Token is returned directly to the browser in the URL fragment (`#` part of the URL) after step 3, bypassing the token exchange step (Steps 4 & 5).
* **No Authorization Code:** The `response_type` is typically `token` (or `id_token token` for OIDC), meaning the token is issued immediately, not an authorization code.
* **Vulnerability:** Since the token is exposed in the browser's URL (history, referrer headers) and is handled by client-side JavaScript, it is highly susceptible to leakage and interception (e.g., via XSS attacks).
* **No Refresh Token:** It typically does not allow for a Refresh Token, forcing reliance on short-lived Access Tokens.

---

## 🔑 Proof Key for Code Exchange (PKCE, Pronounced "Pixy")

**PKCE** is an extension to the Authorization Code flow, originally designed to secure the flow for public clients (mobile apps, SPAs) that cannot securely store a `client_secret`. It is now recommended for **all** OAuth 2.0 clients.

### How PKCE Works

1.  **Client Generates Secrets:** The Client generates a high-entropy, cryptographically random string called the **`code_verifier`**. It then uses a transformation function (usually SHA256) on the `code_verifier` to create a **`code_challenge`**.
    $$code\_challenge = BASE64URL-ENCODE(SHA256(ASCII(code\_verifier)))$$
2.  **Authorization Request:** In Step 1 of the Authorization Code flow, the Client sends the **`code_challenge`** and the `code_challenge_method` (e.g., `S256`) to the Authorization Server.
3.  **Token Exchange:** In Step 4, when the Client exchanges the Authorization Code for an Access Token, it must send the original **`code_verifier`** along with the code.
4.  **Server Verification:** The Authorization Server recalculates the `code_challenge` using the received `code_verifier` and the stored `code_challenge_method`. If the newly calculated challenge **matches** the one it received in Step 2, the client is verified as the legitimate initiator of the request, and the token is issued.

### PKCE Benefits

* **Mitigates Authorization Code Interception:** If a malicious party intercepts the Authorization Code, they cannot exchange it for an Access Token because they do not have the secret `code_verifier`.
* **Secures Public Clients:** Allows public clients to use the secure Authorization Code flow without relying on a non-existent or easily compromised `client_secret`.

---
