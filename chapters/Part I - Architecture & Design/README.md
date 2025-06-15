# Part I — Architecture & Design

> "Good code starts with good decisions."

---

## Purpose

This part focuses on understanding **how to think like a backend engineer before writing code**.

Most tutorials begin by creating an Express application and adding routes.

This repository intentionally follows a different approach.

Before implementation, we first understand

- the business
- the request lifecycle
- application architecture
- responsibilities of each layer
- database design
- scalability considerations
- production trade-offs

By the end of this part, every major architectural decision has already been made.

Implementation becomes a process of translating those decisions into code.

---

# What You Will Learn

This section answers questions such as

- How should a backend be structured?
- Why do Routes, Controllers, Services, and Repositories exist?
- Where should business logic live?
- What makes a backend stateless?
- How should requests travel through the application?
- How should the database model the business?
- Why are certain relationships chosen?
- How do senior engineers review backend architecture?

---

# Learning Goal

The goal is **not** to memorize folder structures.

The goal is to understand

> why each layer exists,
> what responsibility it owns,
> and where it fits into the overall architecture.

Once these concepts are clear, implementation becomes much easier.

---

# How to Read This Part

Read every chapter in order.

Each chapter builds on the previous one.

Do not skip directly to the database chapters.

The request lifecycle explains why the application is structured the way it is.

The folder structure explains where code belongs.

The database chapters explain why the business is modeled in a particular way.

Together, they form the foundation for implementation.

---

# Expected Outcome

After completing this part, you should be able to

- design a backend architecture
- review a folder structure
- separate business logic from infrastructure
- identify responsibilities of each layer
- model a business into database entities
- explain architectural decisions during interviews and design discussions

No production code is written in this section.

The focus is understanding.

---

# Transition to Part II

Once the architecture is complete, we move into implementation.

Every file created in Part II directly maps back to a decision made in this part.

Nothing is implemented without first understanding why it exists.
