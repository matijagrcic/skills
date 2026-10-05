# Skills

Agent skills for building and reviewing TypeScript applications, with a focus on React, TanStack, API contracts, and Cloudflare Workers.

This is a collection of engineering guidance I maintain for coding agents: what to inspect before making a change, which patterns to follow, and how to check the result. The emphasis is on readable code, types that stay connected across boundaries, and fixes grounded in the project already in front of you.

## Install

From your project directory, use the [Skills CLI](https://github.com/vercel-labs/skills) to choose the skills you want:

```sh
npx skills@latest add matijagrcic/skills
```

Or install an individual skill:

```sh
npx skills@latest add matijagrcic/skills --skill bulletproof-react-components
```

**Companion files:** `api-contract-pipeline`, `tanstack-project-patterns`, and `worker-best-practices` read reference documents from your project's root. The skill install alone does not include these root documents. Copy [api-contracts.md](api-contracts.md), [tanstack.md](tanstack.md), and [cloudflare.md](cloudflare.md) into your project as well; they cross-reference one another for work that spans the stack.

[AGENTS.md](AGENTS.md) contains the shared engineering principles and task-to-skill routing. Merge the relevant sections into your existing agent instructions, keeping your project's own commands and conventions.

For a manual setup, copy the skill folders you want from [`.agents/skills`](.agents/skills) into your project's `.agents/skills/` directory, along with the companion files above.

## Available skills

| Skill | Use it when |
| --- | --- |
| [api-contract-pipeline](.agents/skills/api-contract-pipeline/SKILL.md) | Changing server contracts, OpenAPI, Orval clients, Zod schemas, or MSW mocks, and keeping their types connected through to the UI. |
| [bulletproof-react-components](.agents/skills/bulletproof-react-components/SKILL.md) | Building or reviewing React components for hydration, multiple instances, concurrent rendering, portals, and server/client data boundaries. |
| [papercuts](.agents/skills/papercuts/SKILL.md) | Recording small recurring tooling and workflow annoyances in `.agents/PAPERCUTS.md` without interrupting the current task. |
| [tanstack-project-patterns](.agents/skills/tanstack-project-patterns/SKILL.md) | Working with Query, Router, Start, Form, Virtual, AI, or Intent, especially where routing, caching, and rendering meet. |
| [worker-best-practices](.agents/skills/worker-best-practices/SKILL.md) | Building or reviewing Cloudflare Workers, Wrangler configuration, bindings, background work, streaming, and runtime safety. |

## Usage

Ask your agent to use a skill by name and give it a concrete task:

```text
Use bulletproof-react-components to review this dialog for hydration issues
and problems when two instances are rendered on the same page.
```

```text
Use api-contract-pipeline to add pagination to this endpoint and carry the
contract change through the generated client, validation, and mocks.
```

```text
Use tanstack-project-patterns to investigate why this route refetches data
immediately after navigation.
```

```text
Use worker-best-practices to review this Worker's bindings, request handling,
and background tasks before deployment.
```

```text
Use papercuts to record this recurring setup issue, then continue the task.
```

Each skill defines its scope and workflow. Apply the guidance that matches your project, check installed versions, and use the project's existing tools to verify changes.

## Reference documents

The skills describe the workflow; the root documents hold the longer examples and stack guidance.

| Document | Contents |
| --- | --- |
| [AGENTS.md](AGENTS.md) | Engineering principles and rules for loading the relevant skills and references. |
| [api-contracts.md](api-contracts.md) | End-to-end type flow, Orval generation, Query clients, Zod schemas, and MSW composition. |
| [tanstack.md](tanstack.md) | Query and Router integration, Start rendering modes, forms, virtualization, and Intent. |
| [cloudflare.md](cloudflare.md) | Worker conventions, local tooling, bindings, Workflows, databases, and runtime types. |

## Sources and inspiration

The skills and references link to the material they build on. In particular, `bulletproof-react-components` draws on Shu Ding's [Building Bulletproof React Components](https://shud.in/thoughts/build-bulletproof-react-components), and `worker-best-practices` uses [Cloudflare's Workers guidance](https://developers.cloudflare.com/workers/best-practices/workers-best-practices/).

For other thoughtful skill collections, see [Lauren Tan's pstack](https://github.com/cursor/plugins/tree/main/pstack), [HumanLayer's skills](https://github.com/humanlayer/skills), and [Emil Kowalski's skills](https://github.com/emilkowalski/skills). Their READMEs inspired the presentation of this collection.
