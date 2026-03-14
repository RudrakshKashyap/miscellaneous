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

1.  **State:** The current data (usually an object).
2.  **Action:** A plain JavaScript object that tells the reducer what to do (e.g., `{ type: 'INCREMENT' }`).
3.  **Reducer:** A function that takes the current `state` and the `action`, and returns the **new state**.

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

- **Decoupling:** The component doesn't need to know _how_ the state changes; it just says "Hey, this event happened."
- **Predictability:** Since all state changes happen inside one reducer function, it's much easier to debug.
- **Complex Payloads:** You can send lots of data in the `payload` property of the action object.

### 2. **useMemo**

The `useMemo` hook is a React tool for **performance optimization**. It allows you to "memoize" (cache) the result of a calculation between re-renders so that the work is only done when absolutely necessary.

`const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);`

- **The Function:** A function that performs the calculation and returns a value.
- **The Dependencies:** An array of values. The calculation only runs again if one of these values changes.

You should use `useMemo` in two main scenarios:

#### 1. Expensive Calculations

If you have a function that takes a long time to run (like sorting 10,000 rows of data or performing heavy math), you don't want it running every single time your component re-renders (e.g., when a timer ticks or a user types in a separate text box).

#### 2. Referential Equality (Avoiding Unnecessary Re-renders)

In JavaScript, `[] !== []` and `{} !== {}`. If you create an object or array inside your component, it gets a **new memory address** every render.
If that object is passed as a prop to a `React.memo`(React.memo caches the entire component, it performs a shallow comparison of the props) child component, the child will re-render even if the data inside the object is the same. `useMemo` keeps the **reference** the same.

#### `useMemo` vs. `useCallback`

These two are often confused. Here is the simple distinction:

| Hook              | What it caches (memoizes)                                 | Usage                                         |
| ----------------- | --------------------------------------------------------- | --------------------------------------------- |
| **`useMemo`**     | The **result** of a function (a value, object, or array). | `const value = useMemo(() => val, [deps])`    |
| **`useCallback`** | The **function itself**.                                  | `const fn = useCallback(() => {...}, [deps])` |

---

#### Don't over-use it!

It is tempting to wrap everything in `useMemo`, but this can actually **hurt** performance.

- **Memory Cost:** Storing values in memory isn't free.
- **Comparison Cost:** React has to compare the dependency array every render to see if they changed.

**Rule of thumb:** Only use it if you are noticing a lag in the UI or if you are passing complex objects down to highly-optimized child components.

### 3. useEffect

The primary pattern for using `setTimeout` in a functional component is to wrap it inside a `useEffect` hook and return a cleanup function that calls clearTimeout. This ensures the **timer is canceled** if the component unmounts before the delay finishes, preventing potential errors or memory leaks.

```javascript
const [message, setMessage] = useState("");

useEffect(() => {
  const timerId = setTimeout(() => {
    setMessage("Delayed message after 2 seconds!");
  }, 2000);

  return () => {
    clearTimeout(timerId);
  };
}, []);
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

- **Scope:** Only components wrapped inside the Provider can access the data.
- **Updates:** When the `value` prop of the Provider changes, every component "tuned in" via `useContext` will automatically re-render.

#### The useContext Hook

`useContext` is a Hook used inside functional components to "read" the data from the nearest Provider above it in the tree.

- **Syntax:** `const value = useContext(MyContext);`
- **Direct Access:** It skips all the intermediate components in the middle.

---

#### Step 1: Create the Context

First, you create a "Context Object" in a separate file or at the top of your component.

```javascript
import { createContext } from "react";

// 'light' is the default value used if no Provider is found
export const ThemeContext = createContext("light");
```

#### Step 2: Wrap with the Provider

Place the Provider at a high level in your app and pass the data into the `value` prop.

```javascript
function App() {
  const [theme, setTheme] = useState("dark");

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
import { useContext } from "react";
import { ThemeContext } from "./App";

function Button() {
  // Directly grab the 'theme' without it being passed as a prop
  const theme = useContext(ThemeContext);

  return <button className={theme}>Click Me</button>;
}
```

---

While Context is powerful, it makes components harder to reuse because they "depend" on a Provider being present. Use it for:

- **Global State:** Themes (Dark/Light mode), User authentication, or Language/Localization.
- **Broadly Shared Data:** Data that many components at different levels need to know.

> **Note:** If you only need to pass data down 1 or 2 levels, sticking with **props** is usually better for keeping your code simple and your components independent.

---

<br />
<br />
<br />

---

## The "Full Reload" vs. SPA Routing

In both CRA and Vite, using `window.location.href` causes a **hard refresh**. The browser throws away the current state, re-downloads the HTML/JS, and restarts the app.

**The Better Way:**
If you are using `react-router-dom`, you should use the `Maps` function or `<Link>` component. This changes the URL without refreshing the page.

```javascript
// Instead of this:
window.location.href = "/dashboard";

// Use this (React Router):
const navigate = useNavigate();
navigate("/dashboard");
```

---

### 2. The Vercel "404" Trap (The Real Risk)

When you move to Vite on Vercel, the way your routes are handled by the server might change if your `vercel.json` isn't set up for an SPA.

If a user goes to `/someroute` via `window.location.href`, the browser asks Vercel for a file at `/someroute/index.html`. Since that file doesn't exist (Vite creates one single `index.html`), Vercel will throw a **404 error** unless you have a rewrite rule.

**The Fix:**
Ensure you have a `vercel.json` file in your root directory to catch all routes and point them to your Vite entry point:

```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

### Why this fixes it:

When Vercel receives the request for `/wms/picker/scanCode/1893584`, it will now see your `vercel.json` rule. Instead of giving up and showing a 404, it will "secretly" serve the `index.html` file.

Once `index.html` loads in the browser, your **React Router** starts up, looks at the URL in the address bar, and says: _"Oh, I know that route! I'll render the ScanCode component for ID 1893584."_
