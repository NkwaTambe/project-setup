# How to Create a New Shared Package

This guide provides the standard operating procedure for creating a new shared package within this monorepo. Following these steps ensures that any new library is correctly configured and integrated into the project.

## Step-by-Step Workflow

Let's say you want to create a new shared package named `forms`.

**1. Create the Directory**
First, create a new folder for your package inside the `packages` directory.

```bash
mkdir packages/forms
cd packages/forms
```

**2. Initialize `package.json`**
Use `pnpm init` to generate a boilerplate `package.json` file.

```bash
pnpm init
```

**3. Configure `package.json`**
The generated file is generic. You must edit it to match our project's standards. This is the most important step.

```json
{
  "name": "@repo/forms",
  "version": "0.0.0",
  "private": true,
  "exports": {
    "./input": "./src/Input/index.tsx",
    "./textarea": "./src/Textarea/index.tsx"
  },
  "scripts": {
    "lint": "biome check .",
    "build": "tsc"
  },
  "peerDependencies": {
    "react": "^19.1.1"
  },
  "devDependencies": {
    "typescript": "~5.8.3"
  }
}
```

**4. Add `tsconfig.json`**
Create a `tsconfig.json` file inside `packages/forms`. It should extend our base configuration. You can copy this from an existing package like `packages/ui`.

**5. Update Root `tsconfig.json`**
Finally, tell TypeScript that your new package exists by adding it to the `references` array in the main `tsconfig.json` at the root of the project.

```json
// in /tsconfig.json
"references": [
  // ... other packages
  { "path": "packages/forms" }
]
```

---

## Deep Dive: `package.json` Properties

### `"private": true`

**This is a safety feature.** By setting `"private": true`, you are telling `npm` and `pnpm` that this package is for internal use only and should **never** be published to a public registry like npmjs.com. Since `@repo/ui`, `@repo/utils`, and any future shared packages are specific to this project, this prevents costly and embarrassing accidental publications.

### `"exports"`

**This is the most important field for defining your package's public API.** It acts as a gatekeeper, controlling exactly which files within your package other developers are allowed to import.

**The Problem It Solves:**
Without the `"exports"` field, another developer could write this:

`import { ButtonView } from '@repo/ui/src/Button/Button.view.tsx';`

This is terrible. It creates a **tight coupling** to the internal file structure of your `@repo/ui` package. If you ever decide to refactor your files and move `Button.view.tsx`, you will break every single part of the codebase that imported it directly.

**How It Works:**
The `"exports"` field creates a clean, public-facing map to your internal files.

```json
"exports": {
  "./input": "./src/Input/index.tsx"
}
```

-   **The Key (`"./input"`):** This is the **public import path**. This is what the consumer of your package will use.
    `import { Input } from '@repo/forms/input';`

-   **The Value (`"./src/Input/index.tsx"`):** This is the **internal file path**. It maps the public path to the actual file that contains the code.

**The Benefit (Decoupling):**
This creates a **contract**. You are promising that anyone can get the `Input` component from `@repo/forms/input`. Now, you are free to completely refactor the inside of your `src` folder. You can move files, rename them, whatever you want. As long as you update the `"exports"` map to point to the correct new location, **no consumer of your package will break**.

It decouples the public API from the internal implementation, which is essential for creating maintainable, scalable code.
