# API Contracts, End-To-End Types, And Generated Clients

Use this reference for OpenAPI contracts, Orval generation, generated TanStack Query clients, generated Zod schemas, MSW mocks, and type flow from a database or server contract into React UI. Also read [`tanstack.md`](./tanstack.md) when the task changes Query behavior, Router loaders, TanStack Start, TanStack Form, or another TanStack runtime concern.

## End-To-End Type Flow

Use this guidance when working with TanStack Query, TanStack Router, TanStack Start, oRPC-backed OpenAPI contracts, Orval generated clients, or any stack where server data flows into React UI.

Let types flow from the source of truth all the way to the client. Database schema, oRPC/server contracts, the OpenAPI spec, Orval outputs, TanStack Query options, and UI components should share types without manual duplication. Use the project's existing Orval configuration to generate client types, Query hooks/options, Zod schemas, and mocks from the OpenAPI contract; do not introduce a parallel type system just to satisfy one call site.

Agent rules:

- Preserve branded and refined types across the boundary. A `users.email` value branded as `Email` in the database or server contract should still be `Email` when a component receives it from a TanStack Query hook.
- Derive types from source values before writing a new interface. Prefer `typeof`, `Pick`, `Omit`, `Parameters`, `ReturnType`, `Awaited`, indexed access types, generated client types, and procedure output helpers over restating object shapes.
- For TanStack Query, keep query functions and `queryOptions(...)` factories close to the typed server/client call so `useQuery` and `useSuspenseQuery` can infer data and error types. Avoid manually passing generics to hooks unless inference cannot represent the contract.
- For oRPC or TanStack Start server functions, infer input and output from the router, procedure, server function, generated OpenAPI client, or generated Orval schema rather than recreating request and response DTOs by hand.
- When a derived type becomes unwieldy, name the derived type. Do not replace it with a manually copied structural type that can drift from the schema or server contract.

Do not duplicate an API response shape just to type a component argument:

```ts
// Don't: duplicates the OpenAPI/Orval model and can lose branded or refined types.
type UserSummary = {
  id: string;
  email: string;
  displayName: string | null;
};

function UserRow({ user }: { user: UserSummary }) {
  // ...
}
```

Derive the UI type from the generated Orval contract instead:

```ts
import { useListUsers } from "@/lib/api/generated/users/users";

type ListUsersData = NonNullable<ReturnType<typeof useListUsers>["data"]>;
type UserSummary = Pick<
  ListUsersData["items"][number],
  "id" | "email" | "displayName"
>;

function UserRow({ user }: { user: UserSummary }) {
  // ...
}
```

## Orval Generated Query And Zod

Use Orval as the generation layer for the OpenAPI contract. The same `orval.config.ts` should produce TanStack Query clients, typed request/response models, Zod schemas, and MSW mocks from the same spec. This gives Router loaders and route components generated query hooks/options, while TanStack Form can reuse generated Zod schemas for validation.

Example `orval.config.ts`:

```ts
import {
  OutputClient,
  OutputHttpClient,
  OutputMockType,
  OutputMode,
  defineConfig,
} from "orval";

export default defineConfig({
  api: {
    input: {
      target: "./spec/api.yaml",
    },
    output: {
      target: "./lib/api/generated/index.ts",
      schemas: "./lib/api/generated/model",
      client: OutputClient.REACT_QUERY,
      httpClient: OutputHttpClient.AXIOS,
      mode: OutputMode.TAGS_SPLIT,
      clean: true,
      mock: {
        type: OutputMockType.MSW,
        indexMockFiles: true,
      },
      override: {
        mutator: {
          path: "./lib/api/client.ts",
          name: "apiMutator",
        },
      },
    },
  },
  apiZod: {
    input: {
      target: "./spec/api.yaml",
    },
    output: {
      target: "./lib/api/generated/zod/index.ts",
      schemas: {
        path: "./lib/api/generated/zod/model",
        type: "zod",
      },
      client: OutputClient.ZOD,
      mode: OutputMode.TAGS_SPLIT,
      clean: true,
    },
  },
});
```

The `api` output produces the TanStack Query integration, typed client models, and MSW mocks; the `apiZod` output produces Zod schemas that can be shared with TanStack Form validators. Keep generated artifacts under Orval ownership and regenerate them when the OpenAPI spec changes. When a type, mock, or validation schema is wrong, prefer fixing the OpenAPI schema, examples, or Orval configuration instead of hand-writing divergent client types, mock responses, or validators.

### MSW Handler Composition

Load generated Orval handler groups from a single `getMockHandlers` function. This gives MSW one stable entry point while keeping the actual handlers generated from the OpenAPI contract.

Generic `mocks/handlers.ts` pattern:

```ts
import { getApiMock, getHealthMock } from "@/lib/api/generated/index.msw";
import { getAccountsMock } from "@/lib/api/generated/accounts/accounts.msw";
import { getAuthMock } from "@/lib/api/generated/auth/auth.msw";
import { getProjectsMock } from "@/lib/api/generated/projects/projects.msw";
import { getUsersMock } from "@/lib/api/generated/users/users.msw";

export function getMockHandlers() {
  return [
    ...getAccountsMock(),
    ...getAuthMock(),
    ...getProjectsMock(),
    ...getUsersMock(),
    ...getApiMock(),
    ...getHealthMock(),
  ];
}
```

## Related References

- Read [`tanstack.md`](./tanstack.md) for Query, Router, Start, Form, Virtual, Intent, and Better Auth integration guidance.
- Read [`cloudflare.md`](./cloudflare.md) when the contract is implemented by a Worker or Hono on Cloudflare.
