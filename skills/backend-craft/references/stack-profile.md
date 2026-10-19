# Preferred stack (recommendation only)

These are the tools preferred by default. They are a **recommendation, never an imposition**:

- A choice the project already made always wins.
- When a stack decision comes up, offer the profile's choice as the recommended option **only if it meets the requirements**, next to real alternatives, and let the developer decide.
- If it does not fit (product needs, platform limits, team skills, existing code), say why and recommend something else.

## Language and repository

- **TypeScript, strict**, across the whole stack.
- **Bun** as package manager and script runner; **Bun workspaces + Turborepo** when there is more than one app or package.
- **oxlint + oxfmt** for lint and format; one `check` command (types + lint + format) that every task runs before it is done; **Vitest** for tests.
- CI on every push and pull request: frozen lockfile, `check`, then tests.
- Conventional Commits.

## Backend

- **Hono** on **Cloudflare Workers**.
- **D1 + Drizzle** for data, **R2** for files, Workers AI through the **AI SDK** when AI is needed (behind a role module).
- Validation with **Zod** at every boundary; auth with **better-auth**; email with **Resend**; error tracking with **Sentry**.
- Tests: **Vitest** with the Workers pool and a local D1 used only for tests.

## Product services (behind role modules)

- Subscriptions and in-app purchases: **RevenueCat**.
- Product analytics: **PostHog** (no user content or personal data in event properties).

## Decisions the profile does not settle — always ask

- How the client/server contract is kept in sync (types duplicated per app with contract tests, or a shared package).
- Where each component runs when the product's requirements do not fit Workers (see `platform-limits.md`).
