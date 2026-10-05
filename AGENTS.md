## Engineering principles

- Prefer simple, readable, flat code with minimal indirection.
- Search for existing implementations and installed libraries before creating new helpers or abstractions.
- Abstract when it prevents meaningful drift and makes the result simpler to maintain. Avoid speculative or one-use abstraction layers.
- Keep product data normalized and relationships explicit. Do not encode relational data in JSON or text merely to avoid joins.
- For new application-backed backend functionality, default to: TanStack server function → service → repository.
- Keep schema changes, queries, and mutations compatible with both SQLite and Postgres.
- Use idiomatic TypeScript. Use Zod to validate untrusted data and narrow runtime values at trust boundaries.
- Prefer established project helpers and libraries over hand-rolled implementations.
- Prefer idiomatic TanStack Query, Router, and Form patterns for server state, routing, and submitted forms.

## Technology vault routing

The root technology files contain maintained, general-purpose engineering guidance. Before advice, commands, reviews, or edits in an applicable domain, read every matching file in full on that task rather than relying on memory from an earlier task.

- For Cloudflare Workers, Wrangler, Hono on Workers, bindings, D1, KV, R2, Durable Objects, Workflows, Hyperdrive, Worker secrets, runtime types, local inspection, or deployment configuration, use `.agents/skills/worker-best-practices/SKILL.md` and read `cloudflare.md`.
- For TanStack Query, Router, Start, Form, Virtual, AI, Intent, server functions, routing, caching, mutations, or form validation, use `.agents/skills/tanstack-project-patterns/SKILL.md` and read `tanstack.md`.
- For API contracts, OpenAPI, Orval, generated Query clients, generated Zod schemas, MSW, Faker, request or response models, API mutators, branded types across boundaries, or end-to-end type flow, use `.agents/skills/api-contract-pipeline/SKILL.md` and read `api-contracts.md`.
- For work spanning domains, such as a Hono endpoint generated through Orval into a TanStack Start form, read all applicable files and use all applicable skills.

These files guide implementation but do not authorize external writes, deployments, remote migrations, secret changes, or modifications to live services.

## Log papercuts

When small, non-blocking repository friction occurs—a retried tool call, confusing setup step, flaky command, stale cache, misleading error, or non-obvious gotcha—use the `papercuts` skill and append it to `.agents/PAPERCUTS.md` in the moment. Continue the current task. Real bugs and tracked work are not papercuts, and sensitive data must never be logged.

Do not mine an entire session for papercuts or start a broad cleanup unless the user explicitly asks.
