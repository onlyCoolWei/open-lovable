# Repository Guidelines

## Project Structure & Module Organization
Open Lovable is a Next.js 15 app with the main entry under `app/` (routes and server actions). Shared UI pieces sit in `components/`, while smaller primitives wrapping shadcn/Radix live in `atoms/`. Client hooks reside in `hooks/`, and `lib/` hosts data helpers plus sandbox/Firecrawl utilities. Styling is split between `styles/` (global Tailwind layers) and `public/` for static assets. Tooling configs (ESLint, Tailwind, Next) live at the repo root, with reference docs under `docs/`.

## Build, Test, and Development Commands
- `pnpm dev` – run the Turbopack-powered Next dev server on `http://localhost:3000`.
- `pnpm build` – compile for production; run before pushing backend or UI changes.
- `pnpm start` – serve the compiled build locally to mirror Vercel.
- `pnpm lint` – execute ESLint with Next rules; required before opening a PR.
Stick with `pnpm` so dependency resolution matches the checked-in lockfile; use `NODE_ENV=production pnpm build` to reproduce deployment conditions.

## Coding Style & Naming Conventions
Write TypeScript (`.ts`/`.tsx`) wherever possible, preferring async/await over chained promises. Follow the ESLint + Next defaults (no unused vars, use function components, exhaustive deps for hooks). Tailwind classes should stay grouped logically and leverage helpers in `lib/utils`. Components use PascalCase file names, hooks are camelCase prefixed with `use`, and API routes mirror their folders (e.g., `app/api/sandbox/route.ts`). Keep files focused (<200 lines) and colocate utilities next to consumers when practical.

## Testing Guidelines
Automated tests are not yet present; rely on `pnpm lint` plus manual walkthroughs of the AI chat flow (create app, edit component, preview sandbox). If you add tests, prefer Playwright or Vitest suites under `tests/` and name them `feature.spec.ts`. Target coverage for Firecrawl ingestion, sandbox provisioning, and any new server actions.

## Commit & Pull Request Guidelines
Recent history favors short, imperative subject lines (`Add demo images to README`). Avoid emoji or refs in the summary; use the body for context, breaking changes, or migration notes. Every PR should link issues, describe motivation and validation steps, and attach screenshots or screen captures for UI-facing work. Call out any new environment variables or secrets required by the change.

## Security & Configuration Tips
Store secrets in `.env.local` per the README (Firecrawl, OpenAI, Anthropic, Groq, Vercel, E2B, Morph). Never commit credentials; share sample keys via documentation instead. Rotate sandbox tokens before posting public previews, and document new configuration expectations inside `docs/` or the PR description.
