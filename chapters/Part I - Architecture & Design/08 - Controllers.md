# 08 - Controllers

> A controller translates an HTTP request into a business request.

---

# Objective

Once a request has been authenticated, authorized, and validated, it reaches the **Controller**.

The controller is the first layer that belongs to the application.

Its responsibility is simple:

- Receive the HTTP request.
- Extract required information.
- Call the appropriate business service.
- Return an HTTP response.

A controller should never become the place where business decisions are implemented.

---

# Where Does the Controller Fit?

The request lifecycle now looks like this.

```
Client

↓

Route

↓

Authentication

↓

Authorization

↓

Validation

↓

Controller

↓

Service

↓

Repository

↓

Database
```

The controller sits between HTTP and the business layer.

It connects the outside world to the application.

---

# Why Do Controllers Exist?

Imagine Express without controllers.

```
Route

↓

Business Logic

↓

Database Queries

↓

Response
```

Very quickly,

routes become hundreds of lines long.

Different endpoints begin duplicating logic.

Testing becomes difficult.

Controllers separate **HTTP handling** from **business logic**.

---

# Controller Responsibilities

A controller should only perform responsibilities related to HTTP.

These include

- Reading route parameters
- Reading query parameters
- Reading request body
- Reading authenticated user
- Calling the correct service
- Returning an HTTP response

Nothing more.

---

# Controller in Our Clinic Application

Example

Doctor opens a patient profile.

```
GET /patients/:patientId
```

The controller performs the following steps.

```
Receive Request

↓

Read patientId

↓

Read authenticated doctor

↓

Call PatientService

↓

Return Response
```

Notice something.

The controller never decides whether the doctor is allowed to view the patient.

That is business logic.

---

# What Should Controllers Never Do?

Controllers should never

- Write SQL
- Query Prisma directly
- Calculate business rules
- Decide permissions
- Validate request structure
- Hash passwords
- Generate JWT
- Send emails
- Access Redis directly

Controllers coordinate.

They do not implement.

---

# Example Responsibility

Imagine

```
GET /patients/:id
```

The controller knows

```
patientId

doctorId
```

The controller does **not** know

- how patients are stored
- how permissions work
- how consultations are loaded

It forwards everything to the service.

---

# Engineering Note

A good controller should be readable in less than a minute.

If understanding one controller requires scrolling through hundreds of lines of code, it is usually performing responsibilities that belong somewhere else.

---

# Controller Is an HTTP Layer

Controllers understand

- HTTP Requests
- HTTP Responses
- Status Codes

Services should not.

Example

Controller

```
Return

200 OK
```

Service

```
Return Patient
```

The service returns business data.

The controller converts it into an HTTP response.

---

# Controller Inputs

Controllers usually receive

```
Request Body

↓

Route Parameters

↓

Query Parameters

↓

Authenticated User

↓

Headers
```

These are HTTP concepts.

Business services should not depend on Express request objects.

---

# Controller Outputs

Controllers usually return

```
200 OK

201 Created

204 No Content

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found
```

Choosing the correct response code is part of the controller's responsibility.

---

# Controller Does Not Know Business Rules

Suppose

Doctor opens patient history.

The controller should **not** check

```
Same Clinic?
```

It should simply call

```
PatientService
```

The service decides.

Keeping controllers unaware of business rules makes them small and reusable.

---

# Controller Does Not Know Database

Suppose the application changes

```
Prisma

↓

MongoDB
```

Controllers should remain unchanged.

Only repositories should change.

If controllers directly query the database,

they become tightly coupled to persistence.

---

# Typical Request Flow

Example

```
GET /patients/:id
```

Flow

```
Controller

↓

Read patientId

↓

Read doctorId

↓

PatientService.getPatient()

↓

Receive Result

↓

Return Response
```

Every endpoint should follow a similar pattern.

---

# Thin Controller

A good controller is usually

- Small
- Easy to read
- Easy to test

Most of its work is delegation.

Large controllers usually indicate misplaced responsibilities.

---

# Where Should Controllers Live?

Controllers belong to their business module.

Example

```
modules/

    patient/

        patient.controller.ts
```

The patient controller should only manage patient-related endpoints.

Avoid creating generic controllers shared across unrelated modules.

---

# One Controller Per Module

Typical structure

```
doctor/

    doctor.controller.ts

patient/

    patient.controller.ts

appointment/

    appointment.controller.ts

consultation/

    consultation.controller.ts
```

Each controller owns one business module.

---

# Common Mistakes

### Business Logic Inside Controllers

Example

```
if doctor belongs to clinic

↓

Load patient

↓

Compare permissions

↓

Calculate response
```

This belongs inside the service.

---

### Database Queries Inside Controllers

Example

```
Controller

↓

Prisma Query
```

Now every controller knows database details.

Repositories lose their purpose.

---

### Calling Multiple Repositories

Controllers should not coordinate repositories.

Services coordinate business operations.

Controllers coordinate HTTP.

---

### Large Controllers

Controllers with

- loops
- calculations
- transactions
- complex conditions

usually contain misplaced business logic.

---

# Trade-offs

### Very Thin Controllers

Advantages

- Easy to maintain
- Easy to test
- Clear responsibility

Disadvantages

- Requires good service design

---

### Fat Controllers

Advantages

- Quick to build

Disadvantages

- Difficult to maintain
- Difficult to test
- Business logic scattered
- Hard to reuse

As applications grow,

thin controllers become significantly easier to maintain.

---

# Controller Checklist

Before completing a controller,

ask

- Does it only understand HTTP?
- Does it call exactly one business service?
- Does it avoid business decisions?
- Does it avoid database queries?
- Does it avoid validation?
- Does it return appropriate HTTP responses?

If every answer is "Yes",

the controller is correctly designed.

---

# Summary

Controllers act as the bridge between HTTP requests and the business layer.

Their responsibility is to translate incoming requests into business operations and convert business results into HTTP responses.

Controllers should remain small, predictable, and independent of business rules and database implementation.

---

# Next Chapter

The next chapter covers **Services**.

Services are the core of the backend.

They implement business rules, coordinate application behavior, and contain the logic that gives meaning to the application's features.
