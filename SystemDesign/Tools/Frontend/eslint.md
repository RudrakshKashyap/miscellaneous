## ES-lint and other node packages

### 1. The Extension is just a "Bridge"

The VS Code ESLint extension does not actually know how to lint code. It is an interface that looks into your project’s `node_modules`, finds the `eslint` npm package, and runs it in the background to show you those red squiggles.

If you don't have the npm package installed:

- The extension will look for a **Global** version on your Mac.
- If you don't have a global version, the extension simply **won't work**.
- Even if you have a global version, using it is risky because your teammate might have a different global version, leading to "It works on my machine" bugs.

### 2. Version Locking

By installing `eslint` as an npm package in your `package.json`, you are **locking the version**.

- **Without the package:** Your VS Code might use ESLint v9, while your teammate uses v8. You will see different errors on the exact same code.
- **With the package:** Everyone who runs `npm install` gets the exact same version of the linter.

### 3. CI/CD and Automated Checks

The VS Code extension only works when you have the file open.

- **Vercel/GitHub Actions:** When you push code to Vercel, it doesn't "open VS Code." It runs a script. If `eslint` isn't in your `package.json`, your build pipeline can't run `npm run lint` to verify the code quality before deploying.
- **Husky/Pre-commit:** If you want to prevent people from committing "bad" code, your git hooks need the npm package to run the check automatically in the terminal.

---

| Component                     | Role                                | Analogy       |
| ----------------------------- | ----------------------------------- | ------------- |
| **`eslint` (npm package)**    | The logic that analyzes the code.   | The Engine    |
| **`eslint.config.js` (file)** | The list of rules to follow.        | The Rulebook  |
| **ESLint Extension**          | Displays the errors in your editor. | The Dashboard |

```
prettier is for formatting the code while
Eslint is for watching the quality of the code and the best practices.
the two are different and save different purposes.
both are important for good quality projects.
you can make a prettier config and make it run each time someone made a commit to the git by using something like husky so you no longer need to set up formatting rules in eslint.
```

<br />
<br />
<br />

---

# [jsconfig.json](https://code.visualstudio.com/docs/languages/jsconfig)

The file is primarily used by the VS Code editor to provide better code completion and type checking for JavaScript files.

For vite you need and extension to read it:

> tsconfigPaths(), // Reads aliases directly from jsconfig.json!
