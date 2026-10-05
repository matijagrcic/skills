---
name: tanstack-project-patterns
description: Apply the repository's general TanStack Query, Router, Start, Form, Virtual, AI, Intent, and server-function guidance. Use when work touches TanStack packages, routing, caching, mutations, forms, SSR or SPA modes, virtualized lists, or TanStack AI.
---

# TanStack Project Patterns

Before giving implementation advice, changing code, or running a TanStack-related command, read `tanstack.md` from the repository root in full. It is maintained general guidance and includes Intent, linting, Virtual Chat, Query/Router integration, Start rendering modes, forms, and Better Auth.

If the task also touches OpenAPI, Orval, generated Query clients, generated Zod schemas, MSW, Faker, API DTOs, branded types across a network boundary, or end-to-end type flow, also read `api-contracts.md` in full before acting.

## Apply only the relevant mode

- Use TanStack Query for globally addressable server state and cache synchronization.
- Use Router loaders to start route-owned work and prime Query when Query owns the long-term cache; avoid competing caches.
- Use TanStack Form for submitted form values, field state, validation, arrays, and nested inputs.
- Use Start rendering controls only in TanStack Start projects and choose SSR, data-only SSR, route-level client rendering, or SPA mode from the route's actual constraints.
- Use TanStack Virtual for large or dynamically measured lists where virtualization and scroll preservation are needed.
- Use TanStack AI package skills through Intent when those packages are installed.

Do not introduce a TanStack product merely because the reference covers it. First inspect the framework, installed packages, existing routing and state ownership, package manager, and project conventions. Use the installed package types and current official documentation for exact APIs.

Keep reusable query option factories near the typed server or client call, share one QueryClient between Router context and the provider, derive types rather than copying response shapes, and subscribe to the narrowest route or query state needed.

For forms, define controlled defaults, connect fields directly to UI controls, render accessible validation state, and submit validated values through the application's established server or mutation boundary. Do not mix React Hook Form-specific wrappers into a TanStack Form implementation.

Verify changes with the project's own formatter or linter, TypeScript, focused behavior tests, route loading and cache behavior, form validation and submission, and SSR/hydration or virtualization checks when those modes are affected.
