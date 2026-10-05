# Cloudflare Local Explorer

[Local Explorer](https://developers.cloudflare.com/workers/development-testing/local-explorer/) is a browser-based interface for inspecting and editing local Cloudflare Worker binding data during development.

It is available at `/cdn-cgi/explorer` on a local Worker development server and works with both Wrangler and the Cloudflare Vite plugin.

For Worker or Hono API contracts, OpenAPI, Orval-generated clients, generated Zod schemas, MSW mocks, or end-to-end type flow, also read [`api-contracts.md`](./api-contracts.md). Read [`tanstack.md`](./tanstack.md) as well when a TanStack consumer is part of the change.

## Requirements

- Wrangler `4.82.1` or later, or Cloudflare Vite plugin `1.32.0` or later.
- A local development session with bindings configured in the Worker configuration.

## Open Local Explorer

Start the Worker locally:

```sh
npx wrangler dev
```

Then either:

- Press `e` in the Wrangler terminal.
- Open `/cdn-cgi/explorer` on the local Worker route, for example `http://localhost:8787/cdn-cgi/explorer`.

Local Explorer detects bindings from the Wrangler configuration automatically.

## Supported Bindings

Local Explorer can view and edit local data for:

- KV: browse, create, update, and delete key-value pairs.
- R2: list, inspect, upload, and delete objects.
- D1: browse tables and rows, edit data through SQL, and run ad-hoc SQL queries.
- Durable Objects with SQLite storage: browse tables and rows, edit data through SQL, and run SQL queries.
- Workflows: list instances, inspect status and step history, trigger new runs, and retry failed runs.

## Local Workflows

[Workflows local development](https://developers.cloudflare.com/workflows/build/local-development/) is supported through Wrangler's emulated runtime. This lets you develop and test Workflow definitions locally before deploying them.

Local Workflow development requires:

- Wrangler `3.89.0` or later.
- Node.js `18.0.0` or later.
- A running `wrangler dev` session.

Check the Wrangler version:

```sh
npx wrangler --version
```

Start local development:

```sh
npx wrangler dev
```

While `wrangler dev` is running, Wrangler `4.79.0` or later can manage local Workflow instances with the `--local` flag:

```sh
npx wrangler workflows list --local
npx wrangler workflows trigger my-workflow --local
npx wrangler workflows instances describe my-workflow <INSTANCE_ID> --local
```

Use `--port` if the local dev server is not running on the default `8787` port.

Local Explorer can also manage Workflows in the browser. It can view Workflow definitions and instances, inspect step history and status, trigger new runs, send events to running instances, pause, resume, terminate, restart, delete specific instances, or clear all instances.

Workflows are not supported as remote bindings or with `npx wrangler dev --remote`.

## Dynamic Workflows

[Dynamic Workflows](https://developers.cloudflare.com/dynamic-workers/usage/dynamic-workflows/) use `@cloudflare/dynamic-workflows` to run Workflow code inside [Dynamic Workers](https://developers.cloudflare.com/dynamic-workers/), giving runtime-loaded tenant or user code durable Workflow execution.

Use this pattern when the Workflow code itself is dynamic, such as per-tenant SaaS automations, AI agent plans generated at runtime, or multi-tenant job systems where each customer supplies its own processing logic. The Workflows engine still persists step output, retries, sleeps, and event waits, and reloads the correct Dynamic Worker when an instance resumes.

Install the library:

```sh
npm i @cloudflare/dynamic-workflows
```

Configure the Worker Loader with both a Worker Loader binding and a Workflow binding whose `class_name` matches the Dynamic Workflow entrypoint:

```jsonc
{
  "name": "my-worker-loader",
  "main": "src/index.ts",
  "compatibility_date": "2026-06-11",
  "worker_loaders": [
    {
      "binding": "LOADER"
    }
  ],
  "workflows": [
    {
      "name": "dynamic-workflow",
      "binding": "WORKFLOWS",
      "class_name": "DynamicWorkflow"
    }
  ]
}
```

Create the Worker Loader entrypoint by wrapping the Workflow binding with routing metadata and exporting the dynamic Workflow class:

```ts
import {
  createDynamicWorkflowEntrypoint,
  DynamicWorkflowBinding,
  wrapWorkflowBinding,
  type WorkflowRunner,
} from "@cloudflare/dynamic-workflows";

export { DynamicWorkflowBinding };

interface Env {
  WORKFLOWS: Workflow;
  LOADER: WorkerLoader;
}

function loadTenant(env: Env, tenantId: string) {
  return env.LOADER.get(tenantId, async () => ({
    compatibilityDate: "2026-01-01",
    mainModule: "index.js",
    modules: { "index.js": await fetchTenantCode(tenantId) },
    env: { WORKFLOWS: wrapWorkflowBinding({ tenantId }) },
  }));
}

export const DynamicWorkflow = createDynamicWorkflowEntrypoint<Env>(
  async ({ env, metadata }) => {
    const stub = loadTenant(env, metadata.tenantId as string);
    return stub.getEntrypoint("TenantWorkflow") as unknown as WorkflowRunner;
  },
);

export default {
  fetch(request: Request, env: Env) {
    const tenantId = request.headers.get("x-tenant-id")!;
    return loadTenant(env, tenantId).getEntrypoint().fetch(request);
  },
};
```

The Dynamic Worker defines a normal Workflow and starts instances through the wrapped binding:

```ts
import { WorkflowEntrypoint } from "cloudflare:workers";

export class TenantWorkflow extends WorkflowEntrypoint {
  async run(event, step) {
    return step.do("greet", async () => `Hello, ${event.payload.name}!`);
  }
}

export default {
  async fetch(request, env) {
    const instance = await env.WORKFLOWS.create({
      params: await request.json(),
    });

    return Response.json({ id: await instance.id });
  },
};
```

Important notes:

- Re-export `DynamicWorkflowBinding` from the Worker Loader so the runtime can build wrapped Workflow stubs.
- Use `wrapWorkflowBinding({ tenantId })` for routing metadata only. Do not put secrets in metadata because Workflows persists it in the event payload and Dynamic Worker code can read it through instance status.
- `env.WORKFLOWS.create()`, instance IDs, `.status()`, `.pause()`, retries, hibernation, `step.do()`, `step.sleep()`, and `step.waitForEvent()` keep normal Workflows behavior.
- Trigger the Worker Loader with a tenant identifier, for example `x-tenant-id: tenant-42`, and let the Dynamic Worker create the Workflow instance.

## Scheduled Workflows

[Scheduled Workflow instances](https://developers.cloudflare.com/changelog/post/2026-06-02-cron-workflows/) can be created directly from a Workflow binding by adding cron expressions to the binding's `schedules` field in `wrangler.jsonc`.

Each cron run creates a new Workflow instance automatically, so a separate Worker with a `scheduled` handler is not needed just to trigger the Workflow on an interval.

Example `wrangler.jsonc`:

```jsonc
{
  "workflows": [
    {
      "name": "my-scheduled-workflow",
      "binding": "MY_WORKFLOW",
      "class_name": "MyScheduledWorkflow",
      "schedules": ["0 * * * *", "*/15 * * * *", "0 9 * * MON-FRI"]
    }
  ]
}
```

Scheduled Workflows keep the normal Workflow benefits, including durable multi-step execution, built-in retries, and configurable timeouts. They are a good fit for recurring work such as database backups, invoice generation, report aggregation, and cleanup tasks.

## Agents SDK v0.17.0

[Agents SDK v0.17.0](https://developers.cloudflare.com/changelog/post/2026-06-26-agents-sdk-v0170/) added durable background sub-agent runs, progress reporting, a unified `runTurn` entry point, and recovery fixes for production chat agents.

Use these notes when debugging Cloudflare Agents built with `agents`, `@cloudflare/think`, or `@cloudflare/ai-chat`:

- Use `runAgentTool(..., { detached: { ... } })` for long-running sub-agent work that should survive the calling turn, deploys, and evictions. Detached runs can report progress, emit durable milestones, notify the chat on completion, and use `cancelAgentTool(runId)` plus `maxBudgetMs` to avoid abandoned concurrency slots.
- Prefer `this.runTurn({ mode, messages })` in `@cloudflare/think` when a codebase mixes turn entry points. Modes are `wait`, `submit`, and `stream`; nested blocking admissions now throw clearer errors instead of silently deadlocking.
- For hung or stuck chat streams, check whether `AIChatAgent` has `chatStreamStallTimeoutMs` and `chatRecovery` configured. The watchdog can recover stalled model or transport streams, or surface a terminal stream error so client spinners clear.
- If recovered turns fail with missing tool results, check for interrupted server-tool calls. `AIChatAgent` now repairs dead tool calls before inference and exposes `repairInterruptedToolPart(part)` for custom repaired shapes.
- On reconnect issues, inspect `useAgentChat` recovery state and terminal WebSocket close handling. v0.17.0 replays live recovering status to reconnecting clients and exposes terminal connection failures through `connectionError` and `onConnectionError`.
- MCP callbacks that do not inherit the agent async context can now send `McpAgent` server-to-client requests, including callbacks reached through Worker Loader RPC.

Upgrade related packages together:

```sh
npm i agents@latest @cloudflare/think@latest @cloudflare/ai-chat@latest @cloudflare/codemode@latest @cloudflare/voice@latest
```

## Bulk Secrets

The [Workers bulk secrets API](https://developers.cloudflare.com/changelog/post/2026-06-03-bulk-secrets-api/) lets you create, update, or delete multiple Worker secrets in a single request. Secrets not included in the request are left unchanged.

Use the [bulk secrets endpoint](https://developers.cloudflare.com/api/resources/workers/subresources/scripts/subresources/secrets/methods/bulk_update/) from the Cloudflare API, or run the same operation from the CLI with [wrangler secret bulk](https://developers.cloudflare.com/workers/wrangler/commands/workers/#secret-bulk):

```sh
npx wrangler secret bulk < secrets.json
```

Example `secrets.json`:

```json
{
  "secrets": {
    "API_KEY": { "type": "secret_text", "name": "API_KEY", "text": "my-api-key" },
    "DB_PASSWORD": { "type": "secret_text", "name": "DB_PASSWORD", "text": "my-db-password" },
    "OLD_SECRET": null
  }
}
```

Include a secret with a value to create or update it. Set a secret to `null` to delete it. Deletion is not supported with `.env` files. Each request supports up to 100 total operations (creates, updates, and deletes combined).

## Workflow Rollbacks

[Workflow rollback support](https://developers.cloudflare.com/changelog/post/2026-06-05-saga-rollbacks/) allows saga-style compensating logic to be attached to each `step.do()` call. If a Workflow instance fails, rollback handlers run in reverse `step-start` order.

This is useful for multi-step operations that touch external systems, such as inventory reservations, payment authorization, ticket creation, or infrastructure provisioning. Keeping each rollback handler next to the step it undoes can be clearer than collecting cleanup logic in a top-level `catch`.

Example rollback handler:

```ts
await step.do(
  "provision resource",
  async () => {
    const resource = await provisionResource();
    return { resourceId: resource.id };
  },
  {
    rollback: async ({ output }) => {
      const { resourceId } = output as { resourceId: string };
      await deleteResource(resourceId);
    },
    rollbackConfig: {
      retries: { limit: 3, delay: "15 seconds", backoff: "linear" },
      timeout: "2 minutes"
    }
  }
);
```

Rollback handlers can define their own retry and timeout configuration. Workflow instance status responses expose rollback outcomes, and Workflow analytics emits rollback lifecycle events so forward execution failures can be distinguished from rollback failures.

## SQL Studio

For local D1 databases and Durable Objects using SQLite storage, Local Explorer includes a SQL studio with:

- A visual table browser.
- Inline row editing.
- A SQL query editor for arbitrary queries.

This is useful for seeding test data, verifying writes from a Worker, debugging workflow runs, and inspecting Durable Object state.

## PlanetScale With Hyperdrive

Use [Cloudflare Hyperdrive](https://dash.cloudflare.com/?to=/:account/workers/hyperdrive?modal=1) when connecting Workers to PlanetScale Postgres. Hyperdrive pools and accelerates database connections from Workers, which is the recommended pattern for external PostgreSQL access.

Key references:

- Cloudflare blog: [Deploy PlanetScale Postgres with Workers](https://blog.cloudflare.com/deploy-planetscale-postgres-with-workers/)
- Cloudflare integration docs: [PlanetScale integration](https://developers.cloudflare.com/workers/databases/third-party-integrations/planetscale/)
- Hyperdrive provider example: [PlanetScale Postgres](https://developers.cloudflare.com/hyperdrive/examples/connect-to-postgres/postgres-database-providers/planetscale-postgres/)
- PlanetScale launch: [PlanetScale for Postgres](https://planetscale.com/blog/planetscale-for-postgres)
- PlanetScale and Hyperdrive: [Real-time apps with Cloudflare Hyperdrive](https://planetscale.com/blog/cloudflare-hyperdrive-real-time)

Create a Hyperdrive connection from the Cloudflare dashboard:

```text
https://dash.cloudflare.com/?to=/:account/workers/hyperdrive?modal=1
```

## Drizzle With PlanetScale

[Drizzle joined PlanetScale](https://planetscale.com/blog/drizzle-joins-planetscale), and Drizzle can be used from Workers through Hyperdrive when the backing database is PlanetScale Postgres.

Key references:

- Cloudflare docs: [Drizzle ORM with Hyperdrive](https://developers.cloudflare.com/hyperdrive/examples/connect-to-postgres/postgres-drivers-and-libraries/drizzle-orm/)
- PlanetScale announcement: [Drizzle Joins PlanetScale](https://planetscale.com/blog/drizzle-joins-planetscale)
- Demo: [Drizzle and PlanetScale](https://www.youtube.com/watch?v=nIjtRoQtp9w)

## Better Auth With Hono

[Better Auth's Hono integration](https://better-auth.com/docs/integrations/hono) is relevant for Workers apps that use Hono as the HTTP framework. Pair it with Better Auth provider docs when adding sign-in methods:

- Google: [Better Auth Google](https://better-auth.com/docs/authentication/google)
- Apple: [Better Auth Apple](https://better-auth.com/docs/authentication/apple)
- Stripe plugin: [Better Auth Stripe](https://better-auth.com/docs/plugins/stripe)

## API

Local Explorer exposes a local API at `/cdn-cgi/explorer/api`.

Fetch the OpenAPI specification:

```sh
curl http://localhost:8787/cdn-cgi/explorer/api
```

The API lets tools and coding agents discover available local binding operations and programmatically read or modify local development data.

## Worker Best Practices

Use `worker-best-practices` when writing, reviewing, or debugging Cloudflare Workers, especially changes involving Wrangler configuration, bindings, streaming, background work, Worker-to-Worker calls, secrets, observability, or runtime tests.

Key references:

- Cloudflare docs: [Workers Best Practices](https://developers.cloudflare.com/workers/best-practices/workers-best-practices/index.md)
- Local skill: `.agents/skills/worker-best-practices/SKILL.md`
- Workers types v5 changelog: [Simpler runtime types with @cloudflare/workers-types v5](https://developers.cloudflare.com/changelog/post/2026-07-03-workers-types-v5/)

The skill covers the production checklist from Cloudflare's Worker best-practices guide:

- Configuration: keep `compatibility_date` current, enable `nodejs_compat` when dependencies need Node.js built-ins, generate binding types with `wrangler types`, store secrets with `wrangler secret put` or `wrangler secret bulk`, and configure environments, custom domains, and routes deliberately.
- Request handling: stream large request and response bodies, enforce body size limits before buffering, and use `ctx.waitUntil()` only for work that does not affect the response.
- Architecture: prefer bindings over Cloudflare REST APIs from inside Workers, use Queues for decoupled background jobs, Workflows for durable multi-step processes, service bindings for Worker-to-Worker communication, Hyperdrive for external databases, Durable Objects for WebSockets, and Workers Static Assets for new static projects.
- Code patterns: do not store request-scoped state in module-level mutable variables, and do not leave floating promises that are not awaited, returned, or passed to `ctx.waitUntil()`.
- Production readiness: enable Workers Logs and Traces, use Web Crypto for security-sensitive randomness and constant-time secret comparison, avoid `passThroughOnException()` as normal error handling, and test with `@cloudflare/vitest-pool-workers`.

## Workers Runtime Types

Cloudflare released `@cloudflare/workers-types` v5 on 2026-07-03. The package now only exposes the latest runtime types at `@cloudflare/workers-types` and experimental APIs at `@cloudflare/workers-types/experimental`; dated entrypoints such as `@cloudflare/workers-types/2023-03-01` were removed.

When debugging TypeScript errors or runtime API mismatches in Workers projects, prefer generating project-specific types with:

```sh
npx wrangler types
```

This locks generated types to the Worker's configured `compatibility_date` and bindings. If a project imports dated `@cloudflare/workers-types/...` entrypoints, update it to use generated Wrangler types or the latest package entrypoint.
