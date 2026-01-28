# React + Vite Initialization Workflow

## Step 1: Ask Package Manager Preference

**Options:**

- **bun** - (Recommended) All-in-one runtime, fastest installs
- **npm** - Default bundled with Node.js
- **yarn** - Offline cache, stable, community
- **pnpm** - Symlinks, content-addressable storage
- **deno** - Secure runtime, native TypeScript

- Check, store <selected-package-manager> and <version> for use throughout workflow.

```bash
<selected-package-manager> --version
```

## Step 2: Create Vite Project

- Use current directory as <project-name>, store it for use throughout workflow.
- Use <selected-package-manager> to create vite project in current directory.
- Because Vite can't create project with not empty directory, we need to create it in <temp-project-name> directory. After that, we will copy all files from <temp-project-name> to <project-name> and remove <temp-project-name> directory.

  ```bash
  # npm | yarn | pnpm | bun
  <selected-package-manager> create vite <temp-project-name> --template react-ts --no-interactive
  # deno
  deno init --npm vite <temp-project-name> --template react-ts --no-interactive
  ```

## Step 3: Cleanup unnecessary files

- Standard HTML template title by format <Project Name>, not <project-name>
- Remove all eslint configuration files.
- Cleanup src folder, except **main.tsx, App.tsx**
- Update package.json
  - Update build script without typescript.
  - Remove all dependencies, devDependencies except **Vite, React, React DOM** and **NodeJS**
  - Add **React Compiler**, check npm version by `npm view <library-name> version`
  - Add "packageManager": "<selected-package-manager>@<version>"
- Create placeholder folders structure with .gitkeep files.

  ```
  public/
  ├── js/
  ├── images/
  ├── css/
  ├── fonts/
  ├── icons/
  └── favicon/
  src/
  ├── assets/
  ├── components/
  ├── constants/
  ├── contexts/
  ├── hocs/
  ├── hooks/
  ├── layouts/
  ├── pages/
  ├── services/
  ├── stores/
  ├── styles
  │   └── global.css
  ├── utils/
  ```

- Check and update boilerplate files by **rules/init-boilerplate.md**
  - **_IMPORTANT_**: when you ref boilerplate code, **must read `comment` in the boilerplate code** to understand the purpose of the code.
  - Remove `.gitkeep` file if folder has other content.
  - Thinking: slowly but surely. For this Step in this workflow, do not rush or miss any step.
- Use **jss skill** `/jss --react --format` to format code.
- Update README.md to done.
