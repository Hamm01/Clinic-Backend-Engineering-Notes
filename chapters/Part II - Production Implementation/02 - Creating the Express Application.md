# 02 - Creating the Express Application

> Every backend starts with one process.
>
> That process creates an HTTP server, listens for incoming requests, and delegates work to the rest of the application.

---

# Objective

In the previous chapter, we prepared the development environment.

The project now has

- TypeScript
- ESLint
- Prettier
- Build scripts
- Environment variables

Now we will create the first running backend.

By the end of this chapter we will

- Install Express
- Create the application's entry point
- Create the Express application
- Configure basic middleware
- Start the HTTP server
- Verify the backend is running

No business logic will be written yet.

---

# What is an Express Application?

An Express application is the central object that manages the backend.

It is responsible for

- Receiving HTTP requests
- Registering middleware
- Registering routes
- Sending HTTP responses

Every request entering the backend passes through this application.

---

# Project Structure

At this stage the project remains intentionally small.

```text
clinic-backend/

├── src/
│   ├── app.ts
│   └── server.ts
│
├── package.json
├── tsconfig.json
└── ...
```

Notice that we still haven't created

- controllers
- services
- repositories
- routes

Those folders appear only when needed.

---

# Why Two Files?

A common beginner question is

> Why not put everything inside `server.ts`?

Because two different responsibilities exist.

## server.ts

Responsible for

- Starting Node.js
- Listening on a port

## app.ts

Responsible for

- Creating the Express application
- Registering middleware
- Registering routes

Separating these responsibilities makes testing and maintenance easier.

---

# Step 1 — Create the Source Folder

```bash
mkdir src
```

---

# Step 2 — Create app.ts

```ts
import express from 'express'

const app = express()

app.use(express.json())

export default app
```

---

# Understanding app.ts

## Import Express

```ts
import express from 'express'
```

Imports the Express framework.

---

## Create Application

```ts
const app = express()
```

This creates the Express application instance.

Think of it as

```
Incoming Request

↓

Express Application

↓

Middleware

↓

Routes

↓

Response
```

Every request travels through this object.

---

## JSON Middleware

```ts
app.use(express.json())
```

This middleware automatically parses JSON request bodies.

Without it

```http
POST /patients
```

would produce

```ts
req.body === undefined
```

With it

```json
{
  "name": "Rahul",
  "age": 28
}
```

becomes

```ts
req.body
```

available throughout the request lifecycle.

Express already includes this middleware.

No additional package is required.

---

## Export the Application

```ts
export default app
```

We export the application because another file will start the server.

This keeps application creation independent from application startup.

---

# Step 3 — Create server.ts

```ts
import app from './app.js'

const PORT = Number(process.env.PORT) || 3000

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`)
})
```

---

# Understanding server.ts

## Import the Application

```ts
import app from './app.js'
```

Notice the

```text
.js
```

extension.

Even though the source file is

```
app.ts
```

Node.js executes compiled JavaScript.

When using ES Modules with TypeScript (`NodeNext`), importing with the `.js` extension is the recommended approach.

---

## Read the Port

```ts
const PORT = Number(process.env.PORT) || 3000
```

The application first checks

```
PORT
```

from the environment.

If it doesn't exist,

it falls back to

```
3000
```

This allows different environments to use different ports without changing code.

---

## Start the Server

```ts
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`)
})
```

This starts the Node.js process and begins listening for incoming HTTP requests.

At this point the backend is alive.

---

# Step 4 — Update package.json

Ensure the scripts section contains

```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

---

# Step 5 — Run the Server

```bash
pnpm dev
```

Expected output

```text
Server running on http://localhost:3000
```

---

# Step 6 — Verify the Server

Open your browser and visit

```text
http://localhost:3000
```

You will likely see

```text
Cannot GET /
```

This is expected.

The server is running, but no routes have been registered yet.

Receiving this response confirms that

- Node.js is running
- Express is running
- The server is listening correctly

---

# Request Flow

The current request lifecycle is very simple.

```text
Browser

↓

Node.js

↓

Express Application

↓

No Route Found

↓

404 Response
```

In future chapters we will insert

- middleware
- authentication
- authorization
- controllers
- services
- repositories

between the Express application and the final response.

---

# Common Mistakes

## Putting Everything in server.ts

Avoid mixing

- server startup
- middleware
- routes
- business logic

Keep responsibilities separate.

---

## Hardcoding Configuration

Wrong

```ts
const PORT = 3000
```

Prefer

```ts
const PORT = Number(process.env.PORT) || 3000
```

Configuration should come from the environment.

---

## Creating Business Folders Too Early

Do not create

```text
controllers/
services/
repositories/
```

before they are needed.

Allow the project structure to grow naturally.

---

## Ignoring ES Module Imports

When using `module: "NodeNext"`,

prefer importing local files with the `.js` extension.

This matches the generated JavaScript and avoids runtime issues.

---

# Senior Engineer Review

Before approving the initial Express setup, verify

- Is server startup separated from application configuration?
- Is configuration read from the environment?
- Is the application easy to test?
- Are responsibilities clearly separated?
- Does the server start successfully?
- Are only necessary files present?

If these questions are answered positively, the project is ready for the next stage.

---

# Project Status

```text
clinic-backend/

├── src/
│   ├── app.ts
│   └── server.ts
│
├── package.json
├── tsconfig.json
├── .env
├── .env.example
├── eslint.config.js
├── .prettierrc
└── ...
```

---

# Checklist

- ✅ Express application created
- ✅ Application exported
- ✅ Server entry point created
- ✅ JSON middleware configured
- ✅ Server listening on configured port
- ✅ Development server running

---

# Summary

The backend can now accept HTTP connections.

Although no APIs exist yet, we have established the application's entry point and separated the concerns of creating the Express application from starting the server.

This structure will remain stable as the application grows, with future middleware, routes, and business modules being added to the Express application without changing how the server itself starts.

---

# Next Chapter

**03 - Environment Configuration**

The next chapter will centralize configuration by loading environment variables, validating required settings, and creating a single configuration module that the rest of the application can safely use.
