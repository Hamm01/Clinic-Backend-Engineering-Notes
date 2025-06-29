# 06 - Error Handling

> "Good software doesn't avoid errors. It handles them predictably."

---

# Objective

No application is perfect.

Requests fail.

Databases become unavailable.

Tokens expire.

Validation fails.

Unexpected bugs appear.

The goal of error handling is **not to prevent every failure**.

The goal is to ensure that every failure is handled consistently, logged appropriately, and returned to the client in a safe and predictable format.

By the end of this chapter, we will:

- Understand why centralized error handling exists.
- Differentiate operational errors from programming errors.
- Create reusable custom error classes.
- Build a global Express error handler.
- Remove repetitive `try...catch` blocks.
- Integrate logging with error handling.
- Return consistent API responses.

This chapter establishes the error-handling foundation that every future feature will rely on.

---

# Why Error Handling Matters

Consider the following controller.

```ts
export async function createPatient(req, res) {
  const patient = await patientService.create(req.body)

  return res.status(201).json(patient)
}
```

Now imagine the database becomes unavailable.

```text
Controller

↓

Service

↓

Repository

↓

Database ❌
```

An exception is thrown.

Without proper error handling:

- The request may never receive a response.
- Express returns an inconsistent error.
- The client receives an unexpected payload.
- The error may never be logged.
- Debugging becomes difficult.

A production backend should never leave error handling to chance.

---

# The Goal

Regardless of where an error occurs,

the client should receive a predictable response.

Instead of:

```json
{
  "stack": "...",
  "message": "PrismaClientInitializationError..."
}
```

we should return:

```json
{
  "success": false,
  "message": "Internal Server Error"
}
```

The client needs useful information.

It does **not** need implementation details.

---

# Types of Errors

Before writing any code, we need to classify failures.

This is one of the most important concepts in backend development.

There are two broad categories.

---

## Operational Errors

These are expected failures.

Examples:

- Patient not found.
- Appointment already exists.
- JWT expired.
- Invalid credentials.
- Validation failed.
- Database temporarily unavailable.
- File upload exceeds limit.

The application should recover gracefully.

These are part of normal operation.

---

## Programming Errors

These are bugs in our code.

Examples:

```ts
const patient = undefined

patient.name
```

or

```ts
const age = req.body.age.toUpperCase()
```

when `age` is actually a number.

Programming errors indicate mistakes made during development.

They should be fixed, not hidden.

---

# Error Flow

Every request follows this lifecycle.

```text
Client

↓

Route

↓

Middleware

↓

Controller

↓

Service

↓

Repository

↓

Database

↑

Throws Error

↓

Global Error Handler

↓

Logger

↓

HTTP Response
```

Notice something important.

The error is **not** handled where it occurs.

It is allowed to travel upward until it reaches one centralized place.

This keeps the code clean and consistent.

---

# Problems with try...catch Everywhere

Many beginners write controllers like this.

```ts
export async function getPatient(req, res) {
  try {
    const patient = await patientService.findById(req.params.id)

    return res.json(patient)
  } catch (error) {
    logger.error(error)

    return res.status(500).json({
      message: 'Something went wrong'
    })
  }
}
```

Now imagine 80 controllers.

Every controller repeats the same pattern.

- `try`
- `catch`
- log
- return 500

This creates duplicated code and inconsistent responses.

---

# The Better Approach

Instead of handling every error inside the controller,

we allow errors to bubble upward.

```text
Controller

↓

Service

↓

Repository

↓

Throws Error

↓

Global Error Handler
```

Now the controller focuses only on HTTP.

The error handler focuses only on failures.

Each component has one responsibility.

---

# Error Handling as an Architectural Layer

Just like logging,

error handling is **cross-cutting**.

It is used by:

- Controllers
- Services
- Repositories
- Authentication
- Authorization
- Validation
- Background Jobs

Every module participates in error handling,

but only one module is responsible for converting errors into HTTP responses.

---

# Implementation Changes

## Files Created

```text
src/

├── errors/
│   ├── AppError.ts
│   ├── index.ts
│
└── middleware/
    ├── errorHandler.ts
    └── asyncHandler.ts
```

---

## Files Modified

```text
src/

├── app.ts
├── controllers/
└── services/
```

We will create these files one by one.

Each file has a single responsibility.

---

# Why Create an `errors/` Folder?

A common beginner mistake is to throw generic errors everywhere.

Example:

```ts
throw new Error('Patient not found')
```

This works,

but it does not tell Express:

- Which HTTP status should be returned?
- Is this an expected business error?
- Should it be logged as a warning or an error?
- Is it safe to expose the message to the client?

Instead,

we create custom application errors that carry structured information.

This allows the error handler to make intelligent decisions without every controller repeating the same logic.

---

# Next Step

The first file we'll build is the foundation of the entire error system:

```text
src/errors/AppError.ts
```

Every custom error in the application will extend this base class.

By defining one common structure now, we make authentication errors, validation errors, authorization errors, and business errors behave consistently throughout the project.

---

# Creating the Base Error Class

Create the following file.

```text
src/errors/AppError.ts
```

Add the implementation below.

```ts
export class AppError extends Error {
  public readonly statusCode: number
  public readonly isOperational: boolean

  constructor(message: string, statusCode: number, isOperational = true) {
    super(message)

    this.name = this.constructor.name
    this.statusCode = statusCode
    this.isOperational = isOperational

    Error.captureStackTrace?.(this, this.constructor)
  }
}
```

At first glance this looks like a normal JavaScript class.

In reality,

this becomes the **base class** for every expected error in our backend.

---

# Why Extend `Error`?

JavaScript already provides an `Error` class.

```ts
throw new Error('Patient not found')
```

The problem is that `Error` only knows

- message
- stack trace

It does **not** know

- HTTP status code
- whether the error is expected
- whether the message is safe to expose
- how the API should respond

Our application needs this extra information.

---

# Understanding Each Property

## `message`

```ts
message: string
```

The human-readable description.

Example

```text
Patient not found
```

---

## `statusCode`

```ts
statusCode: number
```

Express needs to know

```text
404

or

401

or

409

or

500
```

Instead of deciding this inside every controller,

the error already carries its own HTTP status.

---

## `isOperational`

```ts
isOperational: boolean
```

This is one of the most important properties.

It answers one question.

> Is this an expected application error?

Example

```text
Patient not found
```

Expected?

Yes.

---

Example

```text
JWT expired
```

Expected?

Yes.

---

Example

```text
Database temporarily unavailable
```

Expected?

Usually yes.

---

Example

```text
Cannot read property 'name' of undefined
```

Expected?

No.

That is a programming bug.

---

Later,

our global error handler will use this property to decide

- what to log
- what to return
- whether the application should continue

---

# Why Set the Error Name?

```ts
this.name = this.constructor.name
```

Without this,

every error appears simply as

```text
Error
```

Instead,

we get

```text
NotFoundError

AuthenticationError

ValidationError
```

This makes logs much easier to understand.

---

# Why Capture the Stack Trace?

```ts
Error.captureStackTrace?.(this, this.constructor)
```

Suppose an exception occurs.

Without a stack trace,

we only know

```text
Patient not found
```

With a stack trace,

we know

```text
PatientService.ts

↓

findPatient()

↓

Controller

↓

Route
```

The stack trace tells us exactly where the failure originated.

---

# The AppError Hierarchy

Every custom error will inherit from this class.

```text
                    Error
                      │
                      │
                AppError
        ┌────────┼────────┐
        │        │        │
        │        │        │
 Authentication  Validation  NotFound
      Error         Error       Error
        │
        │
 Authorization
      Error
```

Instead of repeatedly writing

```ts
throw new Error(...)
```

we create errors with meaning.

---

# Creating a Not Found Error

Create

```text
src/errors/NotFoundError.ts
```

```ts
import { AppError } from './AppError.js'

export class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404)
  }
}
```

Now instead of

```ts
throw new Error('Patient not found')
```

we write

```ts
throw new NotFoundError('Patient not found')
```

Notice what changed.

The controller doesn't decide

```text
404
```

The error itself already knows

```text
404
```

---

# Creating an Authentication Error

Create

```text
src/errors/AuthenticationError.ts
```

```ts
import { AppError } from './AppError.js'

export class AuthenticationError extends AppError {
  constructor(message = 'Authentication required') {
    super(message, 401)
  }
}
```

Example

```ts
throw new AuthenticationError('JWT token expired')
```

The error already knows

```text
401 Unauthorized
```

---

# Creating an Authorization Error

Create

```text
src/errors/AuthorizationError.ts
```

```ts
import { AppError } from './AppError.js'

export class AuthorizationError extends AppError {
  constructor(message = 'Access denied') {
    super(message, 403)
  }
}
```

Example

Doctor tries to delete another doctor's appointment.

```ts
throw new AuthorizationError("Cannot modify another doctor's appointment")
```

The client automatically receives

```http
403 Forbidden
```

---

# Creating a Validation Error

Create

```text
src/errors/ValidationError.ts
```

```ts
import { AppError } from './AppError.js'

export class ValidationError extends AppError {
  constructor(message = 'Validation failed') {
    super(message, 400)
  }
}
```

Example

```ts
throw new ValidationError('Email is required')
```

Instead of returning

```http
500 Internal Server Error
```

the API correctly returns

```http
400 Bad Request
```

---

# Export the Errors

Create

```text
src/errors/index.ts
```

```ts
export * from './AppError.js'
export * from './AuthenticationError.js'
export * from './AuthorizationError.js'
export * from './NotFoundError.js'
export * from './ValidationError.js'
```

Now every module imports errors from one place.

```ts
import { NotFoundError, ValidationError } from '../errors/index.js'
```

instead of importing individual files.

This follows the same pattern we used for `config/index.ts`.

---

# How Services Use These Errors

Imagine our clinic backend.

A doctor requests a patient profile.

The service looks like this.

```ts
import { NotFoundError } from '../errors/index.js'

export class PatientService {
  async getPatient(id: string) {
    const patient = await repository.findById(id)

    if (!patient) {
      throw new NotFoundError('Patient not found')
    }

    return patient
  }
}
```

Notice something important.

The service does **not**

- return HTTP responses
- call `res.status()`
- know anything about Express

It simply throws a meaningful business error.

This keeps the business layer independent from HTTP.

---

# Architecture Recap

At this point the request flow looks like this.

```text
Doctor

↓

GET /patients/123

↓

Controller

↓

PatientService

↓

Repository

↓

Patient Not Found

↓

throw NotFoundError()

↓

Controller receives exception

↓

(Next Chapter Section)

Global Error Handler
```

The service's responsibility ends when it throws the appropriate error.

The controller does not need to decide which HTTP status code should be returned.

The global error handler will take care of that for every request.

---

# The Problem with Async Controllers

At this point, our services throw meaningful errors.

However, Express has another challenge.

Consider the following controller.

```ts
export async function getPatient(req, res) {
  const patient = await patientService.getPatient(req.params.id)

  return res.json(patient)
}
```

If `patientService.getPatient()` throws an error,

who catches it?

Without additional handling,

Express may leave the request hanging or require repetitive `try...catch` blocks in every controller.

---

# The Wrong Approach

Many projects end up like this.

```ts
export async function getPatient(req, res) {
  try {
    const patient = await patientService.getPatient(req.params.id)

    return res.json(patient)
  } catch (error) {
    logger.error(error)

    return res.status(500).json({
      success: false,
      message: 'Internal Server Error'
    })
  }
}
```

Imagine

- 50 controllers
- 200 endpoints

Every controller repeats the same code.

This violates the **DRY (Don't Repeat Yourself)** principle.

---

# The Better Approach

Instead,

controllers should only contain business flow.

```text
HTTP Request

↓

Controller

↓

Service

↓

Repository

↓

Error

↓

Global Error Handler
```

The controller should not decide

- how to log
- how to build the error response
- which HTTP status code to return

One centralized component should do that.

---

# Creating an Async Handler

Create

```text
src/middleware/asyncHandler.ts
```

Add the following implementation.

```ts
import { Request, Response, NextFunction } from 'express'

export function asyncHandler(
  handler: (req: Request, res: Response, next: NextFunction) => Promise<unknown>
) {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(handler(req, res, next)).catch(next)
  }
}
```

This file is very small.

Yet,

it removes hundreds of repeated `try...catch` blocks from a large application.

---

# Understanding asyncHandler

Suppose our controller throws

```ts
throw new NotFoundError('Patient not found')
```

Without `asyncHandler`

```text
Controller

↓

Unhandled Promise Rejection

↓

Application Error
```

With `asyncHandler`

```text
Controller

↓

Promise Rejected

↓

next(error)

↓

Global Error Handler
```

Notice the difference.

The controller doesn't catch anything.

It simply throws.

The middleware forwards the error to Express.

---

# Updating the Controller

Before

```ts
export async function getPatient(req, res) {
  const patient = await patientService.getPatient(req.params.id)

  return res.json(patient)
}
```

After

```ts
import { asyncHandler } from '../middleware/asyncHandler.js'

export const getPatient = asyncHandler(async (req, res) => {
  const patient = await patientService.getPatient(req.params.id)

  return res.json(patient)
})
```

No `try...catch`.

The controller is now focused only on HTTP.

---

# Why Is This Better?

Compare the responsibilities.

Old Controller

```text
Receive Request

↓

Business Logic

↓

Handle Errors

↓

Log Errors

↓

Build Response
```

Too many responsibilities.

New Controller

```text
Receive Request

↓

Call Service

↓

Return Response
```

That's it.

Everything else belongs elsewhere.

---

# Creating the Global Error Handler

Now create

```text
src/middleware/errorHandler.ts
```

```ts
import { Request, Response, NextFunction } from 'express'

import { AppError } from '../errors/index.js'
import { logger, env } from '../config/index.js'

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  logger.error(
    {
      error,
      method: req.method,
      path: req.originalUrl
    },
    error.message
  )

  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      success: false,
      message: error.message
    })
  }

  return res.status(500).json({
    success: false,
    message:
      env.NODE_ENV === 'production' ? 'Internal Server Error' : error.message
  })
}
```

This becomes the **single place** responsible for converting exceptions into HTTP responses.

---

# Understanding the Global Error Handler

Every exception eventually arrives here.

Examples

```text
ValidationError

↓

AuthenticationError

↓

AuthorizationError

↓

NotFoundError

↓

Unexpected Error
```

Instead of each controller deciding what to return,

the error handler does it once.

---

# Logging Happens Here

Notice this section.

```ts
logger.error(
  {
    error,
    method: req.method,
    path: req.originalUrl
  },
  error.message
)
```

Why log here?

Because every unexpected failure eventually reaches this middleware.

We don't need every controller to log the same exception.

This avoids duplicate logs.

---

# Handling AppError

Expected business errors

```text
Patient not found

↓

404
```

JWT expired

```text
↓

401
```

Validation failed

```text
↓

400
```

Authorization failed

```text
↓

403
```

The error already knows

its HTTP status.

The middleware simply uses it.

---

# Handling Unexpected Errors

Now imagine this bug.

```ts
const patient = undefined

patient.name
```

This is not an operational error.

This is a programming bug.

The client should **not** receive

```text
Cannot read property 'name' of undefined'
```

Instead

Development

```json
{
  "success": false,
  "message": "Cannot read property 'name' of undefined"
}
```

Production

```json
{
  "success": false,
  "message": "Internal Server Error"
}
```

The real stack trace is preserved in the logs,

not exposed to the client.

---

# Registering the Error Handler

Update

```text
src/app.ts
```

Near the bottom of the file.

```ts
import { errorHandler } from './middleware/errorHandler.js'

/* Routes will be registered here */

app.use(errorHandler)
```

Why last?

Because Express executes middleware in order.

```text
Request

↓

Routes

↓

Controllers

↓

Error Handler
```

If the error handler is registered before the routes,

it will never receive route errors.

For this reason,

it is almost always the **last middleware** registered in the application.

---

# Architecture Recap

Our request lifecycle is now significantly cleaner.

```text
Doctor

↓

GET /patients/:id

↓

Route

↓

Controller

↓

PatientService

↓

Repository

↓

Database

↓

Patient Not Found

↓

throw NotFoundError()

↓

asyncHandler

↓

Global Error Handler

↓

Logger

↓

404 Response
```

Notice how each layer performs exactly one responsibility.

The controller never decides the HTTP status.

The service never creates HTTP responses.

The repository never logs business failures.

The error handler coordinates the final response for the client.

---

# Complete Error Flow

Let's revisit our clinic application.

A doctor requests a patient profile.

```http
GET /patients/pat_101
```

The request travels through the application.

```text
Doctor

↓

Route

↓

Authentication Middleware

↓

Controller

↓

PatientService

↓

PatientRepository

↓

Database
```

Now imagine the patient does not exist.

The repository returns

```ts
null
```

The service decides

```ts
throw new NotFoundError('Patient not found')
```

The controller does nothing.

The exception reaches

```text
asyncHandler

↓

Global Error Handler
```

The error handler responds

```http
404 Not Found
```

The complete lifecycle becomes

```text
Repository

↓

null

↓

Service

↓

throw NotFoundError()

↓

asyncHandler

↓

errorHandler

↓

logger.error()

↓

HTTP Response
```

Notice something.

The service never creates a response.

The repository never returns HTTP status codes.

The controller never contains duplicated `try...catch`.

Each layer performs exactly one job.

---

# Real Production Example

Let's follow a realistic request.

Doctor

```http
POST /appointments
```

Body

```json
{
  "patientId": "pat_102",
  "doctorId": "doc_45"
}
```

The service begins creating the appointment.

```ts
const patient = await patientRepository.findById(patientId)
```

Repository returns

```ts
null
```

Service

```ts
if (!patient) {
  throw new NotFoundError('Patient not found')
}
```

The controller never checks

```ts
if (!patient)
```

The controller simply calls the service.

The global middleware automatically converts the exception into

```http
404
```

Every endpoint now behaves consistently.

---

# Another Example — JWT Expired

Authentication middleware

```ts
try {
  const payload = verifyJwt(token)

  req.user = payload

  next()
} catch {
  throw new AuthenticationError('JWT expired')
}
```

Nobody writes

```ts
res.status(401)
```

inside the middleware.

Instead

```text
AuthenticationError

↓

Global Error Handler

↓

401 Response
```

Again,

one centralized place handles every failure.

---

# Another Example — Validation Failed

Validator

```ts
if (!email) {
  throw new ValidationError('Email is required')
}
```

Later

```text
ValidationError

↓

errorHandler

↓

400 Response
```

The validator never builds JSON responses.

It only reports the failure.

---

# Another Example — Unexpected Bug

Imagine a programming mistake.

```ts
const doctor = undefined

doctor.name
```

This throws

```text
TypeError
```

It is **not**

an `AppError`.

The global middleware reaches

```ts
return res.status(500).json({
  success: false,
  message: 'Internal Server Error'
})
```

Meanwhile

the complete stack trace is safely written into the logs.

The client receives

```json
{
  "success": false,
  "message": "Internal Server Error"
}
```

The developer receives everything needed to debug.

---

# Why We Don't Return Stack Traces

Never send this.

```json
{
  "stack": "...",
  "file": "PatientService.ts",
  "line": 82,
  "databasePassword": "...",
  "sql": "SELECT ..."
}
```

Imagine a malicious user.

They now know

- your file names
- project structure
- SQL queries
- technologies used

This is a security risk.

Production APIs should expose only what the client needs.

---

# Error Handling Responsibilities

| Layer                | Responsibility                                    |
| -------------------- | ------------------------------------------------- |
| Route                | Forward request                                   |
| Middleware           | Authentication, authorization, request processing |
| Controller           | Receive request and call service                  |
| Service              | Make business decisions and throw business errors |
| Repository           | Communicate with the database                     |
| Global Error Handler | Log the error and create the HTTP response        |

Notice that **only one layer** creates the final error response.

---

# Common Mistakes

## Returning HTTP Responses from Services

Wrong

```ts
return res.status(404).json({
  message: 'Patient not found'
})
```

Services should never know Express.

---

## Returning `null` for Business Failures

Wrong

```ts
return null
```

Now every controller must remember to check for `null`.

Instead

```ts
throw new NotFoundError('Patient not found')
```

One consistent approach.

---

## Using `throw new Error()`

Wrong

```ts
throw new Error('Unauthorized')
```

The error handler has no information about

- HTTP status
- error type
- whether it is operational

Always throw a meaningful application error.

---

## Logging the Same Error Multiple Times

Avoid this.

Repository

```text
Database failed
```

Service

```text
Database failed
```

Controller

```text
Database failed
```

Global Error Handler

```text
Database failed
```

Now one failure generates four identical logs.

A good rule is:

- Log business events where they occur.
- Log unexpected exceptions in the global error handler.
- Avoid duplicate logging of the same exception as it propagates upward.

---

# Senior Engineer Review

Before approving the error-handling implementation, verify:

### Architecture

- Are controllers thin?
- Do services throw meaningful errors?
- Is there only one global error handler?
- Is Express isolated from the business layer?

---

### Security

- Are stack traces hidden in production?
- Are sensitive details excluded from API responses?
- Are internal implementation details protected?

---

### Maintainability

- Are custom error classes used consistently?
- Are HTTP status codes centralized?
- Is duplicate `try...catch` code eliminated?

---

### Logging

- Are unexpected errors logged?
- Are errors logged only once?
- Can logs be correlated with requests?

---

# Files Created in This Chapter

```text
src/

├── errors/
│   ├── AppError.ts
│   ├── AuthenticationError.ts
│   ├── AuthorizationError.ts
│   ├── NotFoundError.ts
│   ├── ValidationError.ts
│   └── index.ts
│
└── middleware/
    ├── asyncHandler.ts
    └── errorHandler.ts
```

---

# Files Modified

```text
src/

├── app.ts
└── controllers/
```

---

# Project Status

```text
Backend Foundation

✅ Project Setup

✅ Express Application

✅ Environment Configuration

✅ Understanding the Project Structure

✅ Logging

✅ Error Handling

⬜ Database Connection

⬜ Prisma Setup

⬜ Authentication

⬜ Authorization

⬜ Validation

⬜ Business Modules

⬜ Background Jobs

⬜ Deployment
```

---

# Summary

Error handling is not about catching every exception where it occurs.

Instead, each layer has a clear responsibility:

- **Repositories** retrieve and persist data.
- **Services** make business decisions and throw meaningful application errors.
- **Controllers** coordinate HTTP requests and remain thin.
- **The global error handler** converts exceptions into consistent API responses and records unexpected failures.

By centralizing error handling, the application becomes easier to maintain, more secure, and more predictable for both developers and API consumers.

---

# Architecture Recap

The complete request lifecycle now looks like this.

```text
                HTTP Request
                     │
                     ▼
                 Express App
                     │
                     ▼
                  Middleware
                     │
                     ▼
                 Controller
                     │
                     ▼
                  Service
                     │
                     ▼
                Repository
                     │
                     ▼
                  Database
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     Success               Throws Error
          │                     │
          ▼                     ▼
     Controller           asyncHandler
          │                     │
          └──────────┬──────────┘
                     ▼
          Global Error Handler
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      logger.error()       HTTP Response
```

At this point, the backend has two essential production capabilities:

- **Observability** through structured logging.
- **Reliability** through centralized error handling.

Every feature we build from this point forward will plug into these two systems instead of implementing its own logging or error strategy.

---

# Next Chapter

## 07 - Database Connection

The backend is now structurally ready to communicate with a database.

In the next chapter we will:

- Connect the application to PostgreSQL.
- Introduce Prisma as the ORM.
- Create a singleton Prisma client.
- Understand connection pooling.
- Learn why repositories depend on Prisma instead of services.
- Build the database layer that every business module will use.
