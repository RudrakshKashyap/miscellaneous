
# Workspace

In modern web browsers (like Chrome, Edge, or Brave), the **Workspace** feature in DevTools is a powerful bridge between your browser and your local computer.

Normally, when you change a CSS style or edit JavaScript directly in DevTools, those changes disappear as soon as you refresh the page. **Workspace** allows you to "map" the files on your website to the actual source files on your computer, so any change you make in the browser is automatically saved to your hard drive.

---

## How it Works

When you set up a Workspace, DevTools stops treating the website's code as a temporary copy and starts treating it as a live editor for your project folder.

### The Core Benefits

* **Persistent Saving:** Edit CSS, HTML, or JavaScript in the **Sources** panel, and those changes persist after a page refresh.
* **Live Styling:** You can tweak margins, colors, and layouts in the **Elements** panel, and the updated CSS is saved directly into your local `.css` file.
* **No Context Switching:** You don't have to constantly jump back and forth between your code editor (like VS Code) and the browser. You can do both in one place.

## How to Set It Up

1. Open **DevTools** ( or ).
2. Go to the **Sources** tab.
3. On the left-hand pane, look for the **Filesystem** tab (you may need to click the `>>` icon).
4. Click **+ Add folder to workspace**.
5. Select your project folder and click **Allow** when the browser asks for permission.
6. DevTools will try to "map" the network files to your local files. If it doesn't do it automatically, right-click a file and select **Map to file system resource**.

---

## Important Note on Modern Frameworks

If you are using tools like **React, Vue, or Angular**, you are likely using a "build step" (like Webpack or Vite). In these cases, the code the browser sees is different from the code you write (JSX, TypeScript, etc.).

For Workspace to work with these frameworks, you need **Source Maps** enabled. This tells the browser exactly which line of your original source code corresponds to the "transformed" code it is running.


<br />
<br />
<br />

# [Source Maps](https://www.youtube.com/watch?v=9LKJ2pbrAlE)

A **Source Map** is a special file (usually ending in `.map`) that acts as a bridge between the code you write (Source) and the code the browser actually runs (Generated).

Modern web development often involves "transforming" code for efficiency. Browsers are great at running this transformed code, but humans are terrible at reading it. Source maps solve this by telling the browser: *"This garbled line of code in the browser actually refers to this specific line in your original source file."*

---

## Why are they necessary?

When you build a professional website, your code undergoes several changes:

1. **Minification:** Removing spaces and renaming variables (e.g., `calculateTotal` becomes `a`) to save file size.
2. **Transpilation:** Converting modern JavaScript (ES6+) or TypeScript into older versions of JavaScript that all browsers understand.
3. **Bundling:** Combining hundreds of small files into one or two large files to reduce server requests.

**The Problem:** If your app crashes, the browser might say the error is on "Line 1, Column 54,000." This is useless for debugging.
**The Solution:** With a source map, the browser can show the error on "Line 12 of `AuthService.ts`" instead.

---

## How it Works (Under the Hood)

A source map is a **JSON file** that contains a "map" of the relationship between the two versions of the code.

### A typical `.map` file contains:

* **Version:** The version of the source map spec (usually 3).
* **Sources:** A list of the original file paths (e.g., `index.ts`, `header.scss`).
* **Mappings:** A long, encoded string (using Base64 VLQ) that tells the browser exactly which character in the minified file corresponds to which line/column in the original.

### How the Browser Finds It

At the very bottom of a minified JavaScript or CSS file, you’ll often see a special comment:
`//# sourceMappingURL=main.js.map`

When DevTools is open, the browser sees this comment, downloads the `.map` file, and reconstructs your original file structure in the **Sources** tab.

---

## How to Generate Them

Most modern tools have source maps turned **on** by default in development mode and **off** in production (to keep file sizes small).

* **Vite:** `sourcemap: true` in `vite.config.js`.
* **Webpack:** `devtool: 'source-map'` in `webpack.config.js`.
* **Sass:** Use the `--source-map` flag when compiling.

- [What are source maps?](https://www.youtube.com/watch?v=FIYkjjFYvoI)
- [Part 2](https://www.youtube.com/watch?v=SkUcO4ML5U0)

---
<br />

By default, most source maps contain only the **mappings** (the directions on how to get from the minified code back to the original file) and the **file paths** of the original source code. However, they can also optionally contain the **entire original source code** itself.

### 1. The "Only Mappings" Approach

This is the standard behavior for many production environments. The source map acts as a "bridge" or a set of GPS coordinates.

* **What's inside:** A list of file names (`sources` array) and a complex string of encoded characters (`mappings` field).
* **How it works:** When you open DevTools, the browser reads the map, sees a file name like `App.tsx`, and then tries to fetch that file from your server to display it.
* **Requirement:** For this to work, you must actually upload your original source files to your web server so the browser can find them.

### 2. The "Entire Original Code" Approach

This is common in development or when you don't want to host your raw source files publicly.

* **What's inside:** An additional field called **`sourcesContent`**.
* **How it works:** This field is an array of strings, where each string is the **full text** of an original source file.
* **Benefit:** The browser doesn't need to fetch anything else. Everything required to reconstruct your original code is bundled inside the `.map` file.
* **Downside:** The source map file becomes significantly larger because it effectively contains a second copy of your entire codebase.

---

### Comparison of Fields

If you open a `.map` file (which is just a JSON file), you can see which version you are using:

| Field | Purpose |
| --- | --- |
| `sources` | List of original file paths (e.g., `["src/main.js", "src/utils.js"]`). |
| `mappings` | The data that connects a line/column in the minified file to the original. |
| **`sourcesContent`** | **(Optional)** The actual text/code of the files listed in `sources`. |

### Security Warning

If your source map includes `sourcesContent`, anyone who can download your `.map` file can read your **original, unminified source code**, including comments and private logic.



# Network Panel

Introduced in the early 2000s, **XHR(XMLHttpRequest)** was the foundation of "AJAX."
The **Fetch API** is the modern replacement for XHR. It was designed to be much simpler and more powerful.
It is **Promise-based**. This means you can use `.then()` or the much cleaner `async/await` syntax.

When you type a URL into your address bar and hit Enter, that is a **Document** request (also called a "navigation request"), not a Fetch or XHR request.
The very first request made to a server is to get the `index.html` (or whatever the entry page is).

Once the browser receives that first `document` (the HTML), it starts reading the code from top to bottom. As it finds specific tags, it triggers different **types** of requests:

* **`<link rel="stylesheet">`** → Type: `stylesheet`
* **`<script src="...">`** → Type: `script`
* **`<img src="...">`** → Type: `image`
* **`fetch()` or `new XMLHttpRequest()`** inside a script → Type: `fetch` or `xhr`

> Fetch - Data requested by the script after it started running.

* **Documents and Stylesheets** are high priority because the user can't see the page without them.
* **Images** are medium priority.
* **Fetch/XHR** requests are often given lower priority by the browser’s internal scheduler so they don't block the initial visual "paint" of the page.
