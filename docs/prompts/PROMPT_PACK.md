# SkillMate — Antigravity Prompt Pack

Is file mein SkillMate ko Antigravity se phase-by-phase build karwane ke saare prompts hain.

**Suggested location:** `docs/prompts/PROMPT_PACK.md`

## Contents

0. Setup checklist and how to use this pack
1. `AGENTS.md` (final, goes in repo root)
2. Kickoff prompt (autopilot mode)
3. Phase prompts (controlled mode): 1, 2, 3a, 3b, 4, 5a, 5b, 5c, 5d, 6, 7, 8
4. Utility prompts: Gate Audit, Spec Coverage Audit, Bug Fix, Security Audit, Resume

---

## 0. Setup checklist and how to use this pack

### Repo layout before you start

```text
skillmate/
├── AGENTS.md                      <- paste section 1 here
└── docs/
    ├── SKILLMATE_SPEC.md          <- your full requirements document
    └── prompts/
        └── PROMPT_PACK.md         <- this file
```

### Accounts and credentials you must create yourself

The agent cannot create these. Without a real database the "no mock data" rule cannot be met.

- MongoDB Atlas cluster (separate databases for dev, staging, production)
- Redis (local Docker for dev, Upstash or similar for staging/production)
- Google OAuth client and GitHub OAuth app
- Payment gateway sandbox/test keys (test mode only until legal and provider KYC are clear)
- S3-compatible storage bucket (AWS S3, Cloudflare R2, etc.)
- Email provider account (for OTP and notifications)
- Sentry project
- GitHub repository (CI runs on GitHub Actions)

### Two ways to run

| Mode | What you do | When to use |
|---|---|---|
| **Autopilot** | Paste only the Kickoff prompt (section 2). The agent runs all phases itself and resumes from `docs/PROGRESS.md` when context fills up. | Fast exploration, accepting that you review afterwards. |
| **Controlled** (recommended) | Paste one phase prompt (section 3) per session. Run the Gate Audit (section 4) after each phase. Move on only when it passes. | Anything you want to actually trust, especially Phase 2 and Phase 5c. |

### Rules of thumb

- Start a **new session per phase** so context stays clean. Use the Resume prompt if a phase gets interrupted.
- Run the **Gate Audit** after every phase in a fresh session, so the reviewer is not the author.
- Run the **Security Audit** after Phase 2, Phase 5c, and Phase 8.
- Run the **Spec Coverage Audit** after Phase 3b, Phase 5d, and Phase 8.
- Review `docs/DECISIONS.md` and `docs/SETUP_REQUIRED.md` yourself. The legal items (guardian consent, escrow/marketplace compliance in India, policy pages) need a lawyer and your payment provider, not a prompt.
- Commit and tag at every gate, so you can roll back.

---

## 1. `AGENTS.md` (FINAL)

Save this as `AGENTS.md` in the repo root. Antigravity loads it automatically for every session.

```markdown
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
```

---

## 2. Kickoff prompt (autopilot mode)

```text
You are the principal engineer and tech lead for SkillMate. Read AGENTS.md and docs/SKILLMATE_SPEC.md completely before doing anything.

MISSION
Build SkillMate end to end, following the phases and sub-phases below in order, with a strict verification gate between each.

STEP 0 — INSPECT
Audit the repository: existing structure, reusable parts, missing components, risks. Do not overwrite working code.

STEP 1 — PLANNING DOCS (in /docs)
Generate MASTER_PROMPT.md, TECHNICAL_SPECIFICATION.md, DEVELOPMENT_ROADMAP.md, DATABASE_SCHEMA.md, API_SPECIFICATION.md, SECURITY_AND_PRIVACY.md, TESTING_STRATEGY.md. Also create PROGRESS.md (every spec section as checkable tasks per phase, with status todo / in-progress / implemented / tested / verified), SETUP_REQUIRED.md (every credential or account I must provide), and DECISIONS.md (legal and technical decisions needing my input).

STEP 2 — EXECUTE
Run the phase loop from AGENTS.md for: Phase 1, 2, 3a, 3b, 4, 5a, 5b, 5c, 5d, 6, 7, 8. Scope for each phase is defined by the spec phases, split as: 3a = projects + discovery; 3b = applications, proposals, hiring; 5a = contracts; 5b = milestones + deliverables; 5c = payments + ledger; 5d = disputes and cancellations.
Only when a gate passes, continue to the next phase without waiting for me. If a gate fails, fix until it passes. If a blocker needs my input, log it, work on everything unblocked, and keep going.

STEP 3 — FINAL
After Phase 8, produce docs/PRODUCTION_READINESS.md: what is verified, what is partial, what needs legal/provider/ops review before launch.

Begin with Step 0 and Step 1, then Phase 1.
```

---

## 3. Phase prompts (controlled mode)

Use one per session. Each ends with a STOP so you can review before moving on.

### Phase 1 — Foundation

```text
Execute PHASE 1 per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 3, 4, 20, 24, 25, 26 first.

SCOPE
- pnpm monorepo, TS strict everywhere, ESLint, Prettier, Husky, lint-staged, Vitest, GitHub Actions CI.
- API: Express app factory, versioned /api/v1, helmet, CORS allow-list from env, rate limiting, request IDs, pino logging, central error handler with typed AppError, Zod-validated env config that fails fast, Mongoose connection with retry and graceful shutdown, /health/live and /health/ready (DB + Redis), Swagger UI from OpenAPI, and one example module as the template for all future modules.
- Infra: docker-compose (Mongo single-node replica set, Redis), .env.example, dev/test/seed script skeletons.
- Web: Vite + React + TS, Tailwind, shadcn/ui, Blue + White design tokens, light/dark mode, app shell (sidebar + top navbar, responsive), TanStack Query, Zustand, React Hook Form + Zod, react-i18next (EN/HI), error boundary, loading/empty/error/success components, reduced-motion support, typed API client.

ACCEPTANCE
- From a clean clone, following only the README, `pnpm install && pnpm dev` boots web, api, Mongo, Redis.
- /health/ready reports real DB and Redis state and fails when Mongo is stopped.
- CI is green and the pre-commit hook blocks lint failures.
- App shell has zero critical axe violations and full keyboard navigation.
- One throwaway entity round-trips web → API → MongoDB; remove it afterwards.

Finish with the phase report, tag phase-1-complete, then STOP for my review.
```

### Phase 2 — Auth, age safety, profiles

```text
Execute PHASE 2 per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 2, 5, 6, 19 first. Run the existing test suite before starting.

SCOPE
- Email/password registration and login (Argon2id), email OTP verification (email adapter + BullMQ job, retries), secure password reset, Google and GitHub OAuth.
- Access + refresh token rotation with reuse detection, httpOnly cookies, CSRF, session and device management, logout, login history, RBAC.
- Age eligibility: DOB capture, server-side age calculation, 16+ account rule, 16–17 restricted to non-paid features, 18+ gate on every paid endpoint (add a reusable policy guard so later phases cannot forget it), guardian-consent flow behind a pluggable interface, consent records, no bypass via profile edits.
- Profiles: photo upload through the storage adapter, portfolio, skills (manual entry now; AI later), experience, education, languages, availability, privacy settings, completion percentage, strength indicator, verification badges.
- Guided onboarding, role selection, language/timezone/currency preferences.
- Public profile pages with crawlable HTML per the SEO decision in AGENTS.md.

ACCEPTANCE
- Tests prove: refresh-token reuse revokes the session family; an under-18 user is rejected from a paid endpoint even with a hand-crafted API call; editing DOB does not bypass the gate; RBAC denies cross-user access.
- No plaintext passwords or tokens in DB or logs (verify by inspecting).
- Auth survives refresh, logout/login, and backend restart.
- Every unresolved legal point (guardian consent mechanism, minimum data for minors) is logged in DECISIONS.md.

Finish with the phase report, tag phase-2-complete, then STOP for my review.
```

### Phase 3a — Projects and discovery

```text
Execute PHASE 3a per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 7 and 15 first.

SCOPE
- Project CRUD with a lifecycle state machine: draft, published, paused, closed, cancelled. Illegal transitions must be rejected in the service layer.
- Visibility: public, private, invite-only, custom. Categories, skill requirements, fixed/hourly types, budget (integer minor units), duration, deadlines, experience level.
- Search and filters (category, skills, budget, type, experience, availability) through the search adapter, ranking by skill match, rating, experience, availability, relevance. Saved projects, saved searches, search history with a user-controlled clear option.
- Direct invitations. Rule-based recommendations (no AI yet).
- Required compound indexes; document the query plans for the top 3 queries.

ACCEPTANCE
- Tests cover state transitions and visibility rules (a private project is never visible or searchable by others).
- Pagination is cursor-based and stable under inserts.
- Paid-project creation and participation respect the 18+ guard from Phase 2.
- Data survives refresh, logout/login, and restart.

Finish with the phase report, tag phase-3a-complete, then STOP.
```

### Phase 3b — Applications, proposals, hiring

```text
Execute PHASE 3b per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 7 and 8 (hiring part only) first.

SCOPE
- Application and proposal submission (cover letter, bid, timeline, attachments), review by client, shortlist, reject with reason.
- Negotiation thread and offer management (counter-offers, expiry), offer accept/reject, hire action that creates a contract DRAFT stub for Phase 5a.
- Duplicate application prevention, withdrawal, status history, 18+ guard on paid applications.
- Notifications are not built yet: emit domain events that Phase 6 will consume.

ACCEPTANCE
- Tests cover: one application per freelancer per project (unique index), state machine for proposal/offer, a non-owner cannot review applications, expired offers cannot be accepted.
- Concurrent accept attempts on the same offer result in exactly one hire (use a transaction).

Finish with the phase report, tag phase-3b-complete, then STOP.
```

### Phase 4 — Teams, workspaces, collaboration

```text
Execute PHASE 4 per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 9 and 10 first.

SCOPE
- Organizations and team workspaces, invitations (expiring tokens), custom team sizes, roles (owner, manager, recruiter, finance, member) plus custom permissions via a permission-check layer reused by all modules, member removal, activity and contribution history.
- Project workspaces for client and freelancer: dashboard, tasks (assignees, status, due dates), comments and discussions, file sharing through the storage adapter (type/size validation, malware-scan hook, signed URLs, private by default).
- Real-time chat with Socket.IO + Redis adapter: authenticated sockets, room authorization, message persistence, delivery/read state, reconnection handling, rate limiting.
- Leave milestones for Phase 5b, calendar/meetings/GitHub for Phase 6.

ACCEPTANCE
- Tests prove a user outside a project/org cannot read its messages, files, or tasks, via REST or via sockets.
- Chat works across two server instances (Redis adapter verified).
- Messages survive refresh and restart.
- Playwright e2e: invite member, assign task, exchange chat messages between two browser sessions.

Finish with the phase report, tag phase-4-complete, then STOP.
```

### Phase 5a — Contracts

```text
Execute PHASE 5a per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec section 8 (contracts part) first.

SCOPE
- Contract templates and custom contracts, created from the hire action in Phase 3b.
- Digital acceptance by both parties with a timestamped, IP/device-recorded acceptance record.
- Immutable version history (every edit creates a new version, accepted versions are never mutated), contract-wise revision limits, cancellation request flow with reasons.
- Paid contracts require both parties to pass the 18+ guard at creation and acceptance.

ACCEPTANCE
- Tests prove accepted contract versions cannot be modified, only superseded.
- Amendment flow requires re-acceptance from both sides.
- Add a clear TODO in DECISIONS.md that contract templates need legal review. Do not present generated clauses as legal advice.

Finish with the phase report, tag phase-5a-complete, then STOP.
```

### Phase 5b — Milestones and deliverables

```text
Execute PHASE 5b per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 10 (milestones/progress) and 11 first.

SCOPE
- Milestones per contract (amount in minor units, due date, status), progress tracking.
- Deliverable submission with files and notes, client review, revision requests, configurable revision rounds that respect the contract limit, final approval, submission history, evidence and communication history retained for disputes.
- Milestone completion and project completion workflow driven by state machines.
- Emit domain events for the payments phase (milestone funded, submitted, approved). Do not move money yet.

ACCEPTANCE
- Tests cover: revision-limit enforcement, illegal state transitions, only the right party can perform each action, history is append-only.
- Milestone totals can never exceed the contract amount.

Finish with the phase report, tag phase-5b-complete, then STOP.
```

### Phase 5c — Payments and ledger (HIGHEST RISK: review manually)

```text
Execute PHASE 5c per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec section 12 completely first. Treat this as the highest-risk phase.

SCOPE
- Provider-independent payment adapter interface plus ONE real sandbox/test-mode adapter. No real money.
- Escrow-style lifecycle: initiated → funded → held → released / refunded / partially released, modelled as an explicit state machine, with wording that does not claim SkillMate holds regulated funds.
- Double-entry ledger (ledger_entries always balance, append-only, corrections via reversing entries), transactions, payment_events, integer minor units.
- Webhook endpoint with signature verification, replay protection, and idempotency keys on every money-moving operation.
- Configurable payment/gateway fees disclosed per transaction (no platform commission), multi-currency with INR default.
- Hourly flow: manual time entry, timer, timesheets, client approval, payment from approved timesheets.
- Reconciliation job comparing ledger vs provider events, refund tracking, release and settlement history, client and freelancer payment dashboards.
- Payouts and paid contracts only for 18+ users (reuse the guard).

ACCEPTANCE
- Tests prove: ledger always balances (property-based test), duplicate webhooks do not double-credit, forged webhook signatures are rejected, concurrent release requests produce one release, partial refund math is exact with no float drift.
- Reconciliation detects an injected mismatch.
- Log in DECISIONS.md: provider choice, India marketplace/escrow compliance questions, and the direct-payment fallback model.

Finish with the phase report, tag phase-5c-complete, then STOP.
```

### Phase 5d — Disputes and cancellations

```text
Execute PHASE 5d per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 8 (cancellation), 12 (disputes), and 13 first.

SCOPE
- Cancellation requests with reasons and the counter-party response flow.
- Dispute module: open, evidence submission (files + statements), full communication history, admin assignment, investigation notes, decision (full/partial refund or release) that creates the corresponding ledger entries through the Phase 5c payment service only, never directly.
- Configurable grace periods, completion reminders via BullMQ, auto-release or escalation ONLY if the contract, provider, and law allow it, controlled by a feature flag defaulting to off, with the reason logged in DECISIONS.md.
- Dispute history and audit logs. Minimal admin API endpoints now; the admin UI comes in Phase 7.

ACCEPTANCE
- Tests cover: a decision moves money exactly once, evidence is immutable after submission, non-parties cannot read a dispute, every admin action writes an audit log.
- Scheduled jobs are idempotent and retry safely.

Finish with the phase report, tag phase-5d-complete, then STOP.
```

### Phase 6 — Notifications, AI, search, scheduling, integrations

```text
Execute PHASE 6 per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 10 (calendar, meetings, GitHub), 14, 15, 16 first.

SCOPE
- Notifications: consume the domain events from earlier phases, in-app + email, per-user preferences, BullMQ with retry/backoff/dead-letter handling and an admin-visible failure queue, reminders and scheduled releases.
- AI module behind a provider-independent adapter: skill suggestions, profile tips, project/proposal assistance, recommendations. Require explicit user consent before any personal or project data leaves our system, provide opt-out, minimize data sent, never send private project data for training, label AI output clearly in the UI. Provide a stub/disabled provider so the app works with AI off.
- Search: Atlas Search adapter with the local fallback, better relevance, personalized recommendations.
- Reviews and ratings (only from completed contracts) feeding search ranking.
- Calendar, availability, meeting scheduling with timezone conversion, reschedule/cancel, external video links.
- GitHub integration: repos, branches, commits, PRs, contribution tracking via OAuth app.

ACCEPTANCE
- Tests prove no AI call happens without consent and none happens after opt-out.
- A failed email job retries then lands in the failure queue.
- A review cannot be left without a completed contract.
- Meeting times display correctly across IST and another timezone, including DST edge cases.

Finish with the phase report, tag phase-6-complete, then STOP.
```

### Phase 7 — Admin, moderation, support, analytics

```text
Execute PHASE 7 per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 17, 18, 22 first.

SCOPE
- Secure admin app/area with separate role model and custom admin roles, least privilege, mandatory audit log on every admin action.
- User, project, and organization moderation, reports and complaints, warnings, temporary/permanent suspension, block/unblock, appeals, action history.
- Support tickets: categories, priority, assignment, status tracking, evidence uploads, escalation.
- Dispute and payment monitoring UIs on top of the Phase 5 APIs.
- Analytics: registration funnel, retention, engagement, project success, with privacy controls and aggregation on the server.
- Manual backup trigger and system monitoring views.

ACCEPTANCE
- Tests prove non-admins get 403 on every admin route (enumerate the routes programmatically), suspended users cannot log in or act, an appeal reopens review, and audit logs cannot be edited or deleted via the API.
- Admin UI passes axe checks and keyboard navigation.

Finish with the phase report, tag phase-7-complete, then STOP.
```

### Phase 8 — Hardening, deployment, production verification

```text
Execute PHASE 8 per the phase loop in AGENTS.md. Read docs/PROGRESS.md and spec sections 19, 21, 23, 24, 25 first.

SCOPE
- Security: OWASP Top 10 review of every module, rate limiting per route class, CAPTCHA on signup/login/reset, suspicious-activity and duplicate-account detection, upload hardening, dependency audit, secrets review, security headers, CSRF/XSS/injection tests.
- Privacy and legal: policy pages (privacy, terms, refund/cancellation, escrow/payment, dispute, cookies, acceptable use, agreements) as clearly marked DRAFTS pending legal review, cookie consent, data export, account deactivation/deletion workflow, retention rules implemented as jobs.
- PWA, SEO (sitemap, robots, structured data), performance budgets (Lighthouse), WCAG 2.2 AA audit with axe plus manual keyboard and screen-reader notes.
- Reliability: backup schedule, PITR notes, a real restore test with documented steps, maintenance mode, status page, graceful degradation when Redis or AI is down.
- Deployment: Vercel (web) + Render (api) + Atlas, dev/staging/prod with separate databases and credentials, CI/CD with deploy checks, health checks, rollback procedure, Sentry and uptime monitoring.
- Full Playwright e2e suite for critical flows: signup → hire → fund → deliver → approve → release; dispute path; admin moderation.

ACCEPTANCE
- Staging deployment passes the e2e suite against real Atlas.
- Restore test succeeded and is documented.
- docs/PRODUCTION_READINESS.md lists what is verified, partial, and what still needs legal, provider, and ops review.

Finish with the final report and tag phase-8-complete.
```

---

## 4. Utility prompts

### Gate Audit (after every phase, in a fresh session)

```text
Act as a skeptical senior reviewer, not the author. Read AGENTS.md, docs/PROGRESS.md and docs/SKILLMATE_SPEC.md.
For the most recently completed phase:
1. Re-run typecheck, lint, and the full test suite yourself and paste the real results.
2. Search the codebase for mocks, hardcoded data, TODOs, localStorage persistence, console.log secrets, and skipped tests (.skip, .only).
3. For each acceptance criterion of the phase, show the test or manual check that proves it. Mark any without proof as NOT VERIFIED.
4. Check authorization on every new endpoint: who can call it, and is that enforced server-side?
5. Report the honest status per feature: Verified / Implemented but untested / Partial / Missing. Do not fix anything yet; list issues by severity.
```

### Spec Coverage Audit (after Phase 3b, 5d, and 8)

```text
Build docs/COVERAGE_MATRIX.md: one row per bullet in docs/SKILLMATE_SPEC.md with columns: requirement, phase, status (Verified / Implemented / Partial / Missing / Deferred-with-reason), code location, test location. Be strict: anything without a passing test cannot be Verified. Then list every requirement that was weakened, changed, or silently dropped.
```

### Bug / Regression Fix

```text
Bug: <describe what happens, expected vs actual, steps to reproduce>.
First reproduce it and write a failing test that captures it. Find the root cause (do not patch symptoms). Fix it, make the test pass, run the full suite to confirm no regressions, and add a short entry to docs/LESSONS.md explaining the cause and how to avoid it. Do not touch unrelated code.
```

### Security Audit (after Phase 2, 5c, and 8)

```text
Perform a security review of the whole codebase as an attacker would. Check: authentication and token handling, authorization (IDOR on every :id route), input validation and NoSQL injection, XSS and CSRF, file uploads, webhook verification and replay, rate limits, secrets in git history, dependency vulnerabilities, logging of sensitive data, and age-gate bypasses. For each finding give severity, a reproducible proof, and a fix. Fix critical and high issues with regression tests; list the rest in docs/SECURITY_AND_PRIVACY.md.
```

### Resume (when a session stops or context fills up)

```text
Read AGENTS.md, docs/PROGRESS.md, docs/SETUP_REQUIRED.md and docs/DECISIONS.md. Run the existing test suite first to confirm nothing is broken. Then resume from the exact next unfinished task and continue the phase loop. Report when the next gate passes.
```

---

## Reality check

- Legal items (guardian consent for minors, India escrow/marketplace compliance, policy pages, contract templates) need a lawyer and your payment provider. The agent can only build the technical plumbing.
- The agent can claim things work when they do not. That is why the Gate Audit exists. Do not skip it.
- If you are building this for a job portfolio, depth beats breadth: Phases 1 to 3b, deployed, tested, and documented, impress more than 8 shallow phases.

