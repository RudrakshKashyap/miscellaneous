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

| Package Manager                | Will it install React?       | Will the terminal command fail? | What happens at runtime?                                                                                            |
| ------------------------------ | ---------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **`npm` (v7+)**                | <br>**Yes** (Auto-installs). | No.                             | The package sits in your files, but running React logic inside Angular templates will fail without bridge wrappers. |
| **`pnpm` (Default)**           | <br>**No** (Skips it).       | No (Logs a Warning).            | <br>**Fails at runtime** immediately when the package calls missing React code.                                     |
| **`pnpm` (`auto-install...`)** | <br>**Yes** (Auto-installs). | No.                             | Behaves the same as `npm`.                                                                                          |
| **`pnpm` (`strict-peer...`)**  | **No**                       | <br>**Yes** (Fails instantly).  | N/A (Build/Install blocks completely).                                                                              |

### The Real-World Verdict

Even if your package manager successfully finishes the installation without crashing, **Angular cannot natively execute React components or hooks**. Unless you are planning to utilize a specific micro-frontend bridge engine to host a containerized React element inside your Angular zone, you should look for a framework-agnostic (Vanilla JS) or Angular-native alternative to that package.

<br />
<br />
<br />

---

## The Three Layers

### 1. The global store (one per machine)

```
/Users/rudraksh/Library/pnpm/store/v11
```

This is where package content actually lives on disk, exactly once per version, globally — shared across every project on your machine. Files here are content-addressed (hashed). When you `pnpm install`, pnpm downloads a package here only if this exact version isn't already present. If you have 10 projects using `react@19.2.6`, it's stored once.

Crucially, files are **hard-linked** from the store into your project, not copied. So `node_modules` takes almost no extra disk space — it's pointers to the store.

---

### 2. The repo-root virtual store: `node_modules/.pnpm`

```
node_modules/.pnpm/turbo@2.10.4/node_modules/turbo
node_modules/.pnpm/react-dom@19.2.6_react@19.2.6/node_modules/...
```

This is the one real `node_modules` for your whole monorepo. Every package any workspace depends on gets an entry here, hard-linked from the global store. Notice the naming:

- **`turbo@2.10.4`** — a flat, version-stamped folder per package.
- **`react-dom@19.2.6_react@19.2.6`** — the suffix after `_` encodes its peer dependencies (this `react-dom` was resolved against `react` 19.2.6). This is how pnpm keeps peer-dep variants separate.

Inside each `.pnpm/<pkg>/node_modules/`, that package's own dependencies are symlinked to their `.pnpm` entries. This is what makes pnpm strict: a package can only require what it actually declared, because only its real deps are linked next to it. (npm/yarn's flat hoisting lets you accidentally import undeclared packages.)

---

### 3. The per-location `node_modules` (root + each app/package)

These contain only symlinks, and only for the packages that location directly depends on. Two kinds:

**a) Symlinks into `.pnpm` (external deps):**

```
apps/dashboard/node_modules/typescript
-> ../../../node_modules/.pnpm/typescript@6.0.3/node_modules/typescript
```

So when Vite/Node in `apps/dashboard` resolves `import ... from "typescript"`, it follows this symlink → lands in the root `.pnpm` store → which hard-links to the global store.

**b) Symlinks to your own workspace packages (this is the monorepo magic):**

```
apps/dashboard/node_modules/@workspace/ui
-> ../../../../packages/ui
```

`@workspace/ui` is not a downloaded package — the symlink points straight at your `packages/ui` source folder. That's why editing `packages/ui` is instantly reflected in `apps/dashboard` with no rebuild/republish. pnpm wired this up because `pnpm-workspace.yaml` lists `packages/*`, and `apps/dashboard/package.json` declares `"@workspace/ui": "workspace:*"`.

---

## Answering Your Literal Questions

### Where does my module get installed when I run `pnpm install`?

Physically into the global store (`~/Library/pnpm/store/v11`), once. Then hard-linked into the repo-root `node_modules/.pnpm`. Then symlinked into whichever workspace `node_modules` folders declared it as a dependency.

### How are the 3 `node_modules` folders related? What manages what?

pnpm manages all of them from the root, driven by `pnpm-workspace.yaml` + each `package.json`.

- **Root `node_modules`** = the truth: holds `.pnpm/` (every version of everything, hard-linked from the global store) plus symlinks for root-level deps.
- **`apps/dashboard/node_modules`** and **`packages/ui/node_modules`** = thin symlink-only folders. Each contains just that workspace's declared deps, every one pointing back "up" into the root's `.pnpm/` — or, for workspace deps, sideways to `packages/*` source.

So they're not three independent installs — they're **one install** (root `.pnpm`) with per-workspace symlink "views" on top of it. The leaf `node_modules` folders answer "what is this workspace allowed to import," while the root `.pnpm` + global store answer "where does the actual code live."

---

## The Chain, End to End

**`apps/dashboard` imports `"react-dom"`**

```
apps/dashboard/node_modules/react-dom (symlink)
→ node_modules/.pnpm/react-dom@19.2.6_react@19.2.6/... (repo virtual store)
→ ~/Library/pnpm/store/v11/... (hard link → real bytes, shared globally)
```

**`apps/dashboard` imports `"@workspace/ui"`**

```
apps/dashboard/node_modules/@workspace/ui (symlink)
→ packages/ui/ (your own source — no store involved)
```

> One mental model: the global store is the **warehouse** (real goods, one copy), root `.pnpm` is your repo's **shelf** (labeled links to the warehouse), and each workspace's `node_modules` is a small **basket** holding only the items that workspace ordered.

---

```text
pnpm lint
└─ turbo lint                       (turbo.json: dependsOn ^lint → topological order)
   ├─ packages/ui        → eslint   → ui/eslint.config.js
   │                                   → @workspace/eslint-config/react-library
   └─ apps/dashboard     → eslint   → dashboard/eslint.config.js
                                       → @workspace/eslint-config/vite-app
   (eslint-config, typescript-config, root → no lint script → skipped)
```
