# Recommended stack profile (shared)

The tools the leshi owner uses by default. They are a **recommendation, never an imposition**: when a stack decision comes up (craft §2, `bootstrap-technology` §4), offer the profile's choice as the recommended option **only if it meets the requirements**, next to real alternatives, and let the developer decide. A choice the project already recorded always wins. If the profile's choice does not fit (product needs, platform limits, team skills, existing code), say why and recommend something else.

## Language and repository

- **TypeScript, strict**, across the whole stack.
- **Bun** as package manager and script runner; **Bun workspaces + Turborepo** when there is more than one app or package.
- **oxlint + oxfmt** for lint and format; one `check` command (types + lint + format) that every task runs before it is done; Vitest for tests.
- CI on every push and pull request: frozen lockfile, `check`, then tests.
- Conventional Commits.

## Mobile

- **Expo** with **Expo Router**; EAS Build, and EAS Update for over-the-air releases with channels.
- React Compiler on (no manual memoization).
- Styling: **Unistyles** (tokens and variants).
- Server state: **TanStack Query** with **query-key-factory**; forms: **TanStack Form**; validation: **Zod**.
- Storage: **MMKV** for preferences; **expo-sqlite + Drizzle** when the app is offline-first.
- Lists: **LegendList**; motion: **Reanimated**; keyboard: **react-native-keyboard-controller**; toasts: **sonner-native**; i18n: **i18next**; dates: **dayjs**.

## Web

- Admin or internal apps: **Vite + React** with **Tailwind**, **TanStack Query** and a client router.
- Landing pages: static, served from the same platform as the backend.

## Backend

- **Hono** on **Cloudflare Workers**; **D1 + Drizzle** for data, **R2** for files, Workers AI through the **AI SDK** when AI is needed (behind a role module).
- Validation with **Zod** at every boundary; auth with **better-auth**; email with **Resend**; error tracking with **Sentry**.
- Tests: **Vitest** with the Workers pool and a local D1 only for tests (craft §9).

## Product services

- Subscriptions and in-app purchases: **RevenueCat**.
- Product analytics: **PostHog** (no user content or personal data in event properties).

## Decisions the profile does not settle — always ask

- How the client/server contract is kept in sync (types duplicated per app with contract tests, or a shared package).
- The frontend test policy (logic only plus device QA, plus component tests, plus E2E).
