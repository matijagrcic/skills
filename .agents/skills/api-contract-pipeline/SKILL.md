---
name: api-contract-pipeline
description: Maintain end-to-end API type flow through server contracts, OpenAPI, Orval, TanStack Query, Zod, MSW, and Faker. Use when changing API schemas, generated clients, request or response types, validation schemas, mocks, handlers, mutators, or Orval configuration.
---

# API Contract Pipeline

Before giving implementation advice, changing code, editing a contract, or running generation, read `api-contracts.md` from the repository root in full. It contains the general end-to-end type rules, Orval configuration pattern, and MSW composition guidance.

Also read:

- `tanstack.md` when generated output feeds TanStack Query, Router loaders, Start server functions, TanStack Form, or another TanStack consumer.
- `cloudflare.md` when a Worker, Hono route, binding, D1 or Hyperdrive database, Cloudflare service, or deployment behavior is part of the API change.

## Preserve the contract chain

Treat the database schema and typed server behavior plus the checked-in OpenAPI specification as the source side of the contract. Orval owns its generated clients, models, Query integrations, Zod schemas, Faker factories, and baseline MSW mocks. Consumer types should be inferred or derived from those outputs rather than copied into parallel interfaces.

Before editing, inspect the generator configuration, OpenAPI inputs, mutators, every output directory, and current generated imports. Resolve exact generation targets before running a configuration with `clean: true`; never place handwritten files where the generator may delete them. Preserve the project's existing package manager and HTTP client unless the task explicitly changes them.

When changing an endpoint:

1. Update server behavior and the public contract together.
2. Preserve branded and refined types when the stack supports them.
3. Keep tags aligned with API domains so generated clients and mock handlers remain navigable.
4. Regenerate through the repository's established command.
5. Inspect generated diffs and fix the contract or generator configuration instead of patching generated files.
6. Compose generated MSW groups through one stable entry point, adding narrow scenario overrides only when tests require them.
7. Reuse generated Zod schemas where their input semantics match rather than creating a second validation model.

Keep transport behavior in the established mutator, including base URL selection, authentication, credentials, cancellation, serialization, and normalized errors. Do not spread those concerns across generated call sites.

Verify the server and consumers independently, exercise representative success and failure mocks, and confirm types reach the consuming route, form, or component without manual DTO duplication.
