---
name: bulletproof-react-components
description: Harden React components against SSR, hydration, multiple instances, concurrent rendering, async composition, portals, transitions, hidden activity, data leaks, and memoization pitfalls. Use when building, reviewing, or refactoring React components for Next.js, Remix, React Server Components, React 19 features, reusable component libraries, or production robustness.
---

# Bulletproof React Components

Use this skill to build or review React components that keep working outside the happy path: server rendering, hydration, multiple instances, concurrent rendering, portals, async children, transitions, hidden trees, and client/server data boundaries.

Primary reference:

- [Building Bulletproof React Components](https://shud.in/thoughts/build-bulletproof-react-components)

## Quick Start

When touching a React component, first identify the environments it may run in:

1. Server rendered, client only, or mixed Server/Client Components.
2. Single instance or many instances on the same page.
3. Direct DOM tree, portal, iframe, pop-out, or embedded document.
4. Synchronous children, async children, lazy children, or opaque RSC references.
5. Visible only, hidden with preserved DOM, or animated with transitions.
6. Public data only or data that includes secrets, tokens, private user state, or server-only values.

Then apply only the hardening patterns that match the component's real risks.

## Review Workflow

1. Check server safety: browser APIs such as `window`, `document`, `localStorage`, `matchMedia`, and layout reads must not run during server render.
2. Check hydration behavior: server markup and first client render should agree, or intentional client-only changes should avoid visible flashes and mismatches.
3. Check instance safety: avoid hardcoded DOM IDs, global singleton state, and selectors that break when two copies render.
4. Check concurrent behavior: deduplicate request-scoped async work and avoid assuming render order or one render per update.
5. Check composition: prefer context, slots, or explicit APIs over `cloneElement` when children may be async, lazy, or server-owned.
6. Check document ownership: attach DOM listeners and measurements to the component's `ownerDocument` or `defaultView` when portals or iframes are possible.
7. Check transitions: use `startTransition` for state changes that should participate in React view transitions.
8. Check hidden activity: DOM side effects such as global styles must be disabled or cleaned up when a subtree is preserved but hidden.
9. Check data leaks: never pass sensitive server values into unknown components without narrowing the data or using taint APIs where available.
10. Check persistence semantics: use state, refs, or durable storage when correctness depends on a stable value; do not rely on `useMemo` for semantic persistence.

## Implementation Rules

- Prefer invariants over blanket defensive code. Fix the real failure mode rather than adding broad fallbacks.
- Keep browser-only reads in effects, event handlers, or guarded client code unless the framework provides a pre-hydration script pattern.
- Use `useId()` for stable per-instance IDs instead of hardcoded IDs or random IDs generated during render.
- Use React `cache()` for request-scoped deduplication in Server Components when the same async lookup can be called concurrently.
- Use context for data that descendants need, especially across RSC, `lazy`, async children, or "use cache" boundaries.
- Use `ref.current?.ownerDocument.defaultView` for event targets when the component may render in another document.
- Treat React experimental APIs such as tainting and `Activity`-related behavior as project-version dependent; verify support before applying them.

## Approach Reference

See [REFERENCE.md](REFERENCE.md) for the specific hardening approaches, symptoms, and preferred fixes from the source article.

## Review Checklist

- Does the component render safely on the server?
- Will hydration produce the same initial structure and avoid a user-visible flash?
- Can multiple instances coexist without shared DOM IDs or global collisions?
- Are async reads deduplicated where duplicate concurrent calls matter?
- Does the component work with async, lazy, server, or opaque children?
- Do event listeners, measurements, and DOM writes target the right document?
- Are hidden or preserved DOM side effects cleaned up?
- Are secrets kept server-only across component boundaries?
- Is any value that must stay stable stored with semantic persistence rather than `useMemo`?
