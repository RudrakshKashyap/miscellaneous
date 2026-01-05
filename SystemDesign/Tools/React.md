props - function parameter
state - object to store info & you can display it in component
Q state vs normal variable
react context redux, comp life cycle

arrow functions

react query


react fiber
https://www.youtube.com/@ReactNext/videos

https://www.youtube.com/watch?v=-KEuTPIpLbE


# [Lazy & Suspense](https://react.dev/reference/react/lazy) (automatic in NEXT.js)

The most common place to use this is at the **Route level**. This way, you only download the code for the specific page the user is visiting.

### 1. How `React.lazy` Handles Code Splitting

Normally, when you `import` a component at the top of your file, it gets bundled into your main `index.js` or `app.js`. If your app is large, this makes the initial download very slow.

When you use `React.lazy`, you use a **dynamic import**. This tells build tools like Webpack, Vite, or Esbuild to create a separate file for that component.

```javascript
// This component is NOT in the main bundle anymore.
// It will be in a separate file like 'SomeComponent.chunk.js'
const LazyComponent = React.lazy(() => import('./LazyComponent'));

```

### 2. The Role of `Suspense`

Because the code is now in a separate file, React has to fetch it over the network when it's time to render. This takes time.

`Suspense` is a component that wraps your lazy component. It tells React: *"While you are waiting for the network to finish downloading this component's code, show this fallback UI (like a spinner) instead."*

```jsx
import React, { Suspense } from 'react';

const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading component...</div>}>
        <LazyComponent />
      </Suspense>
    </div>
  );
}

```


## [Suspense / Streaming SSR](https://www.youtube.com/watch?v=cwjsoOZVK34)




---
<br />
<br />
<br />


# [Nest.js](https://www.youtube.com/watch?v=k7o9R6eaSes)

**Next.js** is a popular open-source web development framework built on top of **React**. While React is a library for building user interfaces, Next.js provides the "architecture" and tools needed to build full-scale, production-ready web applications.

### What is Next.js?

Often called "The React Framework for the Web," Next.js simplifies the development process by handling complex features out of the box that you would otherwise have to configure manually in a standard React app.

* **Rendering Options:** It allows for Server-Side Rendering (SSR), Static Site Generation (SSG), and Incremental Static Regeneration (ISR), which makes websites much faster and better for SEO.
* **File-Based Routing:** You don't need a separate library like React Router; any file added to the `app` or `pages` directory automatically becomes a route.
* **Full-Stack Capabilities:** It allows you to write backend code (API routes) in the same project as your frontend.
* **Optimization:** It automatically optimizes images, fonts, and scripts to improve performance.

---

### What Build Tool Does it Use?

It kinda itself act as a buildtool, Next.js uses a combination of powerful tools to compile, bundle, and optimize your code. As of 2025, the framework is in a transition period between two major technologies:

#### 1. The Compiler: SWC

Next.js uses **SWC** (Speedy Web Compiler) as its primary compiler. SWC is written in **Rust** and is significantly faster than the older industry standard, Babel. It handles:

* **Transpilation:** Turning modern JavaScript/TypeScript into code browsers can understand.
* **Minification:** Shrinking your code size for production.

#### 2. The Bundler: Turbopack & Webpack

The "bundler" is what takes all your individual files and packages them together.

* **Turbopack (The New Standard):** Developed by Vercel (the creators of Next.js), Turbopack is an incremental bundler written in Rust. In the latest versions of Next.js (like version 15 and 16), it is the default for local development because it is up to **700x faster** than Webpack for large projects.
* **Webpack (The Legacy Standard):** For years, Webpack was the default. While Next.js is moving toward Turbopack for everything, Webpack is still used for production builds in many configurations or if you have custom plugins that haven't migrated to the Turbo ecosystem yet.


### Server Action

* **Without `"use server"`:** The function is just private code inside the house. The browser has no way to "call" it.
* **With `"use server"`:** You are telling Next.js to build a hidden bridge (an HTTP POST endpoint). When the user clicks the button, the browser sends a request across that bridge to execute that specific function on the server.


<br />
<br />
<br />

# [SASS](https://www.youtube.com/watch?v=akDIJa0AP5c)


When people say "I'm using Sass," they usually mean they are using the **Sass Preprocessor** but writing their code in the **SCSS format**. Basically it just means they are using curly braces instead of indentation.


**Standard CSS:**

```css
.button {
  background: blue;
  color: white;
}

```

**SCSS (Modern/Standard):**

```scss
$primary: blue;

.button {
  background: $primary;
  color: white;
  &:hover { background: darkblue; }
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