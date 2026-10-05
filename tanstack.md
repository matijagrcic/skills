# TanStack Intent

[TanStack Intent](https://tanstack.com/intent/latest/docs/overview) helps coding agents discover and load library-provided agent skills from installed npm packages.

Use Intent when working with TanStack AI or other packages that ship `SKILL.md` files, so Cursor, Claude Code, GitHub Copilot, Codex, and similar coding agents can follow package-specific guidance instead of relying on stale model knowledge.

For OpenAPI contracts, Orval generation, generated TanStack Query clients, generated Zod schemas, MSW mocks, or end-to-end API type flow, also read [`api-contracts.md`](./api-contracts.md). That material is maintained separately so Query, Router, Start, Form, Virtual, Intent, and auth guidance remain available without mixing API generation into every TanStack task.

## Agent Skills

[TanStack AI agent skills](https://tanstack.com/ai/latest/docs/getting-started/agent-skills) are markdown files published inside TanStack AI packages. After installation, they live under `node_modules/<package>/skills/<skill-name>/SKILL.md` and teach coding agents the correct TanStack AI APIs and patterns.

TanStack AI currently documents skills for:

- `@tanstack/ai`: chat experiences, tool calling, adapters, middleware, structured outputs, media generation, AG-UI protocol, and custom backends.
- `@tanstack/ai-code-mode`: setting up Code Mode with a sandbox driver and registering server tools.

## Install Intent

Use the Intent CLI from the project root:

```sh
pnpm dlx @tanstack/intent@latest install
```

Use `--map` when you want Intent to generate or refresh task-to-skill mappings:

```sh
pnpm dlx @tanstack/intent@latest install --map
```

The installer scans `node_modules` for intent-enabled packages, proposes mappings for relevant tasks, and writes them into the agent configuration file, such as `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or another file you choose.

## Consumer Quick Start

Follow the [Intent quick start for consumers](https://tanstack.com/intent/latest/docs/getting-started/quick-start-consumers) when adding Intent to an application that consumes packages with bundled skills.

Typical flow:

1. Install the package that ships skills, such as `@tanstack/ai`.
2. Run `pnpm dlx @tanstack/intent@latest install` from the project root.
3. Review the generated task mappings and tighten the task descriptions if needed.
4. Start a fresh agent session and confirm the agent loads the mapped skill before editing TanStack AI code.

Re-run Intent when you add a new intent-enabled package or want to refresh the generated mappings.

## Ultracite For TanStack Apps

[Ultracite](https://www.ultracite.ai/) is a zero-config preset for Biome, ESLint, Oxlint, Prettier, and Stylelint that can generate editor, agent, and hook configuration for consistent code quality.

Use Ultracite when setting up or refreshing linting and formatting in TanStack apps, especially projects using TanStack Query, Router, or Start. As of [Ultracite 7.8.3](https://github.com/haydenbleasel/ultracite/releases/tag/ultracite%407.8.3), TanStack apps have their own preset with framework detection for Query, Router, and Start.

Agent notes:

- Run the latest initializer from the project root and let it detect the package manager and frameworks.
- Prefer the TanStack preset when Ultracite detects TanStack Query, Router, or Start.

## TanStack Virtual Chat

[TanStack Virtual Chat](https://tanstack.com/blog/tanstack-virtual-chat) documents patterns for building high-performance chat timelines with large or dynamic message lists.

Use TanStack Virtual for React chat UIs that need virtualized rows, dynamic message heights, streaming updates, scroll restoration, and preserving the user's viewport while new messages are appended or older messages are prepended.

Key references:

- Chat guide: [TanStack Virtual Chat](https://tanstack.com/virtual/latest/docs/chat)
- React example: [Virtualized Chat](https://tanstack.com/virtual/latest/docs/framework/react/examples/chat)
- Example source: [TanStack Virtual React Chat](https://github.com/TanStack/virtual/tree/main/examples/react/chat)

The React example is the most direct starting point for implementing a virtualized chat list in an application.

## TanStack Router And Query

TkDodo's TanStack Router articles explain the router's type-safety model, search-param state management, file-based routing, Suspense integration, and how to combine Router loaders with TanStack Query without letting two caches fight each other.

Use this guidance when building TanStack Router, TanStack Start, or React Query routes that need early data loading, prefetch-on-intent, SSR seeding, or shared server state across routes.

Key references:

- Router overview: [The Beauty of TanStack Router](https://tkdodo.eu/blog/the-beauty-of-tan-stack-router)
- Router and Query: [TanStack Router and Query](https://tkdodo.eu/blog/tan-stack-router-and-query)

### Agent Rules

- Prefer route-bound hooks such as `Route.useParams()`, `Route.useSearch()`, or `getRouteApi()` when a component belongs to one route. Use `useParams({ from: "/route/$param" })` or `useSearch({ from: "/route" })` when calling global hooks directly.
- Use `strict: false` only for reusable components that intentionally read params or search across multiple routes; handle optional values explicitly.
- Use typed `Link` and navigation APIs with route patterns plus `params` and `search` objects instead of manually interpolating URL strings.
- Validate search params on the route with `validateSearch` so user-editable URL state is parsed, typed, and serialized by the router. Use Standard Schema-compatible validators when possible.
- Use hook selectors, such as `useSearch({ from, select })`, `useParams({ from, select })`, `useLoaderData({ from, select })`, or `useRouterState({ select })`, to subscribe to the smallest needed slice and avoid rerendering unrelated route UI.
- Prefer file-based routing for most apps because it gives predictable route ownership, fast URL-to-file lookup, automatic route code splitting, and generated route types. Use code-based or virtual routes only when the project has a clear need.
- Lean on route-level Suspense and error boundaries. It is fine for loaders to start `queryClient.prefetchQuery(...)` without awaiting when route components read the same data with `useSuspenseQuery`.
- Use Router loader data for route-specific data that is only needed by one route tree.
- Use TanStack Query for data shared across routes because the Query cache is global and addressable by `queryKey`.
- Start fetches in route loaders so data can load before the component renders and before route bundles finish evaluating.
- Put the same `QueryClient` in both Router context and `QueryClientProvider`; separate clients mean separate caches.
- Turn off Router preload caching when Query owns server-state caching by setting `defaultPreloadStaleTime: 0`.
- Treat loaders as event handlers that prime the Query cache. Avoid using `Route.useLoaderData()` as the long-term read path for Query-owned data.
- In components, read Query-owned data with `useQuery` or `useSuspenseQuery` so Query creates active observers and can refetch on invalidation, focus, reconnect, and other triggers.
- Prefer `useSuspenseQuery` for blocking route data because TanStack Router already provides route-level pending and error boundaries.
- Use `useQuery` for deferred or non-blocking data where `data` may temporarily be `undefined`.
- In TanStack Start SSR, make sure server-fetched data lands in the client Query cache. `useSuspenseQuery` works well with streaming SSR; `useQuery` needs data awaited in the loader if the initial HTML must include that content.

### Recommended Pattern

Define reusable query option factories and use the same options in loaders and components:

```ts
const dashboardQueryOptions = (dashboardId: string) =>
  queryOptions({
    queryKey: ["dashboard", dashboardId],
    queryFn: () => fetchDashboard(dashboardId),
  });
```

Prime the cache from the loader:

```ts
export const Route = createFileRoute("/dashboard/$dashboardId")({
  loader: ({ context, params }) => {
    context.queryClient.prefetchQuery(
      dashboardQueryOptions(params.dashboardId),
    );
  },
  component: Dashboard,
});
```

Read from Query in the route component:

```ts
function Dashboard() {
  const { dashboardId } = Route.useParams()
  const { data } = useSuspenseQuery(dashboardQueryOptions(dashboardId))

  return <h1>{data.title}</h1>
}
```

Use `ensureQueryData` and `await` in the loader when you explicitly want the loader to block before rendering. Use `prefetchQuery` without awaiting when the loader should only start the request early and let the component decide whether it blocks with `useSuspenseQuery` or renders deferred UI with `useQuery`.

## TanStack Start Rendering Modes

Use TanStack Start's rendering controls when a route needs browser-only APIs, when SSR is not valuable for the app, or when data should load on the server without rendering the route component on the server.

Key references:

- Selective SSR: [TanStack Start Selective Server-Side Rendering](https://tanstack.com/start/latest/docs/framework/react/guide/selective-ssr)
- SPA mode: [TanStack Start SPA mode](https://tanstack.com/start/v0/docs/framework/react/guide/spa-mode)

### Selective SSR

TanStack Start server-renders matching routes on the initial request by default. Use the route-level `ssr` property to control whether `beforeLoad`, `loader`, and the route component run on the server for that initial request.

Agent rules:

- Leave `ssr` unset or set `ssr: true` when the route can safely run on the server and should send rendered HTML.
- Use `ssr: false` when `beforeLoad`, `loader`, or the component depends on browser-only APIs such as `localStorage`, `window`, or `canvas`; this prevents server execution of `beforeLoad` and `loader` and prevents server rendering of the component.
- Use `ssr: "data-only"` when `beforeLoad` and `loader` should run on the server, but the route component should render only on the client.
- Use the functional `ssr` form when SSR behavior depends on validated params or search, such as disabling SSR for one document type or switching to `"data-only"` when a query flag is present.
- Set `defaultSsr: false` in `createStart` only when client rendering should be the app-wide default.
- Remember that child routes inherit parent SSR settings and can only become more restrictive: `true` can become `"data-only"` or `false`, and `"data-only"` can become `false`.
- Configure a `pendingComponent` or `defaultPendingComponent` for the first route with `ssr: false` or `ssr: "data-only"` so the server has a useful fallback to render.
- The root route's HTML shell still renders on the server. To disable root route component SSR, define a `shellComponent` that renders `<html>`, `<head>`, `<body>`, `HeadContent`, `Scripts`, and `{children}`, then set `ssr: false` on the root route or `defaultSsr: false`.

Example:

```tsx
export const Route = createFileRoute("/reports/$reportId")({
  ssr: "data-only",
  loader: ({ params }) => loadReport(params.reportId),
  component: ReportPage,
});
```

### SPA Mode

Use SPA mode for Start apps that do not need SSR for SEO, crawlers, or initial render performance. SPA mode still works with server functions, server routes, and external APIs; it only means the initial document contains a static shell instead of fully rendered route HTML.

Agent rules:

- Enable SPA mode in the Start plugin with `spa: { enabled: true }`.
- Expect the build to prerender the root route shell and write it to `/_shell.html` by default.
- Configure hosting redirects so static assets are served first, server function or API paths are allow-listed, and all remaining 404s rewrite to the SPA shell.
- Keep the default shell mask path `/` unless the app has a clear reason to generate the shell from another pathname.
- Override SPA prerender options only when needed; defaults include `outputPath: "/_shell.html"`, `crawlLinks: false`, and `retryCount: 0`.
- Use `router.isShell()` only for shell-specific rendering, and avoid UI that flashes during hydration when the router immediately navigates from the shell to the real route.
- Root route loaders and server-only root functionality can still run while prerendering the shell, so avoid dynamic data in the shell unless it is intended to be baked into the static output.

Basic Vite setup:

```ts
export default defineConfig({
  plugins: [
    tanstackStart({
      spa: {
        enabled: true,
      },
    }),
  ],
});
```

## TanStack Form

[TanStack Form](https://tanstack.com/form/latest) is a headless, type-safe form state library for building complex interactive forms with typed fields, granular subscriptions, nested values, async validation, and framework adapters.

Use TanStack Form when forms need inferred submit values, field-level validation, nested object or array fields, debounced async checks, or UI built from existing product components instead of a prescriptive form wrapper.

Key references:

- shadcn/ui guide: [TanStack Form with shadcn/ui](https://ui.shadcn.com/docs/forms/tanstack-form)
- Product docs: [TanStack Form](https://tanstack.com/form/latest)
- Feature matrix: [TanStack Form comparison](https://tanstack.com/form/latest/docs/comparison)

The shadcn/ui guide shows TanStack Form with `@tanstack/react-form`, Zod validation, render-prop fields, shadcn `Field` components, accessible errors, validation modes, array fields, and nested fields.

### shadcn/ui Form Pattern

When using TanStack Form with shadcn/ui forms, prefer TanStack Form for state and validation while using shadcn/ui's `Field` primitives for markup, spacing, descriptions, and errors. Do not use shadcn's React Hook Form-specific wrapper components for TanStack Form forms.

Core rules:

- Create the form with `useForm` from `@tanstack/react-form`.
- Define `defaultValues` for every field so controls are always controlled.
- Put Zod or another Standard Schema validator under `validators`, usually `onSubmit` for submit-time validation or `onChange` / `onBlur` for live validation.
- Render each field with `form.Field` and its render prop. Inside the render prop, wire the UI control directly to `field.state.value`, `field.handleChange`, and `field.handleBlur`.
- Use shadcn/ui `Field`, `FieldGroup`, `FieldSet`, `FieldLabel`, `FieldDescription`, and `FieldError` to build the accessible form structure.
- Compute invalid state from metadata, commonly `field.state.meta.isTouched && !field.state.meta.isValid`.
- Put `data-invalid={isInvalid}` on `Field` or the containing field primitive, and put `aria-invalid={isInvalid}` on the actual control, such as `Input`, `Textarea`, `SelectTrigger`, `Checkbox`, `RadioGroupItem`, or `Switch`.
- Render `<FieldError errors={field.state.meta.errors} />` next to the field when invalid.
- On submit, call `e.preventDefault()` and then `form.handleSubmit()`.
- Use `form.reset()` for reset buttons.
- Keep browser validation enabled in production when it adds value; the shadcn demo disables it only to demonstrate schema errors.

Basic shape:

```tsx
const form = useForm({
  defaultValues: {
    title: "",
  },
  validators: {
    onSubmit: formSchema,
  },
  onSubmit: async ({ value }) => {
    // Persist or submit validated values.
  },
});

return (
  <form
    onSubmit={(event) => {
      event.preventDefault();
      form.handleSubmit();
    }}
  >
    <FieldGroup>
      <form.Field
        name="title"
        children={(field) => {
          const isInvalid =
            field.state.meta.isTouched && !field.state.meta.isValid;

          return (
            <Field data-invalid={isInvalid}>
              <FieldLabel htmlFor={field.name}>Title</FieldLabel>
              <Input
                id={field.name}
                name={field.name}
                value={field.state.value}
                onBlur={field.handleBlur}
                onChange={(event) => field.handleChange(event.target.value)}
                aria-invalid={isInvalid}
              />
              {isInvalid && <FieldError errors={field.state.meta.errors} />}
            </Field>
          );
        }}
      />
    </FieldGroup>
  </form>
);
```

Component-specific wiring:

- For `Input` and `Textarea`, pass `field.state.value`, `field.handleBlur`, and `field.handleChange(event.target.value)`.
- For `Select`, pass `value={field.state.value}` and `onValueChange={field.handleChange}` to `Select`; put `aria-invalid` on `SelectTrigger`.
- For `Checkbox`, pass `checked={field.state.value}` and `onCheckedChange={field.handleChange}` for a single boolean field.
- For checkbox arrays, set `mode="array"` on `form.Field`, use `field.pushValue(item)` and `field.removeValue(index)`, and add `data-slot="checkbox-group"` to the checkbox `FieldGroup` for shadcn spacing.
- For `RadioGroup`, pass `value={field.state.value}` and `onValueChange={field.handleChange}` to `RadioGroup`; put `aria-invalid` on each `RadioGroupItem`.
- For `Switch`, pass `checked={field.state.value}` and `onCheckedChange={field.handleChange}`.
- For nested array fields, put `mode="array"` on the parent field and use bracket notation for child fields, such as `emails[${index}].address`.

## Better Auth With TanStack Start

[Better Auth's TanStack integration](https://better-auth.com/docs/integrations/tanstack) documents using Better Auth with TanStack Start applications.

Use it when adding session handling, route protection, sign-in flows, or provider-backed auth to a TanStack Start app. Install the Better Auth skills first when you want agent guidance for setup:

```sh
npx skills add better-auth/skills
```

Related Better Auth references:

- Skills: [Better Auth AI Resources](https://better-auth.com/docs/ai-resources/skills)
- Google sign-in: [Google Authentication](https://better-auth.com/docs/authentication/google)
- Apple sign-in: [Apple Authentication](https://better-auth.com/docs/authentication/apple)
- Stripe plugin: [Stripe Plugin](https://better-auth.com/docs/plugins/stripe)
