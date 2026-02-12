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
const LazyComponent = React.lazy(() => import("./LazyComponent"));
```

### 2. The Role of `Suspense`

Because the code is now in a separate file, React has to fetch it over the network when it's time to render. This takes time.

`Suspense` is a component that wraps your lazy component. It tells React: _"While you are waiting for the network to finish downloading this component's code, show this fallback UI (like a spinner) instead."_

```jsx
import React, { Suspense } from "react";

const LazyComponent = React.lazy(() => import("./LazyComponent"));

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

<br />

# Hooks

### 1. **useReducer**
   In React, `useReducer` is an alternative to `useState` that is better suited for managing complex state logic or state that depends on the previous state.

   It is modeled after Redux, using a "reducer" function and "actions" to update data.

   To use `useReducer`, you need three distinct pieces:

   1. **State:** The current data (usually an object).
   2. **Action:** A plain JavaScript object that tells the reducer what to do (e.g., `{ type: 'INCREMENT' }`).
   3. **Reducer:** A function that takes the current `state` and the `action`, and returns the **new state**.

  ```javascript
  // 1. Define the Reducer
  const reducer = (state, action) => {
    switch (action.type) {
      case "SET_MESSAGE":
        return { ...state, message: action.payload };
      case "TOGGLE_LOADING":
        return { ...state, isLoading: !state.isLoading };
      default:
        return state;
    }
  };

  // 2. Component Logic
  function UploadComponent() {
    // const [state, dispatch] = useReducer(reducer, initialState);
    const [state, dispatch] = useReducer(reducer, {
      message: "",
      isLoading: false,
    });

    const handleUpdate = () => {
      // 3. Dispatch an Action
      dispatch({ type: "SET_MESSAGE", payload: "Upload Starting..." });
    };

    return <button onClick={handleUpdate}>Update State</button>;
  }
  ```

In `useState`, you call `setState(newValue)`. In `useReducer`, you don't update the state directly. Instead, you **dispatch** an intent.

* **Decoupling:** The component doesn't need to know *how* the state changes; it just says "Hey, this event happened."
* **Predictability:** Since all state changes happen inside one reducer function, it's much easier to debug.
* **Complex Payloads:** You can send lots of data in the `payload` property of the action object.


### 2. **useMemo**
The `useMemo` hook is a React tool for **performance optimization**. It allows you to "memoize" (cache) the result of a calculation between re-renders so that the work is only done when absolutely necessary.

`const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);`

* **The Function:** A function that performs the calculation and returns a value.
* **The Dependencies:** An array of values. The calculation only runs again if one of these values changes.

You should use `useMemo` in two main scenarios:

#### 1. Expensive Calculations

If you have a function that takes a long time to run (like sorting 10,000 rows of data or performing heavy math), you don't want it running every single time your component re-renders (e.g., when a timer ticks or a user types in a separate text box).

#### 2. Referential Equality (Avoiding Unnecessary Re-renders)

In JavaScript, `[] !== []` and `{} !== {}`. If you create an object or array inside your component, it gets a **new memory address** every render.
If that object is passed as a prop to a `React.memo`(React.memo caches the entire component, it performs a shallow comparison of the props) child component, the child will re-render even if the data inside the object is the same. `useMemo` keeps the **reference** the same.


#### `useMemo` vs. `useCallback`

These two are often confused. Here is the simple distinction:

| Hook | What it caches (memoizes) | Usage |
| --- | --- | --- |
| **`useMemo`** | The **result** of a function (a value, object, or array). | `const value = useMemo(() => val, [deps])` |
| **`useCallback`** | The **function itself**. | `const fn = useCallback(() => {...}, [deps])` |

---

#### Don't over-use it!

It is tempting to wrap everything in `useMemo`, but this can actually **hurt** performance.

* **Memory Cost:** Storing values in memory isn't free.
* **Comparison Cost:** React has to compare the dependency array every render to see if they changed.

**Rule of thumb:** Only use it if you are noticing a lag in the UI or if you are passing complex objects down to highly-optimized child components.


### 3. useEffect

The primary pattern for using `setTimeout` in a functional component is to wrap it inside a `useEffect` hook and return a cleanup function that calls clearTimeout. This ensures the **timer is canceled** if the component unmounts before the delay finishes, preventing potential errors or memory leaks. 

```javascript
const [message, setMessage] = useState('');

  useEffect(() => {
    const timerId = setTimeout(() => {
      setMessage('Delayed message after 2 seconds!');
    }, 2000);

    return () => {
      clearTimeout(timerId);
    };
  }, [])
```

```javascript
// check this out
const [v, setV] = useState(true); 

// Effect A
useEffect(() => {
  console.log("Effect A (Empty Array)");
  setV(false); 
}, []);

// Effect B
useEffect(() => {
  if (v === true) {
    console.log("Effect B (With Variable)");
  }
}, [v]);

```

There are two distinct "Render Cycles" that occur here.

#### Cycle 1: The Initial Mount

1. **Render:** The component renders with `v = true`.
2. **Effect A Runs:** Logs `"Effect A (Empty Array)"`.
3. **State Update:** `setv(false)` is called inside Effect A. This schedules a **re-render**.
4. **Effect B Runs:** Since it is the first mount, it checks `if (v == true)`. Because `v` is still `true` in this render's scope, it logs `"Effect B (With Variable)"`.

#### Cycle 2: The Re-render (caused by Effect A)

1. **Render:** The component re-renders with `v = false`.
2. **Effect A:** Does **not** run (empty array `[]` only runs on mount).
3. **Effect B:** Detects that `v` changed from `true` to `false`. It runs again. However, the condition `if (v == true)` is now **false**, so it logs nothing.


### 4. useContext
In React, `useContext` and the `Provider` component are the two main parts of the **Context API**. Together, they solve the problem of **prop drilling**—the tedious process of passing data through multiple layers of components that don't actually need it.

---

#### The Provider component

The `Provider` is a special component that wraps around a section of your app. It has a `value` prop which holds the data you want to share (like a user object, a theme, or a language setting).

* **Scope:** Only components wrapped inside the Provider can access the data.
* **Updates:** When the `value` prop of the Provider changes, every component "tuned in" via `useContext` will automatically re-render.

#### The useContext Hook

`useContext` is a Hook used inside functional components to "read" the data from the nearest Provider above it in the tree.

* **Syntax:** `const value = useContext(MyContext);`
* **Direct Access:** It skips all the intermediate components in the middle.

---

#### Step 1: Create the Context

First, you create a "Context Object" in a separate file or at the top of your component.

```javascript
import { createContext } from 'react';

// 'light' is the default value used if no Provider is found
export const ThemeContext = createContext('light');

```

#### Step 2: Wrap with the Provider

Place the Provider at a high level in your app and pass the data into the `value` prop.

```javascript
function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={theme}>
      <MainPage /> 
    </ThemeContext.Provider>
  );
}

```

#### Step 3: Consume with useContext

Inside any nested component (no matter how deep), call the Hook.

```javascript
import { useContext } from 'react';
import { ThemeContext } from './App';

function Button() {
  // Directly grab the 'theme' without it being passed as a prop
  const theme = useContext(ThemeContext);
  
  return <button className={theme}>Click Me</button>;
}

```

---

While Context is powerful, it makes components harder to reuse because they "depend" on a Provider being present. Use it for:

* **Global State:** Themes (Dark/Light mode), User authentication, or Language/Localization.
* **Broadly Shared Data:** Data that many components at different levels need to know.

> **Note:** If you only need to pass data down 1 or 2 levels, sticking with **props** is usually better for keeping your code simple and your components independent.

---

<br />
<br />
<br />

# [Nest.js](https://www.youtube.com/watch?v=k7o9R6eaSes)

**Next.js** is a popular open-source web development framework built on top of **React**. While React is a library for building user interfaces, Next.js provides the "architecture" and tools needed to build full-scale, production-ready web applications.

### What is Next.js?

Often called "The React Framework for the Web," Next.js simplifies the development process by handling complex features out of the box that you would otherwise have to configure manually in a standard React app.

- **Rendering Options:** It allows for Server-Side Rendering (SSR), Static Site Generation (SSG), and Incremental Static Regeneration (ISR), which makes websites much faster and better for SEO.
- **File-Based Routing:** You don't need a separate library like React Router; any file added to the `app` or `pages` directory automatically becomes a route.
- **Full-Stack Capabilities:** It allows you to write backend code (API routes) in the same project as your frontend.
- **Optimization:** It automatically optimizes images, fonts, and scripts to improve performance.

---

### What Build Tool Does it Use?

It kinda itself act as a buildtool, Next.js uses a combination of powerful tools to compile, bundle, and optimize your code. As of 2025, the framework is in a transition period between two major technologies:

#### 1. The Compiler: SWC

Next.js uses **SWC** (Speedy Web Compiler) as its primary compiler. SWC is written in **Rust** and is significantly faster than the older industry standard, Babel. It handles:

- **Transpilation:** Turning modern JavaScript/TypeScript into code browsers can understand.
- **Minification:** Shrinking your code size for production.

#### 2. The Bundler: Turbopack & Webpack

The "bundler" is what takes all your individual files and packages them together.

- **Turbopack (The New Standard):** Developed by Vercel (the creators of Next.js), Turbopack is an incremental bundler written in Rust. In the latest versions of Next.js (like version 15 and 16), it is the default for local development because it is up to **700x faster** than Webpack for large projects.
- **Webpack (The Legacy Standard):** For years, Webpack was the default. While Next.js is moving toward Turbopack for everything, Webpack is still used for production builds in many configurations or if you have custom plugins that haven't migrated to the Turbo ecosystem yet.

### Server Action

- **Without `"use server"`:** The function is just private code inside the house. The browser has no way to "call" it.
- **With `"use server"`:** You are telling Next.js to build a hidden bridge (an HTTP POST endpoint). When the user clicks the button, the browser sends a request across that bridge to execute that specific function on the server.

<br />
<br />
<br />

# [SASS](https://www.youtube.com/watch?v=akDIJa0AP5c)

SCSS is a preprocessor scripting language that extends CSS with dynamic features like variables, nesting, and mixins.

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
  &:hover {
    background: darkblue;
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
