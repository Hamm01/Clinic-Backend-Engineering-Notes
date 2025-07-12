# Authentication

Authentication is the first security layer of the backend. Before any protected API can execute business logic, the application must verify **who is making the request**.

This section builds a complete authentication system for the Clinic Backend using **Express**, **TypeScript**, **Prisma**, **PostgreSQL**, **JWT**, and **bcrypt**.

The goal is not only to implement authentication, but to understand the reasoning behind every component, how they interact, and why production applications are designed this way.

---

# Learning Objectives

After completing this section, you should be able to:

- Understand the purpose of authentication.
- Differentiate Authentication from Authorization.
- Design a proper User Account model.
- Store passwords securely using hashing.
- Implement Register and Login APIs.
- Generate and verify JWTs.
- Build Authentication Middleware.
- Protect Express routes.
- Access the currently authenticated user.
- Understand Refresh Tokens.
- Implement Logout.
- Visualize the complete authentication request lifecycle.

---

# Prerequisites

Before starting this section, you should have completed:

- Project Initialization
- Express Application
- Environment Configuration
- Project Structure
- Logging
- Error Handling
- Database Design
- Prisma Migrations
- Seed Data

Authentication depends on the database because user accounts are stored in PostgreSQL.

---

# Folder Structure

```
08 - Authentication
│
├── README.md
│
├── 08.1 - Authentication Overview.md
├── 08.2 - User Account Model.md
├── 08.3 - Password Hashing.md
├── 08.4 - Register API.md
├── 08.5 - Login API.md
├── 08.6 - JSON Web Tokens (JWT).md
├── 08.7 - Authentication Middleware.md
├── 08.8 - Protecting Routes.md
├── 08.9 - Current User (Me Endpoint).md
├── 08.10 - Refresh Tokens.md
├── 08.11 - Logout.md
└── 08.12 - Complete Authentication Flow.md
```

---

# Learning Path

Authentication is implemented in the same order that a production backend is developed.

```
Authentication Overview

↓

User Account Model

↓

Password Hashing

↓

Register API

↓

Login API

↓

JWT

↓

Authentication Middleware

↓

Protected Routes

↓

Current User Endpoint

↓

Refresh Tokens

↓

Logout

↓

Complete Authentication Flow
```

Each chapter builds on the previous one.

Avoid skipping chapters.

---

# Authentication Architecture

```
Client

↓

POST /login

↓

Route

↓

Controller

↓

Service

↓

Repository

↓

Database

↓

Password Verification

↓

JWT Generated

↓

Client Stores Token

↓

Future Requests

↓

Authentication Middleware

↓

Controller

↓

Business Logic

↓

HTTP Response
```

This flow represents every authenticated request in the application.

---

# Technologies Used

| Technology | Purpose                  |
| ---------- | ------------------------ |
| Express    | HTTP Server              |
| TypeScript | Type Safety              |
| PostgreSQL | User Storage             |
| Prisma     | Database ORM             |
| bcrypt     | Password Hashing         |
| JWT        | Stateless Authentication |
| dotenv     | Environment Variables    |

---

# What We Will Build

By the end of this section, the backend will support:

- User Registration
- User Login
- Password Hashing
- JWT Generation
- JWT Verification
- Authentication Middleware
- Protected APIs
- Current User Endpoint (`/me`)
- Refresh Tokens
- Logout

---

# Design Principles

Throughout this section, we follow these principles:

- Authentication before Authorization.
- Passwords are never stored in plain text.
- Business logic remains inside Services.
- Controllers remain thin.
- Database access is isolated inside Repositories.
- Middleware is responsible for request authentication.
- Shared authentication utilities remain reusable.
- Every authenticated request has an identified user.

---

# Business Scenario

Throughout this module, we continue using the Clinic application.

The authentication system will support internal staff such as:

- Administrator
- Doctor
- Receptionist

A user logs in using an email and password.

After successful authentication, the backend issues a JWT.

Every protected request includes that token so the backend can identify the user before executing business logic.

---

# What We Will Not Cover

This module focuses on backend authentication.

The following topics are intentionally outside the scope of this section:

- OAuth (Google, GitHub)
- Multi-Factor Authentication (MFA)
- Social Login
- SSO
- API Keys
- Session-based Authentication
- Identity Providers (Auth0, Clerk, Cognito)

These can be added later as advanced authentication strategies.

---

# Completion Checklist

By the end of this section, you should be able to:

- Explain the authentication lifecycle.
- Design a secure User Account model.
- Hash and verify passwords.
- Build Register and Login APIs.
- Generate secure JWTs.
- Authenticate incoming requests.
- Protect Express routes.
- Access the authenticated user throughout the request lifecycle.
- Understand token refresh strategies.
- Implement a complete authentication system suitable for production.

---

# Next Chapter

Begin with:

```
08.1 - Authentication Overview
```

This chapter explains why authentication exists, how it fits into the request lifecycle, and how the authentication system integrates with the overall backend architecture before writing any implementation code.
