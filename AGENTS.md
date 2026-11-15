# SkillMate — Permanent Agent Rules

## Source of truth
- `docs/SKILLMATE_SPEC.md` is the single source of truth. Never remove, weaken, or silently reinterpret a requirement. If something is impossible or legally blocked, log it in `docs/DECISIONS.md` and ask me.
- `docs/PROGRESS.md` is the project state. Read it at the start of every session, update it after every completed task.

## Phase loop (every phase and sub-phase)
1. Inspect existing code and run the existing test suite.
2. Write the plan into PROGRESS.md.
3. Implement complete functionality: backend + frontend + MongoDB persistence together.
4. Write tests (unit, integration, and e2e for critical flows).
5. Run typecheck, lint, and the FULL test suite, including earlier phases.
6. Verify persistence manually: refresh, logout/login, backend restart.
7. Fix every failure and regression.
8. Update docs and OpenAPI.
9. Write the phase report in PROGRESS.md: Implemented / Tested / Pending / Blockers / How to run and verify.
10. Commit and tag `phase-N-complete`.
Never break earlier modules. Never scaffold the whole platform at once.

## Definition of Done (per feature)
1. Real REST endpoint + Zod validation + Mongoose persistence
2. Frontend wired through TanStack Query, no mock data
3. Vitest/Supertest tests passing
4. Verified: data survives refresh, logout/login, and backend restart
5. OpenAPI docs updated
Do not use the words "done" or "complete" unless all five are true. Be honest about partial work.

## Hard prohibitions
- No mock data, hardcoded responses, localStorage-only persistence, or fake success states in application code.
- No secrets in code or git. Only gitignored `.env` + committed `.env.example`.
- No fake integrations. For payments, email, OAuth, AI, S3: build the adapter + sandbox/test mode and list the needed credentials in `docs/SETUP_REQUIRED.md`.
- Age rules (16+ account, 18+ for paid work/payouts) are enforced server-side in the service layer, never client-only.
- Never claim SkillMate holds or safeguards user funds.

## Mock policy
The database is ALWAYS real (Atlas for staging/prod, Docker replica set locally, mongodb-memory-server in replica-set mode for tests). Test doubles for third-party providers (email, OAuth, payment, AI) are allowed ONLY inside automated tests.

## Pre-decided technical choices (deviate only by logging in docs/DECISIONS.md)
- pnpm workspaces monorepo: apps/web, apps/api, packages/shared (shared Zod schemas + types). Node LTS with `.nvmrc`.
- Each backend module layout: routes, controller, service, repository, model, validation, tests.
- Logging: pino structured logs with request IDs. Sentry for errors.
- i18n: react-i18next, English + Hindi from day 1, no hardcoded UI strings.
- Auth: short-lived access token + refresh token rotation with reuse detection, both in httpOnly Secure cookies. CSRF protection is mandatory. Prefer custom subdomains (app.<domain>, api.<domain>) so SameSite=Lax works; fallback is SameSite=None; Secure. Document which one is used.
- Age: store DOB, compute age server-side, DOB edits after signup need admin review. Guardian consent: pluggable flow, legal question logged in DECISIONS.md.
- Money: integer minor units (paise), never floats. Double-entry ledger, webhook signature verification, idempotency keys on every money-moving operation.
- Search: adapter interface. Atlas Search in staging/prod, MongoDB text-index fallback for local dev and tests.
- Public profiles/landing pages must serve crawlable HTML (meta, OpenGraph, JSON-LD). A plain Vite SPA is not enough, so choose prerender or server-injected meta and log the choice in DECISIONS.md.
- Testing: Vitest + Supertest + mongodb-memory-server (replica set), Playwright for e2e, axe for accessibility.
- CI: GitHub Actions on every PR: typecheck, lint, test, build.
- Seed script creates demo data through the real services. Dev only, never in prod.

## Session rules
- Update PROGRESS.md after every task, not just at phase end.
- If context is getting large, finish the current task, fully update PROGRESS.md, and print "RESUME POINT: <exact next task>".
- When blocked, never fake it: record in `docs/SETUP_REQUIRED.md` or `docs/DECISIONS.md`, continue with unblocked work, and tell me.

