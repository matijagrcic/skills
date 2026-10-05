---
name: worker-best-practices
description: Review and author Cloudflare Workers code against production best practices for configuration, request handling, architecture, observability, security, and testing. Use when writing or reviewing Workers, wrangler configuration, bindings, service-to-service calls, background work, streaming, secrets, logs, or Workers runtime tests.
---

# Worker Best Practices

Use this skill when building, reviewing, or debugging Cloudflare Workers that need to be fast, reliable, observable, and secure.

Before giving Cloudflare advice, changing Worker or Wrangler code, or running a Cloudflare command, read `cloudflare.md` from the repository root in full. It contains the repository's general Cloudflare reference, including Local Explorer, Workflows, Agents SDK, secrets, databases, auth, runtime types, and Worker conventions.

If the Worker task changes an API contract, Hono route shape, OpenAPI document, Orval output, Zod schema, MSW mock, or end-to-end type flow, also read `api-contracts.md`. If it changes a TanStack consumer, also read `tanstack.md`.

Primary reference:

- Cloudflare docs: [Workers Best Practices](https://developers.cloudflare.com/workers/best-practices/workers-best-practices/index.md)
- Workers types v5 changelog: [Simpler runtime types with @cloudflare/workers-types v5](https://developers.cloudflare.com/changelog/post/2026-07-03-workers-types-v5/)

## Review Workflow

1. Identify the Worker entry points, `wrangler.jsonc` or `wrangler.toml`, bindings, and tests.
2. Check configuration first: compatibility date, compatibility flags, generated binding types, secrets, and environment-specific bindings.
3. Check request behavior: streaming, body size limits, `waitUntil`, response correctness, and error handling.
4. Check architecture: bindings instead of REST APIs, Queues or Workflows for async work, service bindings for Worker-to-Worker calls, Hyperdrive for external databases, Durable Objects for WebSockets, and Static Assets for new static projects.
5. Check runtime safety: no request-scoped mutable globals and no floating promises.
6. Check observability, security, and tests before considering the Worker production-ready.

## Configuration

- Keep `compatibility_date` current so new projects get current runtime behavior and existing projects can opt into fixes deliberately.
- Enable `nodejs_compat` when code or dependencies use Node.js built-ins such as `node:crypto`, `node:buffer`, or `node:stream`.
- Generate binding types with `npx wrangler types`; do not hand-write `Env` interfaces that can drift from configuration.
- When debugging type errors or runtime API mismatches, prefer `npx wrangler types` because generated types are locked to the Worker's `compatibility_date` and bindings.
- `@cloudflare/workers-types` v5 exposes only the latest stable entrypoint plus `/experimental`; dated entrypoints such as `@cloudflare/workers-types/2023-03-01` were removed.
- Store secrets with `wrangler secret put` or `wrangler secret bulk`, not in source code or Wrangler config. Use `.env` for local development and keep it ignored.
- Configure environments deliberately. Avoid assuming production bindings, variables, and routes are safe for preview or staging.
- Set custom domains and routes explicitly, and verify the deployed Worker is attached to the intended zone, route, or domain.

## Request Handling

- Stream large request and response bodies instead of buffering with `text()`, `json()`, or `arrayBuffer()` unless a bounded body size is enforced.
- Use `TransformStream` or direct body passthrough for proxying and concatenating large upstream responses.
- Use `ctx.waitUntil()` only for work that does not affect the response, such as analytics, cache writes, logging, and webhook notifications.
- Do not destructure `ctx.waitUntil`; call it as `ctx.waitUntil(...)` to preserve its binding.
- Await work that determines the response. Use `waitUntil` only when the result is not needed by the client response.

## Architecture

- Use Cloudflare service bindings for R2, KV, D1, Queues, Workflows, and other services instead of calling Cloudflare REST APIs from inside a Worker.
- Use Queues for decoupled, retryable, single-step background jobs and buffering.
- Use Workflows for durable, multi-step processes where later steps depend on earlier results or may pause and resume.
- Use service bindings for Worker-to-Worker communication instead of public HTTP calls. Prefer typed RPC when available.
- Use Hyperdrive for external database connections that need pooling and acceleration.
- Use Durable Objects for WebSockets and stateful coordination.
- Use Workers Static Assets for new projects that combine static files with Worker logic.

## Code Patterns

- Never store request-scoped data in module-level mutable variables. Isolates are reused across requests, so globals can leak user data or stale state.
- Pass request state through function arguments or store durable/shared state in the appropriate binding.
- Do not leave floating promises. Every promise should be awaited, returned, or passed to `ctx.waitUntil()`.
- Enable a no-floating-promises lint rule when TypeScript linting is available.

## Observability, Security, And Testing

- Enable Workers Logs and Traces for deployed Workers, and log structured context that helps diagnose failures without leaking secrets.
- Use `crypto.randomUUID()` or `crypto.getRandomValues()` for security-sensitive random values. Do not use `Math.random()` for tokens.
- Compare secrets with constant-time comparison, such as `crypto.subtle.timingSafeEqual()` over fixed-size hashes.
- Do not use `passThroughOnException()` as normal error handling. Prefer explicit `try...catch` and structured error responses.
- Test Workers with `@cloudflare/vitest-pool-workers` so tests run inside the Workers runtime with real binding behavior.
- Remember that the Vitest pool can inject `nodejs_compat`; verify Wrangler config includes the flag if production code depends on Node.js APIs.
