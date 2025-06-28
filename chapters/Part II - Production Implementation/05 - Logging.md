# 05 - Logging

> "If you can't observe your application, you can't confidently operate it."

---

# Objective

Logging is one of the first infrastructure components added to a production backend.

Before implementing authentication, business modules, or database access, we need a reliable way to understand what the application is doing.

By the end of this chapter, we will:

- Understand why structured logging is essential.
- Replace `console.log()` with a production logger.
- Configure **Pino**.
- Create a reusable logger module.
- Integrate logging into the Express application.
- Learn where logging belongs within the application layers.
- Prepare the project for future request logging and error logging.

This chapter introduces the logging foundation that every subsequent chapter will build upon.

---

# Why Does Logging Exist?

Imagine a doctor reports that appointments are not being created.

Without logging, all we know is:

> "Something failed."

With logging, we can answer questions such as:

- Did the request reach the server?
- Which endpoint was called?
- Which authenticated doctor made the request?
- Which patient was involved?
- Which service failed?
- Which database query failed?
- How long did the request take?
- Was the failure caused by validation, business logic, or infrastructure?

Logging turns unknown failures into diagnosable events.

---

# Logging Throughout the Request Lifecycle

Every request travels through multiple layers.

```text
Client

↓

Express

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

↓

Repository

↓

Service

↓

Controller

↓

HTTP Response
```

Each layer may need to record useful information.

For example:

| Layer          | Example Log            |
| -------------- | ---------------------- |
| Server         | Application started    |
| Middleware     | Incoming request       |
| Authentication | Login succeeded        |
| Authorization  | Access denied          |
| Controller     | Creating appointment   |
| Service        | Business rule executed |
| Repository     | Database query failed  |
| Error Handler  | Unexpected exception   |

Notice that each layer logs **its own responsibility**.

A controller should not log SQL queries.

A repository should not log HTTP response codes.

Ownership remains separated.

---

# Why `console.log()` Is Not Enough

Many beginners start with:

```ts
console.log('Server started')
```

This is acceptable while experimenting.

It is not suitable for production systems.

Problems include:

- No timestamps
- No log levels
- No structured output
- Difficult to search
- Difficult to filter
- Difficult to aggregate
- Inconsistent formatting
- Poor integration with monitoring platforms

Now imagine hundreds of requests every second.

```
User logged in

Patient created

Database error

Appointment updated

Done

Error

Finished

Connected
```

Which request failed?

Which doctor caused it?

When did it happen?

How severe was the problem?

Plain text logs quickly become unusable.

---

# What Is Structured Logging?

Instead of writing plain text,

we write structured data.

Instead of

```text
Patient created successfully
```

we produce

```json
{
  "level": "info",
  "time": "2026-07-12T10:42:15Z",
  "module": "PatientService",
  "patientId": "pat_123",
  "doctorId": "doc_456",
  "message": "Patient created successfully"
}
```

Structured logs allow tools such as Elasticsearch, Grafana, Datadog, CloudWatch, and OpenSearch to search, filter, and analyze logs efficiently.

The log becomes data rather than just text.

---

# Choosing a Logging Library

Several logging libraries exist in the Node.js ecosystem.

| Library  | Notes                          |
| -------- | ------------------------------ |
| console  | Development only               |
| Winston  | Mature and highly configurable |
| Bunyan   | Older structured logger        |
| **Pino** | Fast, lightweight, JSON-first  |

For this project, we will use **Pino**.

---

# Why Pino?

Pino has become the preferred logger for many modern Node.js backends because it emphasizes performance and structured output.

Advantages include:

- Very fast
- Minimal overhead
- Native JSON logs
- Excellent TypeScript support
- Simple configuration
- Easy integration with Express
- Compatible with modern log aggregation platforms

Performance matters because every request may generate multiple log entries.

A slow logger can become a bottleneck.

---

# Logging Principles

Throughout this project, we will follow these principles.

## Log Meaningful Events

Good examples:

- Application started
- Database connected
- User authenticated
- Appointment created
- Prescription generated
- Background job completed

Avoid logging every line of code.

Logs should help explain system behaviour, not overwhelm it.

---

## Use Log Levels Correctly

Every log entry communicates importance.

We will primarily use five levels.

| Level   | Purpose                               |
| ------- | ------------------------------------- |
| `info`  | Normal application events             |
| `warn`  | Unexpected but recoverable situations |
| `error` | Failed operations                     |
| `debug` | Development diagnostics               |
| `fatal` | Application cannot continue           |

Choosing the correct level allows production systems to filter logs appropriately.

---

## Never Log Sensitive Information

Some data must never appear in logs.

Examples include:

- Passwords
- JWT tokens
- Refresh tokens
- OTP values
- Credit card details
- Session secrets
- Database passwords

Logs often leave the application and are stored centrally.

Treat them as production data.

---

# Installing Pino

Install the required packages.

```bash
pnpm add pino pino-http
```

### Why Two Packages?

| Package     | Responsibility                     |
| ----------- | ---------------------------------- |
| `pino`      | Core logger                        |
| `pino-http` | Express request logging middleware |

The core logger is used throughout the application.

The HTTP package integrates directly with Express.

---

# Project Changes

After installing the dependencies, we will create the following file.

```text
src/

├── config/
│   ├── env.ts
│   ├── index.ts
│   └── logger.ts   ← New
│
├── app.ts
├── server.ts
```

The logger belongs inside the `config` module because it is application infrastructure.

It is not tied to any business feature.

---

# Creating `src/config/logger.ts`

Create the file:

```text
src/config/logger.ts
```

Add the following implementation.

```ts
import pino from 'pino'
import { env } from './env.js'

export const logger = pino({
  level: env.NODE_ENV === 'production' ? 'info' : 'debug',

  transport:
    env.NODE_ENV === 'production'
      ? undefined
      : {
          target: 'pino-pretty',
          options: {
            colorize: true,
            translateTime: 'SYS:standard'
          }
        }
})
```

At first glance, this file is very small.

However, it becomes the single logging interface for the entire backend.

Every controller,

every service,

every repository,

every middleware,

and every background job

will import this logger instead of using `console.log()`.

---

# Understanding the Logger Configuration

Let's examine each part carefully.

## Import Pino

```ts
import pino from 'pino'
```

This imports the core logging library.

Unlike `console`, Pino creates structured log objects that can be consumed by production monitoring systems.

---

## Import Environment Configuration

```ts
import { env } from './env.js'
```

Notice that we never access `process.env` directly.

The logger receives all configuration from our centralized environment module.

This keeps infrastructure consistent across the application.

---

## Configure the Log Level

```ts
level: env.NODE_ENV === 'production' ? 'info' : 'debug',
```

Every log entry has a severity.

During development, we usually want more information.

```
debug

↓

info

↓

warn

↓

error

↓

fatal
```

In production, thousands of requests may be processed every minute.

Logging every debug message would generate enormous log files and increase storage costs.

Therefore,

Development

```text
NODE_ENV=development

↓

level = debug
```

Production

```text
NODE_ENV=production

↓

level = info
```

This means debug messages are automatically ignored in production.

---

## Configure the Transport

```ts
transport:
  env.NODE_ENV === 'production'
    ? undefined
    : {
        target: 'pino-pretty',
        options: {
          colorize: true,
          translateTime: 'SYS:standard',
        },
      },
```

Pino always produces JSON logs.

Example:

```json
{
  "level": 30,
  "time": 1752324901000,
  "pid": 8124,
  "hostname": "backend-server",
  "msg": "Server started"
}
```

Machines love this format.

Humans do not.

While developing, readable logs improve productivity.

Therefore we use

```
pino-pretty
```

only during development.

In production,

the application should emit raw JSON.

Log aggregation platforms such as

- Elasticsearch
- Grafana Loki
- Datadog
- AWS CloudWatch

expect structured JSON.

Never pretty-print production logs.

---

# Install pino-pretty

Install the development dependency.

```bash
pnpm add -D pino-pretty
```

Why is it a development dependency?

Because production servers should never spend CPU time formatting logs for humans.

Formatting belongs in development.

Production logs are consumed by machines.

---

# Export the Logger

Update

```text
src/config/index.ts
```

```ts
export * from './env.js'
export * from './logger.js'
```

Now every module imports infrastructure from one location.

Instead of

```ts
import { logger } from '../config/logger.js'
```

we can simply write

```ts
import { logger } from '../config/index.js'
```

As the project grows,

this file becomes the public interface of the configuration layer.

---

# Using the Logger During Startup

Update

```text
src/server.ts
```

Before

```ts
import app from './app.js'
import { env } from './config/index.js'

app.listen(env.PORT, () => {
  console.log(`Server running on http://localhost:${env.PORT}`)
})
```

After

```ts
import app from './app.js'
import { env, logger } from './config/index.js'

app.listen(env.PORT, () => {
  logger.info(`Server started on http://localhost:${env.PORT}`)
})
```

Notice what changed.

The application no longer writes directly to the console.

Everything flows through the logger.

This gives us one centralized logging system.

---

# Why Centralize Logging?

Imagine a backend with

- Authentication
- Appointments
- Prescriptions
- Payments
- Notifications
- Background Jobs

If every module uses

```ts
console.log(...)
```

every developer formats messages differently.

Examples

```text
Patient Created

patient created

PATIENT CREATED

Created Patient

Created patient successfully
```

Searching production logs becomes difficult.

Centralizing logging creates consistency.

---

# The Logger Becomes Infrastructure

Think of the logger exactly like the database connection.

The application creates one shared instance.

Every module consumes it.

```
Logger

↓

Controller

↓

Service

↓

Repository

↓

Jobs

↓

Middleware
```

Nobody creates a second logger.

Nobody configures logging independently.

There is one source of truth.

---

# Logging in Controllers

Example

```ts
import { logger } from '../config/index.js'

export async function createPatient(req, res) {
  logger.info('Creating patient')

  // Business logic...

  res.status(201).json({
    success: true
  })
}
```

What should controllers log?

Examples

- Incoming business operation
- Successful HTTP response
- Unexpected controller errors

Controllers should not log SQL statements.

That belongs elsewhere.

---

# Logging in Services

Example

```ts
import { logger } from '../config/index.js'

export class PatientService {
  async createPatient(data) {
    logger.info('Creating patient record')

    // Business logic...
  }
}
```

Service logs describe

business events.

Examples

- Appointment booked
- Consultation completed
- Prescription generated

The service explains **what happened in the business**.

---

# Logging in Repositories

Example

```ts
import { logger } from '../config/index.js'

export class PatientRepository {
  async findById(id: string) {
    logger.debug({ patientId: id }, 'Finding patient')

    // Prisma query...
  }
}
```

Repositories log

- database failures
- slow queries
- retry attempts
- connection issues

Repositories should never log HTTP status codes.

Those belong to controllers.

---

# Structured Context

Instead of

```ts
logger.info('Patient created')
```

prefer

```ts
logger.info(
  {
    patientId,
    doctorId,
    clinicId
  },
  'Patient created'
)
```

This produces structured metadata.

Later,

operations teams can search

```
doctorId = doc_812
```

or

```
clinicId = clinic_42
```

without parsing text.

Always log important identifiers as structured fields.

---

# Choosing Good Log Messages

Good

```
Appointment created
```

Better

```
Appointment created successfully
```

Best

```ts
logger.info(
  {
    appointmentId,
    patientId,
    doctorId
  },
  'Appointment created'
)
```

The message explains the event.

The structured object explains the context.

Both are important.

---

# Logging Philosophy

Think of every log entry as answering one question.

```
What happened?

↓

When?

↓

Where?

↓

Why?

↓

Which entity was involved?
```

If a log cannot answer those questions,

it probably needs improvement.

---

# Logging Every HTTP Request

Until now, we have logged application events manually.

```ts
logger.info('Server started')
```

However, every HTTP request should also be recorded automatically.

Instead of adding logging inside every controller,

we let Express log requests before they reach our application.

---

# Why Log Requests?

Imagine a doctor reports:

> "I clicked Save Appointment, but nothing happened."

Before debugging the code, the first question is:

> Did the request even reach the backend?

Request logging answers questions like:

- Which endpoint was called?
- Which HTTP method was used?
- What response status was returned?
- How long did the request take?
- Which IP made the request?

Without automatic request logging, these questions become difficult to answer.

---

# Using pino-http

`pino-http` is Express middleware.

Just like

```ts
app.use(express.json())
```

we can register

```ts
app.use(pinoHttp(...));
```

Every request now passes through the logger automatically.

---

# Update app.ts

Replace the current file.

```ts
import express from 'express'
import pinoHttp from 'pino-http'

import { logger } from './config/index.js'

const app = express()

app.use(
  pinoHttp({
    logger
  })
)

app.use(express.json())

export default app
```

---

# Understanding the Changes

## Import pino-http

```ts
import pinoHttp from 'pino-http'
```

This middleware integrates Pino with Express.

It automatically logs

- incoming requests
- completed responses
- response times

without writing manual logging code.

---

## Register Middleware

```ts
app.use(
  pinoHttp({
    logger
  })
)
```

This should be one of the first middleware registrations.

Why?

Because every request should be observed from the moment it enters the application.

Future middleware such as

- Authentication
- Authorization
- Validation

will also be included in these logs.

---

# What Does pino-http Log?

A request such as

```http
POST /patients
```

might generate a log similar to

```json
{
  "level": 30,
  "method": "POST",
  "url": "/patients",
  "statusCode": 201,
  "responseTime": 18,
  "msg": "request completed"
}
```

Notice something important.

We didn't write this log.

The middleware generated it automatically.

---

# Request Logging vs Business Logging

These two types of logs have different purposes.

## Request Logging

Answers

- Which endpoint?
- Which method?
- Which status?
- How long?

Example

```text
POST /appointments

201 Created

24 ms
```

---

## Business Logging

Answers

- Which patient?
- Which doctor?
- Which appointment?
- Which business rule?

Example

```text
Appointment confirmed

Doctor: doc_145

Patient: pat_551
```

Do not confuse infrastructure events with business events.

Both are valuable.

---

# Accessing the Logger Inside Requests

After registering `pino-http`,

every request receives its own logger.

```ts
req.log.info('Patient profile requested')
```

Why is this useful?

Because request-specific information can automatically be attached to every log generated during that request.

Later,

we'll extend this with

- Request IDs
- User IDs
- Correlation IDs

making distributed debugging much easier.

---

# What Should Never Be Logged?

Logging improves observability.

It must never compromise security.

Never log

❌ Passwords

```json
{
  "password": "MyPassword123"
}
```

---

❌ JWT Tokens

```json
{
  "token": "eyJhbGciOi..."
}
```

---

❌ Refresh Tokens

---

❌ OTP Codes

---

❌ API Secrets

---

❌ Database Passwords

---

❌ Full Credit Card Numbers

---

❌ Sensitive Medical Notes (unless explicitly required and properly protected)

Remember:

Logs are often stored outside your application.

Treat them as production data.

---

# Logging Personal Information

Our clinic system stores patient information.

Should we log

```text
Rahul Sharma
```

Usually,

No.

Prefer

```text
patientId = pat_841
```

Identifiers are more useful than personal data.

This also helps with privacy regulations and reduces unnecessary exposure.

---

# Logging Errors

Never ignore exceptions.

Instead of

```ts
catch (error) {
  console.error(error);
}
```

prefer

```ts
catch (error) {
  logger.error(error, 'Failed to create appointment');

  throw error;
}
```

The logger records

- stack trace
- error message
- timestamp
- severity

making production debugging significantly easier.

---

# Child Loggers

Large applications often create child loggers.

Example

```ts
const logger = rootLogger.child({
  module: 'PatientService'
})
```

Now every log automatically includes

```json
{
  "module": "PatientService"
}
```

without repeating it manually.

As the project grows,

child loggers help identify which module produced each log entry.

For this handbook,

we will introduce child loggers later when the application contains multiple business modules.

---

# Logging Best Practices

## Log Events, Not Every Line

Good

```text
Appointment created
```

Bad

```text
Entered function

Variable assigned

Checking condition

Calling repository

Repository returned

Leaving function
```

Too much logging becomes noise.

---

## Include Useful Context

Prefer

```ts
logger.info(
  {
    appointmentId,
    doctorId,
    patientId
  },
  'Appointment created'
)
```

instead of

```ts
logger.info('Appointment created')
```

Context makes logs actionable.

---

## Keep Messages Consistent

Choose one style.

For example

```text
Appointment created

Appointment updated

Appointment cancelled
```

Avoid mixing

```text
Created Appointment

Appointment Done

Update Successful
```

Consistency improves searching and dashboards.

---

## Use the Correct Log Level

Do not log everything as

```ts
logger.error(...)
```

An expected validation failure is not an application error.

Reserve `error` and `fatal` for genuine operational failures.

---

# Common Mistakes

The logging system is now integrated into the application.

Before moving forward, let's review the mistakes commonly seen in production projects.

---

## Mistake 1 — Using `console.log()` Everywhere

Bad

```ts
console.log('Patient Created')
```

Problems

- No log levels
- No timestamps
- No request context
- Difficult to search
- Difficult to filter
- Difficult to aggregate

Instead

```ts
logger.info('Patient created')
```

Every log should pass through the application's logger.

---

## Mistake 2 — Logging Sensitive Information

Never log

```json
{
  "password": "Password@123",
  "jwt": "eyJhbGciOi...",
  "refreshToken": "...",
  "otp": "456789"
}
```

Even if logs are internal,

they may eventually reach

- CloudWatch
- Datadog
- Grafana
- Elasticsearch
- SIEM platforms

Treat logs as production data.

---

## Mistake 3 — Logging Entire Request Objects

Avoid

```ts
logger.info(req)
```

or

```ts
logger.info(req.body)
```

A request may contain

- passwords
- uploaded files
- authorization headers
- cookies
- personal information

Instead, log only the fields required for debugging.

Example

```ts
logger.info(
  {
    patientId,
    doctorId,
    appointmentId
  },
  'Appointment requested'
)
```

---

## Mistake 4 — Logging the Same Event Multiple Times

Example

Controller

```ts
logger.info('Creating patient')
```

Service

```ts
logger.info('Creating patient')
```

Repository

```ts
logger.info('Creating patient')
```

Now one business operation produces three identical logs.

Instead,

each layer should log **its own responsibility**.

Controller

```text
Incoming HTTP Request
```

Service

```text
Business operation started
```

Repository

```text
Database query executed
```

Different responsibilities.

Different logs.

---

## Mistake 5 — Logging Every Variable

Bad

```ts
logger.debug(user)

logger.debug(password)

logger.debug(counter)

logger.debug(data)

logger.debug(result)

logger.debug(response)
```

More logs do not automatically improve debugging.

Good logging records

- important events
- failures
- state transitions

not every variable.

---

## Mistake 6 — Incorrect Log Levels

Wrong

```ts
logger.error('Patient profile opened')
```

Nothing failed.

Correct

```ts
logger.info('Patient profile viewed')
```

Use log levels intentionally.

| Event               | Level |
| ------------------- | ----- |
| Application started | info  |
| Patient searched    | info  |
| Invalid input       | warn  |
| Database timeout    | error |
| Application crash   | fatal |

---

# Where Should Logging Be Used?

The following table summarizes the responsibility of each layer.

| Layer           | Should Log? | What Should Be Logged?                                                                                                |
| --------------- | :---------: | --------------------------------------------------------------------------------------------------------------------- |
| Server          |     ✅      | Startup, shutdown                                                                                                     |
| Middleware      |     ✅      | Incoming requests, response time                                                                                      |
| Authentication  |     ✅      | Login success/failure                                                                                                 |
| Authorization   |     ✅      | Access denied                                                                                                         |
| Controller      |     ✅      | HTTP operations                                                                                                       |
| Service         |     ✅      | Business events                                                                                                       |
| Repository      |     ✅      | Database failures, slow queries                                                                                       |
| Background Jobs |     ✅      | Job execution                                                                                                         |
| Validators      | Usually No  | Validation failures are typically returned to the client rather than individually logged unless auditing is required. |

Remember,

logging should describe

**what happened**

not

**every line of code executed**.

---

# Senior Engineer Review

Before approving the logging implementation, a senior backend engineer will usually verify the following.

## Architecture

- Is logging centralized?
- Is there only one logger instance?
- Does every module use the same logger?

---

## Security

- Are secrets excluded?
- Are passwords excluded?
- Are JWTs excluded?
- Is personally identifiable information protected?

---

## Maintainability

- Are log messages consistent?
- Are log levels used correctly?
- Are logs structured?

---

## Production Readiness

- JSON logs in production
- Pretty logs only during development
- Request logging enabled
- Errors logged consistently

---

# Final Project Structure

After completing this chapter, the project should resemble:

```text
clinic-backend/

├── src/
│
├── config/
│   ├── env.ts
│   ├── logger.ts
│   └── index.ts
│
├── controllers/
├── middleware/
├── repositories/
├── routes/
├── services/
├── validators/
├── models/
├── jobs/
├── utils/
├── types/
│
├── app.ts
└── server.ts
│
├── package.json
├── tsconfig.json
├── .env
└── .env.example
```

---

# Files Modified in This Chapter

## New Files

```text
src/config/logger.ts
```

---

## Updated Files

```text
src/config/index.ts

src/app.ts

src/server.ts

package.json
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

⬜ Error Handling

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

Logging is one of the earliest infrastructure components added to a production backend because every other part of the application depends on it.

By introducing a centralized logger, we gain:

- Consistent log formatting
- Structured JSON output
- Configurable log levels
- Automatic request logging
- Better debugging
- Better production monitoring
- Easier integration with observability platforms

Throughout the remainder of this project,

every controller,

every service,

every repository,

every middleware,

and every background job

will use this logger.

From this point onward,

`console.log()` should no longer appear anywhere in the application.

---

# What We Achieved

At this point, our backend has:

- A production-ready logging system.
- Structured logs for application events.
- Automatic HTTP request logging.
- Environment-based log configuration.
- A centralized logger shared across all modules.
- A foundation for future monitoring and debugging.

The application is now observable, making future development and production support significantly easier.

---

# Next Chapter

## 06 - Error Handling

Now that the application can record what happens, the next step is to handle failures consistently.

Instead of allowing errors to propagate unpredictably, we will build a centralized error-handling strategy that:

- Produces consistent API responses.
- Logs unexpected failures.
- Separates operational errors from programming errors.
- Prevents sensitive implementation details from leaking to clients.

This error-handling layer will become the foundation for all future business modules.
