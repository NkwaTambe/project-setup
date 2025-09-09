# Project Structure and Architecture Guide

This document explains the structure of this monorepo, the reasoning behind it, and the conventions you must follow.

### The Core Problem: Your Project Was Not a Monorepo

You had three separate projects sitting in the same folder. That is not a monorepo. It was a future maintenance disaster. My entire goal was to fix that by establishing a **single source of truth** for your dependencies, tooling, and shared code.

---

### 1. Dependency & Tooling Unification

**The Problem:**
-   Each of your apps (`account-manager-app`, etc.) had its own `package.json` with identical `devDependencies` (like `vite`, `typescript`, `react`). Updating a dependency would require you to do it in three separate places.
-   You had two different linters: `biome` in the root and `eslint` in each app. This guarantees conflicting rules and developer confusion.

**The Fix:**
1.  **Hoisted Dependencies:** I moved all common `devDependencies` into the single `package.json` at the project's root. When you run `pnpm install`, pnpm is smart enough to make these tools available to all the sub-packages. You now update a version once, and every package gets it. Simple.
2.  **Single Linter:** I removed `eslint` entirely. `biome` was already in your root, so we use that for everything: linting, formatting, and import sorting. One tool, one configuration (`biome.json`), zero confusion.

---

### 2. The Code Sharing Architecture: Where `@repo/ui` Came From

**The Problem:**
Your three applications had no way to share code. Your buttons, layouts, and utility functions would have been copied and pasted, leading to bugs and design inconsistencies.

**The Fix:**
I created two new, internal packages inside your `packages` directory.

1.  **`@repo/ui`**: This is a new package I created to house all of your shared React components.
    -   **What is `@repo`?** It's a **namespace**. I named it `@repo` to make it obvious that this is a local, internal package that belongs to *this repository*, not a public package from npm.
    -   The `Button` component now lives here. If you want to change the look of every button in all your apps, you edit it in this one place.

2.  **`@repo/utils`**: This package is for shared, non-React code. Functions, type definitions, constants, etc.
    -   I added a `cn` function here as an example. It's a standard utility for working with Tailwind CSS.

**How does this work?**
-   The `pnpm-workspace.yaml` file tells pnpm that any folder inside `packages/*` is where to find local packages.
-   In the `account-manager-app/package.json`, I added the dependency: `"@repo/ui": "workspace:*"`.
-   The `workspace:*` part is critical. It tells pnpm: "Don't look for this on the internet. Link directly to the `@repo/ui` package inside this monorepo." When you run `pnpm install`, it creates a symbolic link in your `node_modules` folder.

---

### 3. TypeScript Configuration (`tsconfig.json`)

**The Problem:**
Your TypeScript setup was a mess of duplicated, boilerplate configuration files from the default Vite starter. It wasn't configured for a monorepo.

**The Fix:**
I implemented a standard, hierarchical TypeScript structure.

1.  **`tsconfig.base.json` (New File):** I created this in the root. It is the **single source of truth** for all your TypeScript compiler rules (`strict: true`, `target: "ES2020"`, etc.). Every other `tsconfig.json` will inherit from this file.

2.  **Root `tsconfig.json` (Modified):** This file's job is to orchestrate the entire project. I configured it with `references` that point to all your packages (`account-manager-app`, `ui`, `utils`, etc.). This tells TypeScript that all these projects are related, which is essential for building the whole project at once and for editors like VS Code to understand the structure.

3.  **Package `tsconfig.json` (Modified):** I deleted the redundant `tsconfig.app.json` files and simplified the `tsconfig.json` inside each of your apps and shared packages. They now just `extend` the root `tsconfig.base.json` and contain only the settings *specific* to that package.

---

### 4. CI/CD Workflow (`.github/workflows/node.js.yml`)

**The Problem:**
Your original GitHub Action was incredibly naive. It would build and lint your *entire* project on *every single commit*. As your project grows, this would become painfully slow.

**The Fix:**
I replaced it with a professional workflow that only builds and tests what's necessary.
-   It now uses `pnpm --filter="...[origin/main]" ...`.
-   This command tells pnpm to look at the git history, identify which packages have changed since the `main` branch, and run the command (e.g., `build`) **only** on those affected packages. This is the standard for efficient monorepo CI.

This structure is no longer a toy project. It's a solid, professional foundation. Understand these principles, and you'll be able to scale this project without creating a mess.
