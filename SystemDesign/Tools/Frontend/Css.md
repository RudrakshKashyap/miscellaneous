# CSS Modules vs Normal CSS

Using standard `import './index.css'` behaves differently than `import styles from './index.module.css'`.

If you just `import './index.module.css'`, the styles won't apply properly because of:

- **Hashing**: The class `.crm-dashboard-v2-theme` inside the CSS gets transformed into a unique hash like `._crm-dashboard-v2-theme_z123x` at build time.

  ### Why it becomes global

  When you write `import './something.css';`, build tools like Vite or Webpack don't "tunnel" those styles specifically into that one component. Instead:
  1. They take all the CSS from that file.
  2. They inject it into a global `<style>` tag in the `<head>` of your website.
  3. Because it's in the `<head>`, those styles are now available to every single element on the page.

- **Mismatched Code**: In your `index.jsx`, you are still using the literal string `<div className='crm-dashboard-v2-theme'>`.

Instead, you need to import the styled object and dynamically apply the hashed class names:

```jsx
import themeStyles from './index.module.css';

<div className={themeStyles['crm-dashboard-v2-theme']}>
```

## `.notation` vs `[]` notation

This is one of the most common "gotchas" in React development with standard CSS files. When you import a CSS Module, it acts like a JavaScript object where keys are class names and values are generated hashed strings.

### 1. `.` (Dot) Notation

You can use dot notation **only if your CSS class name is a valid JavaScript identifier**. The class name cannot contain hyphens (`-`), spaces, or start with a number.

**Example:**
If your CSS class is `.container`:

```jsx
// ✅ This works perfectly
<div className={styles.container}>
```

### 2. `[]` (Bracket) Notation

You **must** use bracket notation when your CSS class name is **not** a valid JavaScript identifier, usually because it contains **hyphens** (like `.btn-primary`). If you try to use `.notation` on a hyphenated class, JavaScript thinks you are trying to subtract!

**Example:**
If your CSS class is `.btn-primary`:

```jsx
// ❌ THIS WILL FAIL or return undefined (JS reads: styles.btn minus primary)
<div className={styles.btn-primary}>

// ✅ THIS IS CORRECT (Exact string lookup)
<div className={styles['btn-primary']}>
```

---

# Tailwind v4

Your observation hits on the core architectural shift in Tailwind v4 regarding how it "sees" your project.

## The Shift in Discovery

| Feature         | Tailwind v3 (PostCSS-based)                               | Tailwind v4 (CSS-first / Oxide)                                        |
| --------------- | --------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Scanning**    | Scans files listed in `tailwind.config.js` content array. | Follows the document graph (imports and Vite's module graph).          |
| **Logic**       | JS-based scanner looks for strings in your source code.   | CSS engine follows `@import` links to build a registry of styles.      |
| **Entry Point** | Any CSS file processed by PostCSS.                        | A single (or multiple) CSS entry point using `@import "tailwindcss";`. |

## Why the CSS Graph is the new Source of Truth

In Tailwind v4, if you import `index.css` directly into `index.jsx` via Vite:

1. Vite sees the import and handles the CSS injection.
2. Tailwind's engine (Oxide) will generate the utility classes you use in your `index.jsx` (because it's hooked into Vite's module graph).
3. **BUT**, if you try to use `@apply` or Tailwind-specific directives inside that `index.css`, it might fail or behave unexpectedly if that file isn't "connected" to your main Tailwind entry point via an `@import` statement.

## The "v4 Way"

To make `LeadHistoryV2/index.css` "visible" to the Tailwind engine so you can use features like `@apply` or your theme's variables seamlessly, you should ideally import it in your main CSS file (`src/styles/tailwind.css`):

```css
/* src/styles/tailwind.css */
@import "tailwindcss";
@import "../pages/CRM Dashboard/LeadHistoryV2/index.css"; /* Now Tailwind v4 knows about it */
```

### Summary

**CSS Graph is the new source of truth**. By abandoning the `content: [...]` array in JS, Tailwind v4 is much faster, but it requires you to be more explicit about your CSS dependency tree. If you keep it as a standalone import in JS, you're essentially treating it as a "plain" CSS file that just happens to be on the page, rather than a "Tailwind-powered" CSS file.

**One nuance for Vite users**: If you're using `@tailwindcss/vite`, the plugin is smart enough to scan any file Vite touches, but the context (like your `@theme` values) is only shared if the files are part of the same CSS entry point.

<br />
<br />
<br />

```css
@layer theme, base, components, utilities;

/* 1. Import with prefix, NO preflight */
@import "tailwindcss/theme" layer(theme) prefix(tw);
@import "tailwindcss/utilities" layer(utilities) prefix(tw);

/* 2. Define variables OUTSIDE of @layer base (v4 standard) */
/* CSS Variables are evaluated at runtime by the browser. */
:root {
  --background: hsl(0 0% 100%);
  --foreground: hsl(0 0% 3.9%);
  --border: hsl(0 0% 89.8%);
  --ring: hsl(0 0% 3.9%);
  --radius: 0.5rem;
}

/* 3. Map variables so tw:bg-background works */
/* you dont exactly need to define it first in root */
/* but you can define it in @theme inline
to create a mapping and later you
can overwrite these variable values in custom.module.css
on runtime
 */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-border: var(--border);
  --color-ring: var(--ring);
  --radius-lg: var(--radius);
}

/* 4. Your custom Base styles */
@layer base {
  * {
    @apply tw:border-border tw:outline-ring/50;
  }
  body {
    @apply tw:bg-background tw:text-foreground;
  }
}

/* custom.module.css */
.card_container {
  --primary: oklch(0.6 0.2 290); /* Override to Purple locally */

  /* Any utility used here will now pick up the Purple --primary */
  @apply tw:bg-primary;
}

/*
### Runtime Behavior
When the browser loads your component:

1. It injects the specific styles for `custom.module.css`.
2. It sees that `.card_container` (which Vite might rename to something like `.card_container_x1y2z`) has its own `--primary`.
3. It applies that value **only to that specific DOM branch**.
4. As soon as the user navigates away or the component is unmounted, that "Local Override" effectively disappears from that part of the screen, but it **never** touched the global `:root` to begin with.
*/
```

---

- **Can I use `@theme` in a module?** **No.** It belongs only in your main global CSS file.
- **Can I overwrite variables in a module?** **Yes**, using standard CSS syntax (`--variable: value`).
- **Do I need `inline`?** **Yes.** Without `inline`, your local standard CSS variables won't "link up" with Tailwind's generated utilities because Tailwind may have renamed the variables during the build.

## **The CSS Cascade Logic**

In CSS, the "last one wins" rule is not a universal law; it is the **final tie-breaker** used only when importance and specificity are equal.

> With the introduction of `@layer`, the order of decision-making is now: **Importance > Layer Priority(inverted with !important) > Specificity > Source Order.**

### The "Full Stack" Priority Order

To visualize this, imagine your CSS as a stack. The browser looks at these in order from **Top (Strongest)** to **Bottom (Weakest)**:

1. **Transitioning Styles** (Highest)
2. **Important User-Agent styles** (Browser defaults)
3. **Important Local User styles** (User browser settings)
4. **Important Author styles (BASE Layer)** — _The Inversion starts here_
5. **Important Author styles (COMPONENTS Layer)**
6. **Important Author styles (UTILITIES Layer)**
7. **Important Author styles (UNLAYERED)**
8. **Animation styles**
9. **Normal Author styles (UNLAYERED)** — _Standard hierarchy starts here_
10. **Normal Author styles (UTILITIES Layer)**
11. **Normal Author styles (COMPONENTS Layer)**
12. **Normal Author styles (BASE Layer)**
13. **Normal User-Agent styles** (Lowest)

### **1. When `!important` is used**

Declarations marked with `!important` operate outside the normal rules of specificity.

- **Global Priority:** An `!important` declaration always wins over any normal declaration, regardless of its location or selector type.
- **Conflict Resolution:** If multiple `!important` rules target the same property:
- **Specificity Wins:** The rule with the **higher specificity** wins.
- **Source Order Wins:** If specificity is identical, the **last one defined** in the source code wins.

---

### **2. When `!important` is NOT used**

For all standard CSS rules, the browser follows a two-step elimination process:

#### **I. Specificity (The Primary Filter)**

- A more specific selector always beats a less specific one.
- **Hierarchy:** An **ID** (`#id`) beats a **Class** (`.class`), and a **Class** beats an **Element** (`p`, `div`).
- This remains true even if the less specific selector appears later in the stylesheet.

#### **II. Source Order (The Final Tie-Breaker)**

- This rule only triggers if both the **importance** and the **specificity** are exactly the same.
- In this case, the browser applies the rule that appears **last** in the CSS file.

### **Scenario A: Different Specificity**

Even though both have `!important`, the browser still looks at the "weight" of the selector.

```css
#header {
  color: red !important;
}

.title {
  color: blue !important;
}
```

- **Winner:** **Red**.
- **Why?** The `#id` selector has higher specificity than the `.class` selector. `!important` doesn't ignore specificity; it just moves the conflict to a "higher league" where specificity still matters.

---

### **Scenario B: Identical Specificity (The Tie-Breaker)**

If the selectors are identical (or have the exact same specificity score), the **Source Order** rule finally kicks in.

```css
.title {
  color: red !important;
}

/* ... some other code ... */

.title {
  color: blue !important;
}
```

- **Winner:** **Blue**.
- **Why?** This is the "Last One Wins" rule. Since the importance and specificity are a perfect tie, the browser picks the declaration it saw last.

---

### **Scenario C: Across @layers**

This is where it gets counter-intuitive. If you use `!important` inside CSS Layers, the **earlier layer wins**. This is called **Importance Inversion**.

```css
@layer base {
  .title {
    color: red !important;
  } /* This wins! */
}

@layer utilities {
  .title {
    color: blue !important;
  }
}
```

- **Why?** Normally, `utilities` beats `base`. But the CSS specification says that for `!important` styles, the priority is **inverted**. This is designed so that a "Base" reset with `!important` cannot be accidentally broken by a "Utility" with `!important`.

# Problem with MUI

Since we are working with **Material UI (MUI)**, remember that MUI injects its styles into the `<head>` at runtime. This means MUI's "Source Order" often comes **after** your external CSS files, which is why your styles might fail even if the specificity looks equal.

Using the **`StyledEngineProvider`** with `injectFirst` is one way to fix this without resorting to `!important`.

```jsx
import { StyledEngineProvider } from "@mui/material/styles";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <StyledEngineProvider injectFirst>
    <App />
  </StyledEngineProvider>,
);
```

---

<br />
<br />
<br />

# [PostCSS](https://postcss.org/docs/postcss-architecture)

**PostCSS** is the "engine" behind the scenes\*\* that almost every modern build tool uses to process CSS.

- **Preprocessors (Sass, Less):** You write code in a unique language (like `.scss`), and a compiler turns it into standard `.css`. They come with a "fixed" set of features (variables, nesting, etc.) out of the box.
- **PostCSS:** You write standard CSS (or modern CSS that isn't supported yet). PostCSS turns that CSS into a data tree (AST), lets **plugins** modify it, and then spits back out a standard CSS file.

### **1. How PostCSS Fits into the Ecosystem**

PostCSS doesn't usually run on its own in a React project. Instead, it "plugs into" your bundler.

- **[Vite](https://vite.dev/guide/features#css):** Has PostCSS support built-in. If it sees a `postcss.config.js`, it automatically runs your CSS through it.
- **Webpack:** Uses `postcss-loader` to do the same thing.
- **Next.js:** Has its own internal PostCSS configuration but allows you to override it.

---

### **2. What PostCSS actually does for you**

PostCSS is essentially a **"Plugin Host."** By itself, it does nothing. But with plugins, it becomes powerful:

1. **Tailwind CSS:** When you write `flex` or `text-red-500`, PostCSS is the tool that scans your HTML/JS files and generates the actual CSS for those classes.
2. **Autoprefixer:** This is the most famous PostCSS plugin. It looks at your CSS and adds "Vendor Prefixes" (like `-webkit-` or `-ms-`) so your dashboard works on old versions of Safari or Internet Explorer.

   These are called **Browser Engine Prefixes**. Back in the day, before CSS features were "official" or standardized, browser developers added these prefixes to test new features without breaking other browsers.

   | Prefix         | Browser Engine     | Primary Browsers                              |
   | -------------- | ------------------ | --------------------------------------------- |
   | **`-webkit-`** | WebKit / Blink     | Chrome, Safari, New Edge, Opera, iOS browsers |
   | **`-moz-`**    | Gecko              | Firefox                                       |
   | **`-ms-`**     | Trident / EdgeHTML | Internet Explorer, Old Edge (rarely used now) |
   | **`-o-`**      | Presto             | Old versions of Opera                         |

3. **CSS Modules:** PostCSS helps "scope" your CSS so that a class name like `.button` in `Home.css` doesn't clash with `.button` in `About.css`.

---

<br />
<br />

# [No Need for any Preprocessor now a days - Mordern native CSS is good enough](https://dev.to/dperrymorrow/you-dont-need-a-css-pre-processor-nj3)

<br />
<br />

# [SASS](https://www.youtube.com/watch?v=akDIJa0AP5c)

SCSS is a preprocessor scripting language that extends CSS with dynamic features like variables, nesting, and mixins.

When people say "I'm using Sass," they usually mean they are using the **Sass Preprocessor** but writing their code in the **SCSS format**. Basically it just means they are using curly braces instead of indentation.

**Standard CSS:**

```css
/* Declaration */
/* Css variables - Natively supported by browsers and live in the DOM, evaluated at runtime. */
:root {
  --primary-color: blue; /* Global scope */
}

.component {
  --component-bg: black; /* Local scope */
}

/* Usage */
.button {
  background: var(
    --primary-color,
    red
  ); /* will use red if --primary-color is not defined*/
  color: white;
}

.component {
  background-color: var(--component-bg);
  color: var(--primary-color);
}

/* repeated selectors*/
.nav {
  background: white;
}
.nav ul {
  list-style: none;
}
.nav ul li {
  display: inline-block;
}
```

**SCSS (Modern/Standard):**

```scss
// Sass variables are
// handled by a preprocessor during build time and are compiled away into plain CSS values.
$primary: blue;

.button {
  background: $primary;
  color: white;
  &:hover {
    background: darkblue;
  }
}
/* usablity/nesting */
.nav {
  background: white;
  ul {
    list-style: none;
    li {
      display: inline-block;
    }
  }
}
```

**Sass (Original/Indented):**

```sass
$primary: blue

.button
  background: $primary
  color: white
  &:hover
    background: darkblue

```

**It also Includes programmatic features like loops, functions, and mathematical operations for more dynamic styling.**

### **Mixins & Functions (Reusability)**

SCSS provides **Mixins**, which are like "functions for styles." If you have a complex group of styles you use everywhere (like a specific button shadow or a flex-center hack), you can define it once.

**SCSS Mixin:**

```scss
// Define a mixin with a parameter and a default value
@mixin border-radius($radius: 5px) {
  -webkit-border-radius: $radius; // Vendor prefixes for compatibility
  -moz-border-radius: $radius;
  border-radius: $radius;
}

// Include the mixin in a selector
.box {
  @include border-radius(10px);
}

.circle {
  @include border-radius(50%);
}

.square {
  @include border-radius; // Uses the default 5px value
}
```

### **Modularity (@use)**

In standard CSS, managing 10 different files is a nightmare because of how `@import` works (it creates extra HTTP requests).
SCSS allows you to split your dashboard into small files (e.g., `_sidebar.scss`, `_colors.scss`) and merge them into **one single CSS file** during the build process.

```scss
@use _sidebar.scss;
@use _color.scss;
```

<br />
<br />
<br />

# [Tailwind v4](https://tailwindcss.com/docs/compatibility)

Tailwind CSS v4 is a major architectural shift. While it is technically still available as a PostCSS plugin for backward compatibility, it has been rewritten from the ground up as a **high-performance CSS engine**.

Tailwind CSS v4 does not require PostCSS, as it has shifted to a high-performance, native Rust-based engine (Oxide) that handles imports, vendor prefixing, and nested CSS automatically. It is designed as a standalone tool or via framework-specific plugins (like Vite), eliminating the need for postcss.config.js or autoprefixer.

Tailwind v4 has **Lightning CSS built directly into its core**.

- **When used as a PostCSS plugin:** Tailwind uses Lightning CSS internally to handle things like `@import` bundling, vendor prefixing (replacing Autoprefixer), and modern syntax transforms (like `oklch()` colors).
- **The Result:** Even if you use it through PostCSS, you get the performance benefits of the new Rust-based engine (Oxide) and Lightning CSS's lightning-fast processing. You can typically **remove `autoprefixer` and `postcss-import`** from your project because Tailwind v4 handles those jobs now.

---

### **Tailwind + Vite: PostCSS or Lightning CSS?**

When you use Tailwind with Vite in v4, the "PostCSS vs. Lightning CSS" answer depends on which **Tailwind plugin** you choose to install:

#### **Option A: The Vite Plugin (Recommended)**

If you install `@tailwindcss/vite`, Tailwind integrates **directly into Vite’s internal build pipeline**.

- **How it works:** It bypasses PostCSS entirely for Tailwind-specific tasks. It uses Vite's module graph to detect which files you are using, making it significantly faster (up to 10x) than the PostCSS version.
- **Does it use Lightning CSS?** Yes, it uses its internal Lightning CSS engine to process your styles.
- **Note:** You can still have a `postcss.config.js` for _other_ non-Tailwind plugins, but the Tailwind Vite plugin will handle the bulk of the CSS work.

#### **Option B: The PostCSS Plugin**

If you choose to stay with the `@tailwindcss/postcss` plugin while using Vite:

- **How it works:** Vite sees the PostCSS configuration and hands the CSS over to PostCSS. PostCSS then calls the Tailwind plugin.
- **Does it use Lightning CSS?** Yes, the plugin still uses Lightning CSS internally, but the overhead of PostCSS makes it slightly slower than the dedicated Vite plugin.
- Using this pluging you can use actually tailwind v4 with sass (not recommended though),
  - **The Pipeline:**
    `SCSS Files` → `Sass Compiler` → `PostCSS (@tailwindcss/postcss)` → `LightningCss (internally)` → `Final CSS` (slower Two-step process)

| Feature       | [Lightning CSS](https://lightningcss.dev/)                                             | PostCSS                                                                      |
| ------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Language      | Rust                                                                                   | JavaScript                                                                   |
| Performance   | Incredibly fast (up to 100x faster than JS-based tools)                                | Slower due to JavaScript overhead and garbage collection                     |
| Architecture  | "Batteries-included" with built-in features (minification, prefixing, etc.)            | Modular, requires a chain of separate plugins for most tasks                 |
| Configuration | Simple, minimal configuration                                                          | Can lead to a "config jungle" with many plugins                              |
| Functionality | Handles core tasks like bundling, minification, and transpilation of future CSS syntax | Transforms CSS via a flexible plugin ecosystem, allowing for niche use cases |
| Ecosystem     | Smaller, growing ecosystem                                                             | Large, established ecosystem of plugins                                      |

## @apply

In Tailwind CSS, `@apply` is a directive that lets you "copy-paste" utility classes into a traditional CSS rule. It’s essentially a way to use Tailwind's power inside a standard `.css` file.

See below example, here we are writing tailwind classes in normal css file.

```css
.btn-primary {
  @apply px-4 py-2 bg-blue-500 text-white rounded-lg transition;
}

.btn-primary:hover {
  @apply bg-blue-600;
}
```

## [Preflight](https://tailwindcss.com/docs/preflight)

Preflight removes all of the default margins from all elements including headings, blockquotes, paragraphs, etc

This makes it harder to accidentally rely on margin values applied by the **user-agent stylesheet**(you can check this styles in css tab of dev tools, there will be user-agent stylesheet written, that is applied by browser) that are not part of your spacing scale.

<br />
<br />
<br />

# [CSS Layers](https://www.youtube.com/watch?v=Pr1PezCc4FU)

**CSS Cascade Layers** (`@layer`) are a modern CSS feature that gives you explicit control over the **cascade**—the logic the browser uses to decide which style wins when multiple rules apply to the same element.

Traditionally, the cascade was determined by source order and selector specificity (e.g., an ID always beats a class). Cascade layers add a new dimension: **Layer Order**. Styles in a higher-priority layer will always override styles in a lower-priority layer, regardless of how specific the selectors are.

You can name your layers anything

### Priority Order of `!important`

When you use `!important`, the rules of the cascade are essentially **flipped**. While normal styles prioritize the "last declared" layer, `!important` styles prioritize the **earlier declared** layers.

```css
// this line will define the order
@layer theme, base, components, utilities;

@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities)
```
