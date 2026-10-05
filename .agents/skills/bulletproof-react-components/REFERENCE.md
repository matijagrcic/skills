# Bulletproof React Component Approaches

This reference expands the checklist in `SKILL.md`. Use it as a decision guide, not a mandate to add every pattern to every component.

Source: [Building Bulletproof React Components](https://shud.in/thoughts/build-bulletproof-react-components)

## 1. Server-Proof

Use when a component may render in SSR, React Server Components, Next.js, Remix, or build-time rendering.

Watch for:

- Browser globals read during render: `window`, `document`, `localStorage`, `sessionStorage`, `matchMedia`, `ResizeObserver`.
- Lazy initial state that still reads browser APIs during server render.

Prefer:

- Move browser reads into `useEffect`, event handlers, or client-only boundaries.
- Initialize server-safe defaults, then update after mount.
- For required initial values, use framework-supported request data, cookies, headers, or a pre-hydration script.

Avoid:

- `typeof window` branches that render different markup on server and client unless hydration behavior is intentional and tested.

## 2. Hydration-Proof

Use when client state can differ from server output, such as theme, locale, auth-adjacent UI, media queries, or persisted preferences.

Watch for:

- Flash of wrong theme or layout.
- Hydration mismatch warnings.
- Server default state that changes immediately after mount.

Prefer:

- Make the first server render and first client render agree.
- Use inline pre-hydration scripts for visual preferences that must be correct before paint.
- Use cookies or server-readable preference storage when possible.

Avoid:

- Rendering one branch on the server and a different branch on first client render without suppressing or isolating the mismatch.

## 3. Instance-Proof

Use when a component can appear more than once on a page or inside repeated layouts.

Watch for:

- Hardcoded DOM IDs.
- Module-level mutable state used as component instance state.
- `document.querySelector()` targeting the first matching element globally.

Prefer:

- `useId()` for stable server/client IDs.
- Refs for local DOM access.
- Per-instance state and context providers.

Avoid:

- Random IDs generated during render, because they can destabilize hydration.

## 4. Concurrent-Proof

Use when Server Components, Suspense, concurrent rendering, or multiple component instances can request the same data.

Watch for:

- Duplicate database or API calls in a single request.
- Shared mutable caches that cross users or requests.
- Logic that assumes one render pass per update.

Prefer:

- React `cache()` for request-scoped async deduplication in Server Components.
- Framework data caches with explicit invalidation semantics.
- Idempotent render logic with side effects moved out of render.

Avoid:

- Global process caches for user-specific data unless they include safe scoping, invalidation, and isolation.

## 5. Composition-Proof

Use when a component receives arbitrary children, library consumers can pass unknown nodes, or the project uses RSC, `React.lazy`, Suspense, or async children.

Watch for:

- `React.cloneElement(children, extraProps)` used as the only communication path.
- Assumptions that `children` is a concrete React element.
- Components that inspect child types or props deeply.

Prefer:

- Context for shared state and actions.
- Explicit slot props for known extension points.
- Render props only when consumers need computed state or callbacks.

Avoid:

- Deep child tree mutation, because async or server-owned children can be opaque.

## 6. Portal-Proof

Use when components can render in portals, iframes, pop-outs, embedded documents, shadow hosts, or test environments with custom documents.

Watch for:

- Event listeners attached to global `window` or `document`.
- Measurements or selections taken from the wrong document.
- Keyboard shortcuts that fail in pop-out windows or iframe contexts.

Prefer:

- Store a ref to the rendered element.
- Resolve `const doc = ref.current?.ownerDocument`.
- Resolve `const win = doc?.defaultView`.
- Attach listeners to the owning document or window and clean them up from the same target.

Avoid:

- Assuming the top-level browser `window` is the component's execution context.

## 7. Transition-Proof

Use when React view transitions or transition-aware UI state should animate rather than snap.

Watch for:

- State changes inside a `<ViewTransition>` that do not animate.
- Expensive UI changes scheduled as urgent updates.

Prefer:

- Wrap transition-participating state changes in `startTransition`.
- Keep urgent input updates separate from non-urgent visual transitions.

Avoid:

- Using transitions to hide slow data fetching or logic that should be optimized directly.

## 8. Activity-Proof

Use when hidden UI may preserve DOM, such as React Activity-style APIs, tab panels, offscreen UI, preserved routes, or manually hidden subtrees.

Watch for:

- `<style>` tags that keep applying global CSS while the component is hidden.
- Global event listeners, observers, timers, or subscriptions that remain active while hidden.
- DOM-level side effects outside React state.

Prefer:

- Disable global styles when hidden, for example by toggling `media`.
- Clean up subscriptions and observers when the component is inactive.
- Scope styles and side effects to the smallest possible DOM subtree.

Avoid:

- Assuming hidden means unmounted.

## 9. Leak-Proof

Use when Server Components or server-only modules pass data into components that may later become client components or render unknown children.

Watch for:

- Passing full user/session objects when only a display field is needed.
- Tokens, credentials, private metadata, or internal IDs bundled into props.
- Third-party or loosely owned components receiving sensitive objects.

Prefer:

- Narrow objects before passing them across component boundaries.
- Keep secrets in server-only modules and functions.
- Use React experimental taint APIs, where available, to fail fast if sensitive values reach the client.

Avoid:

- Relying on today's component implementation to remain server-only forever.

## 10. Future-Proof Persistence

Use when a value must remain stable for correctness, identity, or user-visible continuity.

Watch for:

- `useMemo` used to generate random values, IDs, accent colors, or semantic object identity.
- Correctness that depends on memoized values never being discarded.

Prefer:

- `useState` lazy initializers for values that must persist for the component lifetime.
- `useRef` for stable mutable values that do not trigger rendering.
- Server or durable storage for values that must survive remounts.

Avoid:

- Treating `useMemo` as a semantic guarantee. It is a performance optimization and React may discard it.

## Choosing An Approach

- New reusable component: check all ten categories lightly, then apply the ones that match its API and runtime.
- SSR bug: start with server-proof and hydration-proof checks.
- Duplicate requests: start with concurrent-proof checks.
- Component library API issue: start with instance-proof and composition-proof checks.
- Portal, iframe, or pop-out bug: start with portal-proof checks.
- Visual transition issue: start with transition-proof checks.
- Hidden UI side effect: start with activity-proof checks.
- Secret or token exposure risk: start with leak-proof checks.
- Flickering generated values: start with future-proof persistence checks.
