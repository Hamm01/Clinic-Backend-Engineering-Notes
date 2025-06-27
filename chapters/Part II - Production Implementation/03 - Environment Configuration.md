# 03 - Environment Configuration

> Configuration is not business logic.
>
> A backend should know **where** to read configuration, but it should never hardcode it.

---

# Objective

Until now we have

- Created the project
- Installed dependencies
- Created the Express application

Now we will configure the application.

By the end of this chapter we will

- Create the production folder structure
- Configure environment variables
- Validate required configuration
- Create a single configuration module
- Remove direct usage of `process.env`

---

# Why Configuration Matters

Imagine these environments.

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Should they all use the same

- Database
- JWT Secret
- Port
- API Keys

No.

Each environment has different configuration.

Configuration belongs outside the application.

---

# Updated Project Structure

From this chapter onward our project will follow this structure.

```text
clinic-backend/

├── src/
│
├── config/
│   ├── env.ts
│   └── index.ts
│
├── routes/
├── controllers/
├── services/
├── repositories/
├── middleware/
├── validators/
├── models/
├── utils/
├── types/
├── jobs/
│
├── app.ts
├── server.ts
│
├── package.json
├── tsconfig.json
├── .env
├── .env.example
└── ...
```

Notice something.

Most folders are still empty.

That is perfectly acceptable.

The architecture comes first.

Implementation fills the folders gradually.

---

# Step 1 — Install Zod

We will validate our environment variables using Zod.

```bash
pnpm add zod
```

Why?

Without validation

```text
DATABASE_URL = undefined
```

may not fail until the application makes its first database query.

Instead,

we want the application to fail immediately during startup.

This is known as **Fail Fast**.

---

# Step 2 — Update .env.example

```env
PORT=3000

NODE_ENV=development

DATABASE_URL=

JWT_SECRET=
```

This file documents every required environment variable.

It should always be committed.

---

# Step 3 — Create .env

```env
PORT=3000

NODE_ENV=development

DATABASE_URL=postgresql://postgres:password@localhost:5432/clinic

JWT_SECRET=replace-with-a-long-random-secret
```

Never commit this file.

It contains secrets.

---

# Step 4 — Create config/env.ts

```ts
import 'dotenv/config'
import { z } from 'zod'

const envSchema = z.object({
  PORT: z.coerce.number().default(3000),

  NODE_ENV: z.enum(['development', 'test', 'production']),

  DATABASE_URL: z.string().min(1),

  JWT_SECRET: z.string().min(32)
})

export const env = envSchema.parse(process.env)
```

---

# Understanding env.ts

## Load dotenv

```ts
import 'dotenv/config'
```

Loads `.env` before anything else executes.

---

## Define the Schema

```ts
const envSchema = ...
```

The schema defines

- required variables
- variable types
- default values
- validation rules

---

## Parse

```ts
envSchema.parse(process.env)
```

If any variable is missing,

the application immediately exits with a descriptive error.

No partially configured application should start.

---

# Step 5 — Create config/index.ts

```ts
export * from './env.js'
```

Why?

Instead of importing

```ts
import { env } from './config/env.js'
```

everywhere,

we import

```ts
import { env } from './config/index.js'
```

Later,

additional configuration modules

- logger
- database
- redis

can also be exported from the same location.

---

# Step 6 — Update server.ts

Before

```ts
const PORT = Number(process.env.PORT) || 3000
```

After

```ts
import app from './app.js'
import { env } from './config/index.js'

app.listen(env.PORT, () => {
  console.log(`Server running on http://localhost:${env.PORT}`)
})
```

Notice

`process.env`

has disappeared.

Only one file in the project knows how configuration is loaded.

---

# Why Avoid process.env Everywhere?

Imagine

```text
Authentication

↓

process.env.JWT_SECRET
```

Another file

```text
Database

↓

process.env.DATABASE_URL
```

Another file

```text
Jobs

↓

process.env.NODE_ENV
```

Soon,

configuration is scattered throughout the application.

Instead,

everything imports

```ts
env
```

from one location.

This centralizes configuration and improves maintainability.

---

# Common Mistakes

## Using process.env Directly

Wrong

```ts
process.env.JWT_SECRET
```

Preferred

```ts
env.JWT_SECRET
```

---

## Skipping Validation

Never assume environment variables exist.

Validate them at startup.

---

## Committing .env

Commit

```text
.env.example
```

Never commit

```text
.env
```

---

## Loading dotenv in Multiple Files

Load configuration once.

Every other module should consume the exported configuration object.

---

# Senior Engineer Review

Before approving environment configuration, verify

- Is configuration separated from business logic?
- Is every required variable validated?
- Does the application fail fast?
- Is `process.env` centralized?
- Is `.env` excluded from version control?
- Is `.env.example` up to date?

---

# Project Status

```text
clinic-backend/

├── src/
│   ├── config/
│   │   ├── env.ts
│   │   └── index.ts
│   │
│   ├── routes/
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   ├── validators/
│   ├── models/
│   ├── utils/
│   ├── types/
│   ├── jobs/
│   │
│   ├── app.ts
│   └── server.ts
│
├── .env
├── .env.example
├── package.json
└── tsconfig.json
```

---

# Checklist

- ✅ Environment variables documented
- ✅ `.env` created
- ✅ Configuration validated
- ✅ Typed configuration exported
- ✅ `process.env` removed from application code
- ✅ Project folder structure established

---

# Summary

Configuration is infrastructure, not business logic.

By loading and validating environment variables in one place, the application becomes safer, easier to maintain, and easier to deploy across different environments.

Every future module—database, authentication, caching, logging, and background jobs—will consume the same configuration object rather than interacting with `process.env` directly.

---

# Next Chapter

**04 - Project Structure**

Now that the project skeleton exists, we will implement the first infrastructure modules and explain the responsibility of every folder before building business features.
