# ⚡ Build and Use Your Own Next.js Starter Kit via npx

Tired of setting up the same project structure again and again? This guide walks you through creating a custom React/Next.js starter kit and installing it anywhere using `npx`. We’ll build a TypeScript CLI that copies a template, supports custom commands (routes/layouts), plays nicely with private packages, and can be versioned via Git tags.

### 📝 Disclaimer  
> **This content was generated with the assistance of AI. Please conduct your own due diligence before applying any information presented here.**

## 🧠 Basic Starter Application

We’ll build a project structured like this:

```
create-vivek-next-app/
├── bin/
│   └── index.ts        ← Main TypeScript CLI logic
├── template/           ← Your React starter kit
├── tsconfig.json
├── package.json
```

### 1️⃣ Set Up the Project

```bash
mkdir create-vivek-next-app
cd create-vivek-next-app
npm init -y
npm install fs-extra
npm install -D typescript ts-node @types/node @types/fs-extra
npx tsc --init
```

Update `tsconfig.json` to ensure the CLI will run properly:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "./",
    "outDir": "./dist",
    "esModuleInterop": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "strict": true
  },
  "include": ["bin/**/*"]
}
```

### 2️⃣ Add the CLI Code in TypeScript

**File:** `bin/index.ts`

```ts
#!/usr/bin/env ts-node

import path from "path";
import fs from "fs-extra";
import { execSync } from "child_process";

async function main() {
  const [, , projectName = "my-app"] = process.argv;
  const currentDir = process.cwd();
  const templateDir = path.join(__dirname, "..", "template");
  const destination = path.join(currentDir, projectName);

  console.log(`\n📁 Creating project in: ${destination}`);

  try {
    await fs.copy(templateDir, destination);
    console.log("✅ Template files copied.");

    process.chdir(destination);
    console.log("\n📦 Installing dependencies...\n");
    execSync("npm install", { stdio: "inherit" });

    console.log(`\n🚀 Project setup complete!\n`);
    console.log(`👉 Next steps:\n  cd ${projectName}\n  npm start\n`);
  } catch (err) {
    console.error("❌ Error setting up project:", err);
  }
}

main();
```

### 3️⃣ Add Your React App Template

Put your custom starter (e.g., a Vite or CRA app) inside the `template/` directory. For example:

```
template/
├── public/
├── src/
├── package.json
└── README.md
```

You can customize the structure however you like — e.g., with Tailwind, ESLint, etc.

### 4️⃣ Configure `package.json`

Update the `bin` field to run your CLI:

```json
{
  "name": "create-vivek-next-app",
  "version": "1.0.0",
  "bin": {
    "create-vivek-next-app": "bin/index.ts"
  },
  "scripts": {
    "dev": "ts-node bin/index.ts"
  },
  "type": "commonjs",
  "dependencies": {
    "fs-extra": "^11.2.0"
  },
  "devDependencies": {
    "@types/node": "^20.11.30",
    "ts-node": "^10.9.2",
    "typescript": "^5.4.5"
  }
}
```

### 5️⃣ Push to GitHub

```bash
git init
git remote add origin https://github.com/your-username/create-vivek-next-app.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

### Test Locally

Run it locally first :

```bash
npx ts-node bin/index.ts test-app
```

Or, once pushed to GitHub:

```bash
npx github:your-username/create-next-starter my-app
```

## Adding Custom Commands

Now, lets create a `create-route` CLI command that follows our folder structure.
Let’s make it so you can run something like:

```sh
npx create-vivek-next-app create-route blog
npx create-vivek-next-app create-layout app-layout
```

We’ll now have **three separate scripts**:

1. `createProject.ts` → Creates the Next.js project from your `template/` folder.
2. `createRoute.ts` → Adds a new route based on the route structure already present inside the template.
3. `createLayout.ts` → Adds a new layout file/structure (from template or minimal boilerplate).

Here’s the updated folder structure I recommend:

```
/scripts
  ├── createProject.ts
  ├── createRoute.ts
  ├── createLayout.ts
  ├── index.ts
/template
  ├── app
  │   ├── layout.tsx
  │   ├── page.tsx
  │   └── ...
  └── components
      └── ...
package.json
```

### **scripts/createProject.ts**

```ts
#!/usr/bin/env ts-node

import path from "path";
import fs from "fs-extra";
import { execSync } from "child_process";

export async function createProject(projectName: string) {
  const currentDir = process.cwd();
  const templateDir = path.join(__dirname, "..", "template");
  const destination = path.join(currentDir, projectName);

  console.log(`\n📁 Creating Next.js app in: ${destination}`);

  try {
    await fs.copy(templateDir, destination);
    console.log("✅ Template files copied.");

    process.chdir(destination);
    console.log("\n📦 Installing dependencies...\n");
    execSync("npm install", { stdio: "inherit" });

    console.log(`\n🚀 Project setup complete!\n`);
    console.log(`👉 Next steps:\n  cd ${projectName}\n  npm run dev\n`);
  } catch (err) {
    console.error("❌ Error setting up project:", err);
  }
}
```

### **scripts/createRoute.ts**

```ts
import path from "path";
import fs from "fs-extra";

export async function createRoute(routeName: string) {
  const currentDir = process.cwd();
  const routeTemplateDir = path.join(
    __dirname,
    "..",
    "template",
    "app",
    "(app)",
    "home",
  );

  console.log(routeTemplateDir);

  const appDir = path.join(currentDir, "app", routeName);

  if (!fs.existsSync("package.json")) {
    console.error("❌ Not inside a project folder.");
    process.exit(1);
  }

  if (!fs.existsSync(routeTemplateDir)) {
    console.error("❌ route-template folder not found in template.");
    process.exit(1);
  }

  console.log(`\n📂 Creating new route: ${routeName}`);
  await fs.copy(routeTemplateDir, appDir);

  console.log("✅ Route created successfully!");
}
```

> **Note:** You just create a dummy route inside `template/app/example-route/` and we reuse it for all new routes.

### **scripts/createLayout.ts**

```ts
import path from "path";
import fs from "fs-extra";

export async function createLayout(layoutName: string) {
  const projectDir = process.cwd();
  const layoutTemplateDir = path.join(
    __dirname,
    "..",
    "template",
    "app",
    "(app)",
  );
  const newLayoutDir = path.join(projectDir, "app", `(${layoutName})`);

  try {
    if (!(await fs.pathExists(layoutTemplateDir))) {
      throw new Error(
        "Layout template not found in template/app/example-layout",
      );
    }

    await fs.copy(layoutTemplateDir, newLayoutDir);
    console.log(`✅ New layout '${layoutName}' created in app/${layoutName}`);
  } catch (err) {
    console.error("❌ Error creating layout:", err);
  }
}
```

### **index.ts (CLI entrypoint)**

```ts
#!/usr/bin/env ts-node

import { createLayout } from "./create-layout";
import { createProject } from "./create-project";
import { createRoute } from "./create-route";

async function main() {
  const [, , command, arg] = process.argv;

  if (!command) {
    // Default behaviour: create a project
    await createProject(command);
  } else if (command === "create-route") {
    if (!arg) {
      console.error("❌ Please provide a route name.");
      process.exit(1);
    }
    await createRoute(arg);
  } else if (command === "create-layout") {
    if (!arg) {
      console.error("❌ Please provide a layout name.");
      process.exit(1);
    }
    await createLayout(arg);
  } else {
    // If first arg isn't a command, treat it as project name
    await createProject(command);
  }
}

main();
```

## Handling Private Packages

If your template has a dependency on a **private GitHub Package Registry library**, then `npm install` will fail for anyone who doesn’t have a valid **Personal Access Token (PAT)** configured for GitHub.

### **Remove the dependency from `package.json`**

- **Pros:** Installation never fails, works for everyone.
- **Cons:** Users must manually install the private package themselves after setting up authentication.
- **How:**
  - Remove the private package from `package.json`.
  - Add setup instructions in your README like:

    ```bash
    npm install @your-org/your-package
    ```

    after configuring their `.npmrc` with:

    ```
    //npm.pkg.github.com/:_authToken=YOUR_GITHUB_TOKEN
    @your-org:registry=https://npm.pkg.github.com
    ```

## 🚀 Versioning and Installing Your GitHub-based Project Template with `npx`

When you build a project template—like a custom **Next.js starter**—you often want to make it easy for others to install it using a simple one-liner like:

```bash
npx github:username/repo-name
```

But here’s where things get tricky:

- What if **Next.js 15** comes out and you want to upgrade your template?
- What if some users still want the **Next.js 14** version?

In the **npm world**, versioning is handled automatically (`npx create-something@1.0.0`), but with GitHub installs, the workflow is different.

Let’s break it down.

### **1️⃣ How `npx` Works with npm Packages vs GitHub**

### `npx` with npm Packages

When you run:

```bash
npx create-next-app@14.0.0
```

- `npx` downloads the **exact version** of the package from the npm registry.
- Only the published package files (based on `files` field or `.npmignore`) are downloaded, not the entire repo.
- Versioning is built-in via npm.

### `npx github:`

When you run:

```bash
npx github:username/repo-name
```

- `npx` downloads the **entire GitHub repository** (zip of the branch or tag).
- There’s **no npm versioning**, so you can’t do `@1.0.0` here.
- The default is `main` (or the repo’s default branch) unless you specify a **branch, commit, or tag**:

```bash
npx github:username/repo-name#v1.0.0
```

### **2️⃣ Why npm Versioning Doesn’t Work with GitHub Installs**

npm versioning works because the npm registry **stores published versions** of your package and serves them on demand.
GitHub doesn’t maintain versions in the same way for `npx`. You need to use **tags** or branches to simulate versioning.

### **3️⃣ Using Git Tags for Versioning Your Template**

To allow users to install older or newer versions of your GitHub template:

1. **Find the commit you want to tag**
   - Go to the GitHub repo
   - Navigate to **Commits**
   - Click on the commit you want
   - Copy the commit hash (optional if tagging latest)

2. **Create a tag (UI way)**
   - Go to the repo’s **Releases** tab
   - Click **Draft a new release**
   - Set a **tag name** (e.g., `v1.0.0`)
   - Optionally link it to a specific commit
   - Publish the release

3. **Installing a specific version**

```bash
npx github:username/repo-name#v1.0.0
```

4. **Deleting a tag**
   - From **Releases** page, delete the release and associated tag if no longer needed.

### **4️⃣ Example: Installing Different Versions**

Here’s how users can install different versions of your Next.js template:

```bash
# Install Next.js 14 version
npx github:username/create-vivek-next-app#v1.0.0

# Install Next.js 15 version
npx github:username/create-vivek-next-app#v2.0.0
```

## 🚀 Future Enhancements

* Convert the project into a fully installable **npm package**
* Add CLI support using the `bin` field to enable execution via `npx`
* Publish the package to **npm registry** and/or **GitHub Packages**
* Introduce versioned releases for better stability and upgrade control
* Support scaffolding new projects using a command like:

  ```bash
  npx @your-scope/create-app
  ```
* Add plugin-based architecture to extend functionality without modifying core code
* Improve template customization (flags for TypeScript, ESLint, Tailwind, etc.)
