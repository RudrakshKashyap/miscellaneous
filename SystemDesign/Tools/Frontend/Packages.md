In any Node.js or monorepo project, the `package.json` file uses three distinct buckets to manage your packages. Understanding the difference prevents bundle bloat, version mismatches, and deployment crashes.

Here is the breakdown of **dependencies**, **devDependencies**, and **peerDependencies**.

---

## 1. Dependencies (`dependencies`)

**"The load-bearing pillars of your live application."**

These are the packages that your application **absolutely requires to run in production**. Without them, your final built code will crash because it relies on their logic at runtime. When you deploy your app to production, your hosting platform will install these packages.

- **When to use:** For utility libraries, state management, or HTTP clients that are directly compiled into your user-facing code.
- **Examples:** `axios`, `lodash`, `lucide-react`, `zustand`.
- **Install Command:** `npm install <package>` (or `pnpm add <package>`)

---

## 2. Dev Dependencies (`devDependencies`)

**"The scaffolding used to build the house."**

These are packages needed **only during local development and the build step**. Once your code is compiled into plain JavaScript/TypeScript files for production, these tools are completely discarded. They are never shipped to the end-user's browser.

- **When to use:** For compilers, bundlers, linters, formatters, and testing suites.
- **Examples:** `vite`, `typescript`, `eslint`, `tailwindcss`, `vitest`.
- **Install Command:** `npm install -D <package>` (or `pnpm add -D <package>`)

---

## 3. Peer Dependencies (`peerDependencies`)

**"The 'Bring Your Own' architectural agreement."**

This is the trickiest one, primarily used when you are **building a library or a shared package** meant to be consumed by other applications.

By declaring a peer dependency, your package says: _"I need this specific tool to function, but I am not going to install my own copy. I expect the main application importing me to already have it installed."_ \* **When to use:** When building plugins or shared component libraries to prevent the "Duplicate Framework Nightmare." For instance, if a shared icon package bundles its own copy of React, and the main app brings another copy, the browser loads two separate engines and throws fatal runtime errors.

- **Example:** `react`, `react-dom`, `tailwindcss` (inside a UI component library package).
- **How it looks in `package.json`:**

```json
"peerDependencies": {
  "react": "^18.0.0 || ^19.0.0"
}
}

```

---

## Summary at a Glance

| Dependency Type        | Needed in Dev? | Needed in Production Build? | Who Installs It?                                           | Typical Use Case                              |
| ---------------------- | -------------- | --------------------------- | ---------------------------------------------------------- | --------------------------------------------- |
| **`dependencies`**     | Yes            | **Yes**                     | Your app's package manager automatically.                  | Runtime code (e.g., Axios, Core UI logic).    |
| **`devDependencies`**  | Yes            | **No**                      | Your app's package manager, but only for building/testing. | Build tools (e.g., Vite, TypeScript, ESLint). |
| **`peerDependencies`** | No             | **Yes**                     | **The parent application** that imports your library.      | Core ecosystems (e.g., React, Tailwind).      |

<br />
<br />
<br />

---

# what happens if you install a package that has a peer dependency react into an angular app?

Installing a package that expects React as a peer dependency into an Angular application creates a classic architectural mismatch. Because an Angular app naturally won’t have React in its primary configuration, React counts as a **missing peer dependency**.

What happens next—whether your project silently tolerates it, automatically down-downloads it, or completely blows up—depends entirely on the **package manager** you are running.

---

## 1. If you are using standard `npm` (v7+)

Modern versions of `npm` take a fully automated approach to missing peer links:

- **The Outcome:** `npm` will scan the package, see that React is missing, and **automatically download and install React** into your Angular project's `node_modules` folder.

- **The Catch:** `npm` doesn't look at your project architecture; it blindly satisfies the configuration requirements. You will end up with the entire React library sitting on your hard drive alongside Angular, slightly bloating your local storage.

---

## 2. If you are using `pnpm` (Default Behavior)

`pnpm` takes a more hands-off approach out of the box, throwing the responsibility back to you:

- **The Outcome:** It will neither force-install React nor stop the installation process. It will successfully pull your new package down but will print a conspicuous warning message in your terminal: `WARN Issues with peer dependencies found`.

- **The Runtime Risk:** Because `pnpm` skipped installing React , the moment your Angular application boots up and the imported package tries to execute React-specific lifecycle methods or JSX hooks, your application will **fail at runtime** with a fatal crash because the expected engine code is completely absent from the memory tree.

---

## 3. If your `pnpm` configuration is customized

If you have specified strict rules inside a local `.npmrc` configuration file, the default behavior changes:

- **With `auto-install-peers=true`:** `pnpm` will mimic `npm` and automatically install React directly into the node modules space to resolve the dependency gap.

- **With `strict-peer-dependencies=true`:** The package manager will treat the missing React dependency as an unpardonable mistake. It will **fail the installation immediately** in your terminal with an `ERR_PNPM_PEER_DEP_ISSUES` code, refusing to alter your lockfile until you fix it.

---

## Summary of Behaviors

| Package Manager | Will it install React?       | Will the terminal command fail? | What happens at runtime? |
| --------------- | ---------------------------- | ------------------------------- | ------------------------ |
| **`npm` (v7+)** | <br>**Yes** (Auto-installs). | No. | The package sits in your files, but running React logic inside Angular templates will fail without bridge wrappers. |
| **`pnpm` (Default)** | <br>**No** (Skips it). | No (Logs a Warning). | <br>**Fails at runtime** immediately when the package calls missing React code. |
| **`pnpm` (`auto-install...`)** | <br>**Yes** (Auto-installs). | No. | Behaves the same as `npm`. |
| **`pnpm` (`strict-peer...`)** | **No** | <br>**Yes** (Fails instantly). | N/A (Build/Install blocks completely). |

### The Real-World Verdict

Even if your package manager successfully finishes the installation without crashing, **Angular cannot natively execute React components or hooks**. Unless you are planning to utilize a specific micro-frontend bridge engine to host a containerized React element inside your Angular zone, you should look for a framework-agnostic (Vanilla JS) or Angular-native alternative to that package.
