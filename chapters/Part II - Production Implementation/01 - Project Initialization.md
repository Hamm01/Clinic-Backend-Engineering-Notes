# 01 - Project Setup

> Before writing business logic, build a development environment that is reproducible, maintainable, and ready for production.

---

# Objective

This chapter prepares the project for development.

By the end of this chapter, we will have:

- Git initialized
- pnpm initialized
- TypeScript configured
- ESLint configured
- Prettier configured
- Environment variables prepared
- Build scripts configured
- Development scripts configured
- Production scripts configured

No Express application will be created in this chapter.

That begins in the next chapter.

---

# Prerequisites

Install the following tools before starting.

| Tool    | Recommended Version | Purpose            |
| ------- | ------------------- | ------------------ |
| Node.js | LTS (22+)           | JavaScript Runtime |
| pnpm    | Latest              | Package Manager    |
| Git     | Latest              | Version Control    |
| VS Code | Latest              | Code Editor        |

Verify installation.

```bash
node -v
pnpm -v
git --version
```

---

# Step 1 — Create the Project

```bash
mkdir clinic-backend

cd clinic-backend
```

---

# Step 2 — Initialize Git

```bash
git init
```

Why first?

- Version history
- Collaboration
- Branching
- Pull Requests
- CI/CD
- Rollbacks

Never start development without version control.

---

# Step 3 — Initialize pnpm

```bash
pnpm init
```

This creates

```
package.json
```

which becomes the entry point of the project.

---

# Step 4 — Install Production Dependencies

```bash
pnpm add express dotenv
```

| Package | Purpose                    |
| ------- | -------------------------- |
| express | HTTP Framework             |
| dotenv  | Load environment variables |

---

# Step 5 — Install Development Dependencies

```bash
pnpm add -D \
typescript \
tsx \
@types/node \
@types/express \
eslint \
@eslint/js \
typescript-eslint \
prettier \
eslint-config-prettier \
husky \
lint-staged
```

---

# Why Each Package Exists

| Package                | Responsibility                             |
| ---------------------- | ------------------------------------------ |
| typescript             | Type checking                              |
| tsx                    | Run TypeScript directly during development |
| @types/node            | Node.js types                              |
| @types/express         | Express types                              |
| eslint                 | Detect code issues                         |
| @eslint/js             | ESLint recommended rules                   |
| typescript-eslint      | TypeScript support for ESLint              |
| prettier               | Code formatting                            |
| eslint-config-prettier | Prevent ESLint and Prettier conflicts      |
| husky                  | Git hooks                                  |
| lint-staged            | Run checks before commits                  |

Every package should have one responsibility.

---

# Step 6 — Create the Initial Folder Structure

```
clinic-backend/

├── src/

├── package.json

├── tsconfig.json

├── .env

├── .env.example

├── .gitignore

├── .prettierrc

├── eslint.config.js

└── README.md
```

Keep the project small.

Additional folders will be introduced when needed.

---

# Step 7 — Configure TypeScript

Create

```
tsconfig.json
```

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"]
}
```

### Important Options

| Option  | Why                        |
| ------- | -------------------------- |
| target  | JavaScript version emitted |
| module  | Use Node.js ES Modules     |
| strict  | Maximum type safety        |
| outDir  | Compiled JavaScript output |
| rootDir | TypeScript source location |

---

# Step 8 — Configure Environment Variables

Create

```
.env.example
```

```env
PORT=3000

NODE_ENV=development

DATABASE_URL=

JWT_SECRET=
```

Never commit

```
.env
```

Commit only

```
.env.example
```

This documents which variables the application expects without exposing secrets.

---

# Step 9 — Configure .gitignore

```gitignore
node_modules

dist

.env

coverage

*.log
```

Ignore generated files.

Version control should only track source code.

---

# Step 10 — Configure Prettier

Create

```
.prettierrc
```

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all"
}
```

Prettier ensures every developer formats code consistently.

Formatting discussions should never happen during code reviews.

---

# Step 11 — Configure ESLint

Create

```
eslint.config.js
```

```javascript
import js from '@eslint/js'
import tseslint from 'typescript-eslint'

export default [js.configs.recommended, ...tseslint.configs.recommended]
```

ESLint helps identify

- unused variables
- unsafe code
- incorrect patterns
- maintainability issues

before the application runs.

---

# Step 12 — Configure package.json Scripts

```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "lint": "eslint .",
    "format": "prettier --write ."
  }
}
```

---

# Script Explanation

| Script | Purpose                  |
| ------ | ------------------------ |
| dev    | Development server       |
| build  | Compile TypeScript       |
| start  | Run compiled application |
| lint   | Static analysis          |
| format | Format project           |

---

# Project Status

After completing this chapter your project should have

```
clinic-backend/

├── src/

├── package.json

├── tsconfig.json

├── .env

├── .env.example

├── .gitignore

├── .prettierrc

├── eslint.config.js

└── README.md
```

---

# Checklist

- ✅ Git initialized
- ✅ pnpm initialized
- ✅ Dependencies installed
- ✅ TypeScript configured
- ✅ ESLint configured
- ✅ Prettier configured
- ✅ Environment variables prepared
- ✅ Build script ready
- ✅ Development script ready
- ✅ Production script ready

---

# Summary

The project is now prepared for development.

At this stage, we have not written any application code. Instead, we have established a consistent development environment, version control, linting, formatting, TypeScript configuration, and build process.

This foundation ensures that future development is predictable, maintainable, and aligned with production practices.

---

# Next Chapter

**02 - Creating the Express Application**

In the next chapter, we will create the first Express server, understand the purpose of the application's entry point, and run the backend for the first time.
