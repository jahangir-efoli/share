# AppThrive — Project Status & History

> Moved out of `CLAUDE.md` on 2026-04-22 to keep the always-loaded file lean. Session-specific state, phase changelog, env setup, and reference tables live here. Memory (`~/.claude/projects/-Users-mac-Documents-Claude-Projects-AppThrive/memory/MEMORY.md`) remains the authoritative source for the latest session handoff.

---

## Current stage (2026-04-25 — session close)

**Stage:** v0.5 + REST API 1-3 + Inngest→pg-boss complete + **two real apps (EmbedUp, DiscountRay) syncing live Partner-API data**. **`origin/main` at `def089a`**. No uncommitted work.

### What shipped this session (2026-04-25)

Seven commits. Headline themes: real Partner-API data flowing for two apps, Inngest residue removed for good, Reconnect UX repaired, Merchants page rebuilt for the two-app reality, sync resilient to 429 + unbounded backfills.

| commit | what |
|---|---|
| `d89b1fa` | `merchants_total` counter recomputed from `merchant_subscriptions` (was wiping to 0 on cursor-only syncs) |
| `ba1a280` | CLAUDE / AGENTS / PROJECT-STATUS pointers redirected from `packages/inngest/` → `packages/jobs/handlers/` |
| `c4fbe5a` | Purged 6 remaining dead `inngest.createFunction(...)` registrations + the no-op stub client. ~1,050 LOC removed |
| `41c105a` | Reconnect button on `/apps` now routes to `/apps/[id]/reconnect` — a server route that pre-fills + locks org/app and accepts a fresh Partner API token (overwrites with v2 envelope encryption) |
| `bf30626` | `/merchants` rewritten — per-app stat cards, search by shopName/shopDomain, sortable columns, default sort by `lastActiveAt`, pagination, app filter scoped to the org. Banner now treats any `syncing` row >15min stale as `failed` (orphaned worker recovery) and the `'error'` typo that hid the failed-state variant is fixed |
| `054f758` | Dropped the stale `lint-scoring-concurrency.mjs` custom lint (scanned a deleted dir post-Phase-D); formatted my own files; CI's `Custom lints` job now green |
| `def089a` | Partner-API client retries 429 (respecting Retry-After) + 502/503/504 with exp backoff, in-handler with bounded waits. `performShopifyAppSync` no longer claims success on partial runs. pg-boss handler auto-publishes a follow-up `sync/app.requested` whenever `hitPageLimit=true`, so a 100k-event backfill drains in ~10-15 min wall-clock with zero clicks. 8 new tests |

### What prod looks like today

- **Async backend:** pg-boss only. `JOBS_BACKEND=pgboss`, `ENABLE_PGBOSS=true`, `CRON_SECRET` set in Vercel Production.
- **Real Partner-API data:** **603 merchants** for `org_qVRMh2D86LJd`, all sourced from `partner_api_sync` events. Zero orphans. EmbedUp = 113 subscriptions, DiscountRay = 530 subscriptions (226 active / 304 uninstalled).
- **Schema:** migration 0019 (`apps.partner_org_id`) is the latest. No pending drift this session.
- **Outbound webhooks:** end-to-end signed delivery proven; retry ladder engaged.
- **Public docs:** `/docs/api` renders all 18 REST endpoints + 10 webhook payload schemas.

### Operational fixes applied to prod DB this session

- Cleared 2 orphaned pg-boss `sync/app.requested` jobs for DiscountRay (one stuck 22h, one 24h — `supervise: false` means nothing reaps actives, so the 15-min staleness projection in `/api/apps/sync-status` is the user-facing recovery).
- Reset `apps.last_sync_status='failed'` for DiscountRay with a clear "stalled" message so the banner cleared.
- Deleted **15 seed merchants** with `shopify_shop_id=100001..100015` (Dawn & Sparkle, Oak & Ember, Wildcrest, etc.) from `packages/db/seeds/phase4-dogfood.ts` + `packages/queue-rules/dogfood-data.ts`. Cascade: 1795 score-history + 75 scores + 10 subscriptions + 3 dogfood_seed events.

### Known issues worth addressing next session

- **CI Lint job still fails on `apps/web/__tests__/ingest-batch.test.ts`** — typescript-eslint 8.58 plugin crash in `visitFunctionTypeSignature`. Pre-existing, not new this session. Either bump the plugin past 8.58 or scope-disable `@typescript-eslint/no-unused-vars` on that file (existing pattern elsewhere in the repo).
- **CI Format check still fails on ~200 pre-existing files.** This branch's own files are Prettier-clean per `prettier --check $(git diff origin/main..HEAD)`. A repo-wide `prettier --write` sweep would close it.
- **Compliance test flakes under parallel load** (`__tests__/consent.test.ts > hashes the email`) — race on `PII_HASH_KEY` loading. Passes when run alone. Pre-existing.
- **Webhook.site free tier capped at 50 messages/URL** — failed deliveries in prod are webhook.site rate-limit drops, not real failures. Revoke at `/settings/webhooks` if you want them to stop.

### Next-session candidates

- **(a) Trigger DiscountRay sync end-to-end with the new 429 + auto-continue path.** If clean, the platform has handled its first ~5,000-event Partner-API backfill without manual intervention — that's the v0.4 dogfood-readiness signal we've been waiting for.
- **(b) EFOLI dogfood week-1.** Runbook at `docs/runbooks/efoli-dogfood-week-1.md`. With both apps ingesting real data now, the only blocker was DiscountRay reconnection — done.
- **(c) Stripe live keys swap.** 6-step swap when ready to monetize.
- **(d) Legal counsel review** of 7 `<LegalScaffold>` placeholders.
- **(e) Pre-existing CI cleanup** — Prettier sweep + typescript-eslint plugin bump.
- **(f) Compliance test flake fix** — race on PII_HASH_KEY.
- **(g) v0.6 roadmap discovery** — best done after a real EFOLI dogfood week reveals what's actually missing.

See memory `session_2026_04_25_real_data_flowing.md` for the full session arc.

---

## Shipped through v0.5 Phase 17

- Monorepo scaffolding, 3 Next.js 15 apps: web (3000), admin (3001), marketing (3002)
- Drizzle schema on Neon, 11 migrations applied (0000–0010). No pending migrations.
- **v0.1 Foundation:** Clerk auth (invite-gated), admin panel (HTTP Basic), Partner API sync
- **v0.2 Events + Metrics + Templates + Segments + Campaigns:** BYO-ESP (Resend + SendGrid), 3-path ingest, metric rollups, segment compiler, campaign dispatch
- **v0.2.5 Hardening:** dependency security (closed 10 Dependabot alerts inc. CVE-2026-39356)
- **v0.3 Intelligence (Phases 3–9):** 5-score engine + UI, queue rule engine, pre-send safety guard, queue UI + morning digest, metric thresholds, compliance floor (`@appthrive/audit` + `@appthrive/compliance`, DSR workflows, retention UI, Shopify GDPR webhooks)
- **v0.4 Durable Workflows + Org Thresholds + EFOLI Migration Code (Phases 10–14):**
  - `@appthrive/workflows` compiler + evaluator + `WORKFLOW_FIELD_REGISTRY`
  - Inngest durable executor with `step.sleep`/`sleepUntil`/`waitForEvent`/`invoke` wrappers
  - 3 seeded playbooks (Welcome Series, Dormant Winback, Churn-Risk Alert) auto-installed on org creation
  - `workflow_versions` immutable versioning (migration 0009), multi-successor DAG
  - Org-scope metric thresholds with distinct `metrics/org-threshold.crossed` event + daily 03:15 UTC cron + `basai_ratio` derived metric (migration 0010)
  - `@appthrive/migration` preflight gate + 9-step durable migration workflow + `docs/runbooks/efoli-dogfood-week-1.md`
- **v0.5 Phase 15 Weekly intelligence reports:** Saturday compute cron, hourly per-tz dispatch, `/reports/weekly` config CRUD, public preference center + RFC 8058 one-click unsub, onboarding Step 5
- **v0.5 Phase 16 Stripe billing:**
  - `@appthrive/billing` — `PLAN_LIMITS`, `checkLimit`, `checkFeature`, `getOrgPlan` / `getOrgPlanLimits`, `./plans` + `./gate` browser-safe subpaths
  - Pricing: Free $0 · Pro $29/mo · Scale $149/mo · Internal (admin-only) · 20% annual
  - Stripe adapter: mock-first pattern; live swap is 6 steps (env + `pnpm add stripe` + fill `live-adapter.ts`)
  - `/api/webhooks/stripe` handles subscription upsert/deleted + invoice paid/failed
  - `/settings/billing` checkout + invoice history + cancel confirmation + portal-back toast
  - Plan-gate enforcement on 6 surfaces: merchants (Partner sync), apps (connect), weekly reports, ingest rate-limit (per-plan tier), team seats (Clerk invite), workflows editor
  - Admin plan override at `/accounts/[id]` (4-tile selector, audit-logged)
- **v0.5 Phase 17 Marketing + self-serve signup + legal scaffolds:**
  - `/sign-up/self-serve` unlisted route + webhook `selfServe: true` bypass
  - Real home page (hero + trust strip + problem + approach + pricing teaser + CTA)
  - `/pricing` reading `PLAN_LIMITS` with monthly/annual toggle + 8 FAQ items
  - Shared `MarketingHeader` / `MarketingFooter` / `CTABanner`
  - 7 legal scaffolds (`/privacy /terms /dpa /cookies /subprocessors /aup /security`) with "pending legal counsel review" amber notice
- **Brand:** Founder-provided logo wired across all 3 apps (6 PNG variants per app + Next.js convention `app/icon.png` + `app/apple-icon.png`)
- **Tailwind v4 shadcn token mapping:** `@theme inline` block in `packages/config/tailwind/globals.css`

## Shipped 2026-04-21 (Phases 18–22)

- **Phase 18 — marketing hardening + expansion + legal drafts** (`aff5efe`). Waitlist form bug fixed, `joinWaitlist` hardened, admin notification email, applicant invite emails on approval, cross-domain `/get-started` CTAs via `NEXT_PUBLIC_APP_URL`. New pages `/about /how-it-works /manifesto /changelog`. `/subprocessors` full-published. `/privacy /terms /dpa /cookies /aup` render full drafts under amber "counsel review" banner. New `@appthrive/providers/system-email` helper.
- **Phase 19 — admin OAuth + MFA + class-based dark mode** (`24f6832`). Hard cutover to Auth.js v5 (Google + GitHub) + mandatory TOTP MFA. Migration 0011 extends `admin_users`. Authed routes moved under `app/(authed)/`. Shared Tailwind config flipped to class-based dark (`@custom-variant dark`). HTTP Basic removed; `ADMIN_USERNAME/PASSWORD` obsolete.
- **Phase 20a+20b — workflow editor foundations + executor plumbing** (`0455ae0`). 3 new step kinds (send-slack, send-analytics-event, wait-for-metric) + 1 new trigger (http-webhook) + 1 new registry field (merchant.segmentIds). Migration 0012 adds `workflows.layout`. Migration 0013 adds `workflow_http_webhooks` + `notification` integration type. Public `/api/workflows/webhook/[webhookId]` with HMAC verify.
- **Phase 20c–20g — workflow visual editor** (`f4683c1` / `ea72166` / `dd79ac3` / `95092fb` / `35fce60`). @xyflow/react canvas, 9-kind palette, custom node components, right-side inspector (react-hook-form + Zod per kind), save/publish → `workflow_versions` immutable rows, editable JSON tab, import/export, 50-snapshot undo stack, beautify (autoLayout), `/workflows/[id]/versions/diff` route, round-trip byte-identical test for 3 seeded playbooks.
- **Phase 21a — AI auto-handle foundation** (`8829259`). `@appthrive/ai-autohandle`, migration 0014 (`ai_proposals` + `ai_proposal_state` enum), server actions all RBAC-gated, audit-logged, safety-guarded. "Ask AI" button + dialog in queue UI.
- **Phase 21a.5 — multi-provider** (`413fc4c`). Anthropic + OpenAI pluggable interface, model-id-driven dispatch. Supported: Opus 4.7 / Opus 4.6 / Sonnet 4.6 / Haiku 4.5 / gpt-4o / gpt-4o-mini. Default `claude-haiku-4-5`.
- **Phase 21b.1 — BYO-key AI providers (hard cutover)** (`7fc4daf`). **No platform env fallback.** Per-org `integrations` row (`type='ai'`). `AiProviderNotConfiguredError` + precedence: explicit model → rule-level → org default → first-connected. `/settings/ai-autohandle` page. Queue "Ask AI" auto-hides when `countConnectedAiProviders === 0`.
- **Phase 21b.2 — in-modal proposal preview + inline reject** (`964f9f3`). Template name, `was → now` subject diff, body preamble card, confidence badge, cost bucket, model id. Execute disabled when `templateId === null`. 400-char inline rejection textarea.
- **Phase 21b.3 — plan-gated auto-execute + per-rule overrides** (`2287cd2`). `'ai:auto-execute'` FeatureKey (Free false / Pro-Scale-Internal true). `AiAutohandleOrgSettings.autoExecute` + `autoExecuteDelaySec` + per-rule overrides. `ai.auto-execute-on-proposal` runs `evaluateAutoExecuteGate` → durable `step.sleep(delaySec)` → re-check → execute.
- **Phase 22 — per-org DEK envelope encryption** (`a5761b2`). Migration 0015 adds `per_org_encryption_keys`. `@appthrive/crypto/{kek, kek-vercel, dek}` — `KekProvider` interface (KMS-swap ready), Vercel-secret adapter with multi-version coexistence, 60s LRU DEK cache, insert-race tolerant. v2 envelope format `v2:<keyVersion>:iv:tag:ct`. 28 call sites refactored. `admin_users.totp_secret_encrypted` kept on legacy path. `apps/web/scripts/migrate-to-envelope-encryption.ts` idempotent re-encrypter. `docs/runbooks/encryption-kek-rotation.md`.

## Shipped 2026-04-22 (Inngest→pg-boss Option B)

- **Phase A — scaffold `@appthrive/jobs`** (`f3e8c3c`). pg-boss v10 client, publish wrappers, `runOnce` Upstash SETNX idempotency, drain-loop worker for `/api/cron/jobs-work`. Gated behind `ENABLE_PGBOSS`. Migration 0016 adds `workflow_waits`. 19 unit tests.
- **Phase B1 — dual-publish** (`5947a5f`). `apps/web/lib/emit-job.ts` wraps `inngest.send()` + `jobs.publish()` behind `JOBS_BACKEND` (`inngest` / `pgboss` / `both`). 7 call sites swapped. 24-entry JOB_NAMES registry. 14 tests.
- **Phase B2 — 12 cron handlers** (`820bd19`). Vercel Cron minute-entry for every Inngest cron. Shared `runCron(req, name, handler)` gates on CRON_SECRET + ENABLE_PGBOSS.
- **Phase B3 — 11 simple event handlers** (`2c8ffa9`). Composites for `billing/payment.*`, `subscription.updated`, `app.installed/uninstalled`, `metrics/threshold.crossed`. Bridges 1:1 → 1:N fan-out.
- **Phase B3.1 — 3 complex event handlers** (`d033626`). `metrics/org-threshold.crossed`, `reports/compute-weekly.for-org`, `queue/digest.send`.
- **Phase B3.2 — 2 largest email orchestrators** (`f5c0e62`). `campaign/send.requested` reuses dispatchBatch (300 LOC untouched) + `reports/dispatch.for-merchant` reuses buildDispatchContext + renderReportHtml + ensurePreferences. 8 new subpath exports.
- **Phase C1a — stateless workflow executor + resolveWaits** (`fb53314`). 375 LOC. `workflowsExecuteRun` rewritten from durable-replay to per-step model — each invocation executes ONE step, schedules next via `publishAfter`. `wait-for-event` → insert `workflow_waits` + schedule timeout; incoming events call `resolveWaits` atomically.
- **Phase C3 — AI auto-execute two-phase handler** (`e0305c5`). `AI_PROPOSAL_PERSISTED` split: Phase 1 evaluates gate, publishAfter delay, Phase 2 re-checks + executes. Preserves "don't silently kill a proposal" posture.
- **Phase C4 — DSR workflow handler** (`1e0fcd8`). 505 LOC Inngest refactored to export `runDsrRequest(data, step)` + `InngestStep`. pg-boss port supplies non-memoizing synthetic step. Short-circuits if DSR completed.
- **Phase C5 — EFOLI migration handler + cleanups** (`eddf345`). Same reuse-via-fake-step pattern. 632 LOC migration body extracted to `runMigration(data, step, logger)`. Synthetic `sendEvent` no-ops dead emits. Short-circuits if `apps.lastSyncStatus === 'success'`. Lint overrides extended for typescript-eslint 8.58 crash bug. Removed 4 unnecessary try/catch + 3 dead eslint-disable.

**Port summary:**
- Every cron, event handler, and durable workflow step has a real pg-boss counterpart. 40 registered real handlers, zero stubs.
- 8 Inngest source files had `export` keywords added to inner helpers + 11 new subpath exports. ~1500 LOC reused without duplication. Zero Inngest-side behavior change.
- Dual flag default keeps Inngest authoritative: `ENABLE_PGBOSS=false` + `JOBS_BACKEND=inngest`.

---

## Pending follow-ups (non-blocking)

- **One-shot re-encrypt** — `apps/web/scripts/migrate-to-envelope-encryption.ts` to convert v1 → v2 ciphertext in prod. App operates correctly in mixed state via `decryptMaybeForOrg`. See memory `session_2026_04_21_phase_22_pushed.md` for commands.
- **Smoke tests** — `/settings/ai-autohandle` connect, `/integrations` ESP test, `/queue` Ask-AI, Shopify webhook HMAC verify.
- **B4 soak** (pg-boss) — flip flags in dev/preview ≥48h. Watch Sentry + `jobs` logs.
- **Phase D cutover** — flip `JOBS_BACKEND=pgboss` in prod. After ≥48h stable, delete `@appthrive/inngest` + `/api/inngest/route.ts` + 11 subpath imports. Outbound webhook delivery will move from Inngest function to pg-boss handler (both share `performDelivery` core — flip is cost-free). Helpers relocate to `@appthrive/campaigns`, `@appthrive/reports`, `@appthrive/scoring`, etc. — see memory for plan.
- **Outbound webhooks post-push verification** — create a test subscription at `/settings/webhooks` pointing at `https://webhook.site`, trigger a sync on an EFOLI app, confirm `merchant.created` + `installation.created` deliveries show up in the subscription's delivery log.
- **Secondary contacts (Shopify staff fetch)** — deferred; full phase-sized work per `docs/runbooks/secondary-contacts-deferred.md`. Needs merchant-side OAuth install, not a sprint item.
- **REST API SDK npm publish** — `@appthrive/api-client` is a private workspace package today. Publish to npm once external testers validate the shape.

---

## Verification at 2026-04-22 session close

- 22/22 monorepo typecheck green
- 0 lint errors (2 pre-existing ESP warnings, documented quirk)
- Tests: web 99/99 (+4 skipped) · inngest 159/159 · jobs 19/19 · crypto 52/52 · billing 50/50 · ai-autohandle 21/21 · workflows 49/49 · providers 26/26

## Verification at 2026-04-23 session close (REST API Phases 2-3 + Option C)

- 25/25 monorepo typecheck green (+2 new packages: api-client, outbound-webhooks)
- 24/24 monorepo test green
- Tests: web 151/151 (+4 skipped) · inngest 195/195 · outbound-webhooks 16/16 · billing 50/50 · (rest unchanged)
- Drift check: `pnpm --filter @appthrive/web check:openapi` in sync — CI `openapi-drift` job enforces it on every PR
- Migrations: 0017 (PAT, already in prod) + 0018 (outbound webhooks, applies via `vercel-build` on next deploy)
- New Vercel cron: none — outbound webhook delivery fires via Inngest events, not cron
- New env vars: none — Phase 2-3 uses existing Clerk + per-org DEK + Upstash + DB infrastructure
- `apps/web` production build succeeds (`next build` + `pnpm vercel-build`)

---

## Env setup for local dev

- **`apps/web/.env.local`** — Clerk keys, Neon `DATABASE_URL`, Inngest, `ENCRYPTION_KEY` (legacy, kept for admin TOTP + pre-22 decrypt), `ENCRYPTION_KEK_V1` (base64 32-byte KEK for per-org DEKs), Upstash Redis.
  - **Added 2026-04-22:** `ENABLE_PGBOSS` + `JOBS_BACKEND` + `CRON_SECRET` — all optional; defaults keep Inngest authoritative.
  - `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` are **no longer read at runtime** (Phase 21b.1 hard cutover). AI is BYO-key, per-org via `/settings/ai-autohandle` or dev seeder `apps/web/scripts/seed-ai-provider.ts` (`ORG_ID=… ANTHROPIC_API_KEY=… pnpm --filter @appthrive/web tsx scripts/seed-ai-provider.ts`).
- **`apps/admin/.env.local`** — **OAuth required** (post-Phase 19): `AUTH_SECRET`, `NEXTAUTH_URL=http://localhost:3001`, `ADMIN_GOOGLE_CLIENT_ID/SECRET`, `ADMIN_GITHUB_CLIENT_ID/SECRET`, `ADMIN_BOOTSTRAP_EMAIL=razu@efoli.com`. `ADMIN_ALLOWED_DOMAINS` optional (defaults `efoli.com,appthrive.io`). First OAuth login from bootstrap email auto-creates super_admin + forces TOTP. `ADMIN_USERNAME/PASSWORD` now ignored.
- **`apps/marketing/.env.local`** — `RESEND_API_KEY` + `PLATFORM_FROM_EMAIL` + `UPSTASH_REDIS_REST_URL/TOKEN` + `NEXT_PUBLIC_APP_URL` + `DATABASE_URL` for waitlist.

---

## Known quirks

- `apps/web/app/(app)/segments/segment-composer.tsx` + several other tsx files have `@typescript-eslint/no-unused-vars` disabled in scoped ESLint overrides. typescript-eslint 8.58 crashes on function-type-signature param visitation; revisit when plugin bumps past 8.58.
- `apps/web/app/api/integrations/esp/{connect,test-connection}/route.ts` carries pre-existing unused `no-console` eslint-disable directives.
- `apps/marketing/next.config.ts` warning about `experimental.typedRoutes` → `typedRoutes` move.

---

## Planned tech stack

| Layer           | Choice                                                | Notes                                                      |
| --------------- | ----------------------------------------------------- | ---------------------------------------------------------- |
| Framework       | Next.js 15 (App Router)                               | RSC + API routes + middleware                              |
| Language        | TypeScript 5.4+                                       | Strict mode everywhere                                     |
| Runtime         | Node.js 20 LTS                                        | Vercel serverless                                          |
| Monorepo        | Turborepo + pnpm workspaces                           | 3 apps, 11+ packages                                       |
| Database        | Neon Postgres                                         | Serverless, auto-branching for PRs                         |
| ORM             | Drizzle                                               | Type-safe, thin, SQL-close                                 |
| Auth            | Clerk                                                 | Multi-tenant orgs, MFA, impersonation                      |
| Jobs            | Inngest (pg-boss dormant)                             | Durable execution; pg-boss port shipped behind flags       |
| Cache           | Upstash Redis                                         | Rate limiting, session cache, segment cache                |
| Billing         | Stripe                                                | Not Shopify Billing (we sell to developers, not merchants) |
| UI              | shadcn/ui + Tremor v4 blocks                          | Radix, Tailwind 4 (OKLCH)                                  |
| Typography      | Geist + Inter                                         | Vercel/Linear aesthetic                                    |
| Forms           | React Hook Form + Zod                                 |                                                            |
| Tables          | TanStack Table v8                                     |                                                            |
| Charts          | Recharts v3                                           | Via shadcn chart components                                |
| Email rendering | React Email + MJML                                    | Customer-facing + weekly reports                           |
| Observability   | Sentry + Axiom + Vercel Analytics + Inngest Dashboard |                                                            |
| CI/CD           | GitHub Actions + Vercel                               | Preview per PR, auto-deploy on main                        |

---

## Planned repo structure

```
apps/
├── web/           # app.appthrive.io (customer-facing)
├── admin/         # admin.appthrive.io (internal)
└── marketing/     # appthrive.io

packages/
├── db/             # @appthrive/db — Drizzle schema + client
├── auth/           # @appthrive/auth — Clerk wrappers, roles
├── jobs/           # @appthrive/jobs — pg-boss client + handlers (sole prod async backend)
├── jobs-helpers/   # @appthrive/jobs-helpers — framework-free helpers shared by handlers
├── providers/      # @appthrive/providers — ESP adapters
├── scoring/        # @appthrive/scoring — 5-score computation
├── queue-rules/    # @appthrive/queue-rules — Action Queue rule engine
├── metrics/        # @appthrive/metrics — merchant metrics pipeline
├── sdk/            # @appthrive/sdk — client SDK for developers
├── email-renderer/ # @appthrive/email-renderer — React Email + MJML
├── crypto/         # @appthrive/crypto — encryption, HMAC, hashing
├── billing/        # @appthrive/billing — PLAN_LIMITS, gates
├── audit/          # @appthrive/audit
├── compliance/     # @appthrive/compliance — DSR, retention
├── workflows/      # @appthrive/workflows — compiler + evaluator
├── ai-autohandle/  # @appthrive/ai-autohandle
├── migration/      # @appthrive/migration — EFOLI preflight + migration
├── ui/             # @appthrive/ui — shared components
└── config/         # @appthrive/config — ESLint, TS, Tailwind presets
```

---

## Founder decisions already locked

| # | Decision |
| --- | --- |
| Post-uninstall emails | Default 2 (exit survey + 30-day winback); flexible — developer can add more with consent |
| EFOLI dogfooding pricing | **"Internal" plan** — full features, $0. Shareable with trusted advisors. |
| Multi-currency | **All revenue normalized to USD.** Store original currency as reference only. |
| Unsubscribe page | **Both** — preference center (granular) + ESP one-click (compliance) |
| Invite-only signup limit | **50 per week.** Configurable via admin panel. |
| Invite code benefits | **Permanent benefits supported** — grandfather pricing, discounts, extended trials. |

Implementation details: `docs/AppThrive-Sprint-Plan.md` → "Founder Decisions".

---

## Docs added since initial scaffolding

- `AppThrive-Sprint-1-Retro.md` — Sprint 1 outcomes + decisions to revisit
- `AppThrive-Sprint-2-Plan.md` — Sprint 2 plan with locked Q1–Q7 decisions
- `AppThrive-Sprint-2-External-Services-Setup.md` — Upstash + Resend + SendGrid walkthrough

---

## Resume-from-compact playbook

When a session is compacted, memory persists at `~/.claude/projects/-Users-mac-Documents-Claude-Projects-AppThrive/memory/`. To resume:

1. Read `MEMORY.md` top entry (`session_2026_04_22_option_b_complete.md`) for the close-out.
2. Run `cd /Users/mac/Documents/Claude/Projects/AppThrive && git status && git log --oneline origin/main..HEAD` — expect clean working tree, 0 commits ahead.
3. Three valid next moves: (a) B4 soak, (b) Phase D cutover, (c) unrelated next work. PRD Q4 RESOLVED flip still pending a docs sweep (see `project_deferred_prd_q4_update.md`).
4. Invariants to re-verify before B4/D: `ENABLE_PGBOSS` + `JOBS_BACKEND` values per Vercel env; migration 0016 applied (workflow_waits exists); `/api/cron/*` returns `{status:'disabled'}` when flag off.
5. Claude respects "develop locally, push when asked" (`feedback_local_first_dev.md`). Authorization is per-request.

---

_Last updated: 2026-04-22 session close. Update when major architecture changes or docs are added._
