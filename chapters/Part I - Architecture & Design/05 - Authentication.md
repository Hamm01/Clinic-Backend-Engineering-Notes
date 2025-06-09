# 05 - Authentication

> Authentication answers one question:
>
> **"Who is making this request?"**

---

# Objective

Before a backend allows access to protected resources, it must identify the user making the request.

Authentication is the process of verifying identity.

It does **not** decide what a user is allowed to do.

It only answers:

> "Who are you?"

Authorization, which comes later, answers:

> "What are you allowed to do?"

Understanding this difference is one of the most important concepts in backend engineering.

---

# Authentication in Our Clinic Application

Our application has multiple types of users.

- Admin
- Doctor
- Receptionist

Patients do not log in to this system.

Only internal staff can access the backend portal.

Example

```
Doctor

↓

Login

↓

JWT Generated

↓

Doctor accesses patients

↓

Doctor creates consultation

↓

Doctor prescribes medicines
```

Every protected request must first identify the doctor.

---

# Why Authentication Exists

Without authentication,

every request would look identical.

Example

```
GET /patients/123
```

The backend would not know

- Who sent the request?
- Which clinic do they belong to?
- What role do they have?

Authentication provides identity.

---

# Authentication vs Authorization

These concepts should never be mixed.

Authentication

```
Who are you?
```

Authorization

```
What are you allowed to do?
```

Example

Doctor logs in successfully.

Authentication

✓ Success

Doctor tries to access another clinic's patient.

Authorization

✗ Denied

The identity is valid.

The action is not.

---

# Authentication Flow

A typical request follows this flow.

```
Doctor

↓

Login

↓

Credentials Verified

↓

JWT Generated

↓

Client Stores Token

↓

Future Requests

↓

Authentication Middleware

↓

Request Continues
```

Authentication usually happens only once.

Every future request verifies the token.

---

# Login Process

Doctor enters

- Email
- Password

Backend receives

```
POST /auth/login
```

The backend then performs the following steps.

1. Find doctor by email.
2. Verify account exists.
3. Compare password.
4. Generate JWT.
5. Return access token.

If any step fails,

login fails.

---

# What Should Be Verified?

During login,

the backend should verify

- User exists.
- Password is correct.
- Account is active.
- Account is not suspended.
- Login method is allowed.

Business requirements decide which checks are necessary.

---

# What Happens After Login?

Once login succeeds,

the backend generates an identity token.

Example information inside the token

```
User ID

Doctor ID

Clinic ID

Role
```

Notice something.

The token contains **identity**.

It should not contain business data like

- patient records
- appointments
- prescriptions

---

# Every Protected Request

Suppose

```
GET /patients/123
```

The request contains

```
Authorization

Bearer <token>
```

The backend verifies

- token signature
- expiration
- issuer
- integrity

If valid,

the request continues.

Otherwise,

authentication fails.

---

# Authentication Middleware

Authentication should happen before business logic.

Example flow

```
Incoming Request

↓

Authentication Middleware

↓

Controller

↓

Service

↓

Repository
```

If authentication fails,

the request never reaches the controller.

---

# What Does Authentication Produce?

After successful verification,

the application knows

```
Current User

↓

User ID

↓

Role

↓

Clinic
```

Every remaining layer can now use this information.

Example

```
req.user
```

The controller should never decode JWT manually.

Authentication middleware performs this work once.

---

# Authentication Responsibilities

Authentication is responsible for

- Identifying users
- Verifying credentials
- Verifying tokens
- Extracting identity
- Rejecting invalid requests

Authentication is **not** responsible for

- Checking permissions
- Business rules
- Data validation
- Database queries unrelated to identity

Keep responsibilities focused.

---

# Where Does Authentication Live?

Authentication is a cross-cutting concern.

Typical structure

```
modules/

    auth/
```

Contains

- Login
- Refresh Token
- Logout

Shared authentication middleware

```
shared/

    middleware/

        authentication.ts
```

Authentication should be reusable by every protected module.

---

# Common Mistakes

### Mixing Authentication and Authorization

Wrong

```
Verify JWT

↓

Check permissions

↓

Business validation
```

One middleware should not perform every responsibility.

Authentication identifies.

Authorization decides permissions.

---

### Decoding JWT Inside Controllers

Wrong

```
Controller

↓

Decode JWT

↓

Extract User
```

Every controller duplicates the same work.

Authentication middleware should perform this once.

---

### Trusting Client Data

Never trust

```
doctorId

clinicId

role
```

sent inside the request body.

Always derive identity from the authenticated token.

---

### Storing Business Data Inside JWT

JWT should contain identity,

not application state.

Avoid storing

- appointments
- consultation history
- permissions list
- patient information

Keep tokens small.

---

# Trade-offs

### Session-Based Authentication

Advantages

- Easy logout
- Easy token invalidation

Disadvantages

- Shared session storage required
- Harder to scale horizontally

---

### JWT Authentication

Advantages

- Stateless
- Scales easily
- Simple API authentication

Disadvantages

- Revocation requires additional design
- Token expiration must be managed carefully

For our clinic application,

JWT is a suitable choice because the backend remains stateless.

---

# Authentication Checklist

Before implementing authentication,

verify the following.

- Who can log in?
- Which credential is used?
- Where are passwords stored?
- How are passwords verified?
- What information belongs inside the token?
- How is the token verified?
- Which routes require authentication?

If these questions are answered,

authentication design is complete.

---

# Summary

Authentication establishes identity.

It verifies who is making the request and provides that identity to the rest of the backend.

It does not make business decisions or permission checks.

Keeping authentication focused makes the application easier to understand, test, and maintain.

---

# Next Chapter

The next chapter covers **Authorization**.

Authentication identifies the user.

Authorization determines whether that authenticated user is allowed to perform a specific business action.
