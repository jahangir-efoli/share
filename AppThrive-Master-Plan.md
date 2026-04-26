# AppThrive Master Plan — From v0.5 to Production Launch

> **Created:** 2026-04-25 | **Based on:** Deep codebase analysis (94K LOC, 22 packages, 65 DB tables, 55 API endpoints, 47 UI pages) + PRD v1.3 + ChatGPT milestone doc gap analysis
> **Purpose:** Agent-assignable sprint plan to close all gaps and ship a complete, scalable, AI-ready product

---

## Executive Summary

**AppThrive is 75% complete.** The core platform (multi-tenancy, Partner API sync, 5-score engine, action queue, workflows, email dispatch, billing, compliance, REST API, SDK) is built and serving 603 real merchants in production. The remaining 25% falls into four categories:

1. **Feature gaps** (M2 merchant 360, M4 additional ESPs, M13 dev config layer)
2. **Scalability** (DB partitioning, caching, backpressure, per-endpoint rate limits)
3. **AI-native readiness** (MCP server, vector search, data export pipelines)
4. **Launch readiness** (legal review, admin panel completion, Stripe live keys, dogfood validation)

This document provides **23 sprint tasks** organized into **6 sprints** (3 weeks each), each decomposed into agent-assignable subtasks with exact file paths, data sources, and acceptance criteria.

---

## Part 1: Current State Assessment

### What's Built & Working (Production)

| Area | Status | Evidence |
|------|--------|----------|
| Multi-tenant auth (Clerk) | ✅ Live | orgId on all queries, invite-gated |
| Partner API sync | ✅ Live | 603 merchants (EmbedUp 113, DiscountRay 530) |
| Per-org encryption | ✅ Live | AES-256-GCM DEK envelope, 28 call sites |
| 5-score engine | ✅ Live | health, churn, activation, expansion, deliverability |
| Action Queue | ✅ Live | 15 default rules + custom + morning digest |
| Workflow editor | ✅ Live | @xyflow/react, 9 step kinds, undo/redo, versioning |
| BYO-sender email | ✅ Live | Resend + SendGrid adapters |
| Weekly reports | ✅ Live | Saturday compute, per-TZ dispatch, preference center |
| AI auto-handle | ✅ Live | BYO-key (Anthropic + OpenAI), auto-execute on Pro+ |
| REST API v1 | ✅ Live | 18 endpoints, PAT auth, OpenAPI drift-check in CI |
| Developer SDK | ✅ Live | track(), trackBatch(), incrementMetric(), setMetric() |
| Outbound webhooks | ✅ Live | 10 event types, signed, retry ladder, dead-letter |
| Compliance (GDPR) | ✅ Live | DSR, crypto-erasure, retention, audit logs |
| Billing (Stripe) | ✅ Mock | Free/Pro/Scale/Internal plans, 6 plan-gated surfaces |
| pg-boss async | ✅ Live | 40 handlers (cron, events, workflows, DSR, migration) |
| Marketing site | ✅ Live | 14 pages, waitlist, approval flow |
| Admin panel | ✅ Live | OAuth+MFA, account management, plan override |

### Critical Gaps (Ordered by Impact)

| # | Gap | Milestone | Impact | Effort |
|---|-----|-----------|--------|--------|
| 1 | Merchant 360 page incomplete | M2 | High — core UX | 3-5 days |
| 2 | Event timeline UI missing | M3 | High — core UX | 2-3 days |
| 3 | DB partitioning for scale | M14 | High — blocks >10K merchants | 2-3 days |
| 4 | Stripe live keys | M12 | High — blocks revenue | 1 day |
| 5 | EFOLI dogfood validation | M16 | High — blocks launch | 1 week |
| 6 | Additional ESP adapters | M4 | Medium — customer demand | 2-3 days |
| 7 | Merchant tags (first-class) | M2 | Medium — segmentation depth | 2 days |
| 8 | Per-endpoint rate limits | M14 | Medium — API abuse prevention | 1-2 days |
| 9 | Response caching | M14 | Medium — performance | 1-2 days |
| 10 | MCP server | M15 | Medium — differentiation | 3-5 days |
| 11 | Dev config layer (no-code) | M13 | Medium — developer experience | 5-8 days |
| 12 | Usage dashboard | M12 | Low — nice-to-have for billing | 1-2 days |
| 13 | Admin panel completion | M16 | Low — internal tooling | 3-5 days |
| 14 | Legal counsel review | M16 | Low (non-code) — blocks public launch | External |
| 15 | Vector embeddings | M15 | Low — future AI features | 3-5 days |
| 16 | Additional workflow playbooks | M6 | Low — value-add | 2-3 days |

---

## Part 2: Data Source Architecture

### Current Data Flow

```
┌─────────────────────────────────────────────────────┐
│                   DATA SOURCES                       │
├──────────────┬──────────────┬───────────────────────┤
│ Shopify      │ Developer    │ Platform-Generated    │
│ Partner API  │ Apps         │                       │
├──────────────┼──────────────┼───────────────────────┤
│ Events via   │ SDK track()  │ Score computation     │
│ cursor-based │ SDK metric() │ Queue generation      │
│ sync (pg-    │ Webhook POST │ Report computation    │
│ boss cron)   │ to /ingest/  │ Threshold evaluation  │
│              │ {org}/{app}  │ Workflow execution    │
│ Shopify GDPR │              │                       │
│ webhooks     │ Shopify-     │ AI proposals          │
│ (mandatory)  │ direct       │ Campaign sends        │
│              │ webhooks     │ ESP bounce/delivery   │
└──────┬───────┴──────┬───────┴───────────┬───────────┘
       │              │                   │
       ▼              ▼                   ▼
┌─────────────────────────────────────────────────────┐
│              INGESTION PIPELINE                      │
│  Zod validation → HMAC verify → idempotency check   │
│  → merchant resolution → event insert → metric      │
│  rollup → threshold eval → workflow trigger          │
└─────────────────────────┬───────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│              NEON POSTGRES (65 tables)                │
│  merchants, events, metrics, scores, queue,          │
│  workflows, campaigns, compliance, billing           │
└─────────────────────────────────────────────────────┘
```

### Data Sources to Add (Future-Proofing)

| Source | Integration Method | Priority | Notes |
|--------|-------------------|----------|-------|
| Shopify Admin API (per-merchant) | OAuth install flow | V1+ | Requires merchant-side app install; enables shop details, orders, customers |
| ESP engagement data (opens/clicks) | Webhook from SendGrid/Resend | Sprint 2 | Already have ESP webhook endpoints; need to map to merchant metrics |
| Billing platform data | Stripe webhook enrichment | Sprint 1 | Already receiving; need to map invoice data to merchant revenue metrics |
| Third-party analytics | Ingest API (`POST /ingest/{org}/{app}`) | Already works | Developers can send PostHog/Mixpanel data via SDK |
| CSV bulk import | New `/api/v1/import` endpoint | Sprint 3 | For merchants migrating from spreadsheets |
| Custom webhooks | `POST /api/v1/webhooks/inbound` | Sprint 4 | Generic webhook receiver with configurable payload mapping |

### Data Source Adapter Interface (To Build)

```typescript
// packages/providers/data-source-adapter.ts
export interface DataSourceAdapter {
  readonly name: string;
  readonly version: string;
  
  /** Can this adapter handle the given event shape? */
  canHandle(event: unknown): boolean;
  
  /** Normalize to AppThrive's canonical event format */
  normalize(event: unknown): NormalizedEvent;
  
  /** Map normalized event to metric updates */
  mapToMetrics(event: NormalizedEvent): MetricUpdate[];
  
  /** Validate adapter-specific configuration */
  validateConfig(config: unknown): AdapterConfig;
}
```

---

## Part 3: Sprint Plan

### Sprint 1 — Foundation Hardening (Days 1-15)
**Theme:** Close critical gaps, prepare for EFOLI dogfood

#### Task 1.1: Merchant 360 Detail Page
**Priority:** P0 | **Effort:** 4 days | **Milestone:** M2

**What exists:**
- `apps/web/app/(app)/merchants/[id]/page.tsx` — shows scores + subscriptions
- `packages/db/schema/merchant-extended.ts` — `shop_notes` table exists
- `packages/db/schema/merchants.ts` — `customAttributes` JSONB column

**What to build:**
- [ ] **Timeline tab** — vertical event stream for the merchant across all apps
  - Source: `events` table filtered by `merchantId`, ordered by `occurredAt DESC`
  - Show: event type icon, timestamp, app badge, key properties
  - Pagination: cursor-based, 50 per page
  - File: `apps/web/app/(app)/merchants/[id]/timeline.tsx`
  
- [ ] **Messages tab** — all emails/comms sent to this merchant
  - Source: `messages` table + `campaign_sends` filtered by merchantId
  - Show: subject, template, ESP used, delivery status, opens/clicks
  - File: `apps/web/app/(app)/merchants/[id]/messages.tsx`

- [ ] **Notes tab** — private notes (already in DB, no UI)
  - Source: `shop_notes` table
  - CRUD: add/edit/delete notes with author attribution
  - File: `apps/web/app/(app)/merchants/[id]/notes.tsx`

- [ ] **Compliance tab** — DSR status, consent, data retention
  - Source: `dsr_requests` + `consent_records` filtered by merchantId
  - Show: active DSRs, consent status, retention schedule
  - File: `apps/web/app/(app)/merchants/[id]/compliance.tsx`

- [ ] **"Send message" dropdown** on merchant detail header
  - Options: "Send via [connected ESP]", "Add note (no send)"
  - Opens composer with template selector + variable insertion + preview

**Acceptance criteria:**
- All 5 tabs render with real data from EFOLI's 603 merchants
- Timeline shows Partner API sync events correctly
- Notes persist across sessions

#### Task 1.2: Merchant Tags UI & Wiring
**Priority:** P1 | **Effort:** 2 days | **Milestone:** M2

**What exists:** `merchant_tags` table already in `packages/db/schema/merchants.ts` with indexes. No UI or API endpoints yet.

**What to build:**
- [ ] Tag CRUD API: `POST/GET/DELETE /api/v1/tags`
- [ ] Tag assignment: `POST /api/v1/merchants/{id}/tags`
- [ ] Bulk tag operations: `POST /api/v1/merchants/bulk-tag`
- [ ] Tag UI component (color-coded pills, autocomplete)
- [ ] Tag filter in merchant list sidebar
- [ ] Segment condition: `merchant.tags contains [tagId]`

**Data source:** User-created tags + bulk operations
**Acceptance:** Tags persist, appear on merchant cards, filter correctly in segments

#### Task 1.3: Stripe Live Keys Swap
**Priority:** P0 | **Effort:** 0.5 days | **Milestone:** M12

**What exists:** `packages/billing/adapters/mock-adapter.ts` + `live-adapter.ts` skeleton

**Steps (documented in PROJECT-STATUS.md):**
1. `pnpm add stripe` in `packages/billing`
2. Set `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET` in Vercel
3. Create products/prices in Stripe Dashboard matching `PLAN_LIMITS`
4. Fill `live-adapter.ts` with real Stripe SDK calls
5. Set `BILLING_ADAPTER=stripe` in Vercel production
6. Verify webhook endpoint at `/api/webhooks/stripe`

**Acceptance:** Real checkout flow works, invoice history populates, plan changes reflected

#### Task 1.4: Database Partitioning Strategy
**Priority:** P0 | **Effort:** 2 days | **Milestone:** M14

**Tables needing partitioning (grow unbounded):**
- `events` — currently all events in one table; at 10K events/min = 430M/year
- `merchant_metrics` — rollups accumulate per metric × period × merchant
- `outbound_webhook_deliveries` — delivery logs per webhook subscription
- `score_history` — daily snapshots per merchant × score type

**Implementation:**
- [ ] Create partition migration (0020 or 0021) using Postgres native RANGE partitioning on `created_at` (monthly)
- [ ] Add partition management cron: auto-create next month's partition, detach partitions older than retention policy
- [ ] Document Neon PgBouncer configuration in `docs/runbooks/connection-pooling.md`
- [ ] Add `EXPLAIN ANALYZE` checks to top 10 queries in CI (optional but recommended)

**Approach:** Use Postgres declarative partitioning (no app-level sharding). Neon supports it natively.

```sql
-- Example: partition events table
CREATE TABLE events_new (LIKE events INCLUDING ALL) PARTITION BY RANGE (created_at);
CREATE TABLE events_2026_04 PARTITION OF events_new FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');
-- Migrate data, swap table names, update Drizzle schema
```

#### Task 1.5: Per-Endpoint Rate Limits
**Priority:** P1 | **Effort:** 1 day | **Milestone:** M14

**What exists:** `apps/web/lib/ingest-auth.ts` — per-org sliding window on Upstash Redis

**What to build:**
- [ ] Rate limit config per endpoint category:
  - `/api/v1/events` (ingest): keep existing per-plan tier
  - `/api/v1/merchants` (read): 300/min Free, 3000/min Pro, 30000/min Scale
  - `/api/v1/queue` (actions): 60/min Free, 600/min Pro, 6000/min Scale
  - `/api/v1/auth/tokens` (sensitive): 5/min per user (prevent enumeration)
  - `/api/v1/webhooks` (config): 30/min per org
- [ ] Implement in `apps/web/middleware.ts` with endpoint pattern matching
- [ ] Return proper `429` + `Retry-After` + `X-RateLimit-*` headers

---

### Sprint 2 — Communication & Observability (Days 16-30)
**Theme:** Complete messaging infrastructure, add observability

#### Task 2.1: Additional ESP Adapters
**Priority:** P1 | **Effort:** 3 days | **Milestone:** M4

**What exists:**
- `packages/providers/email/resend.ts` — working adapter
- `packages/providers/email/sendgrid.ts` — working adapter
- `packages/providers/email/types.ts` — `EmailProvider` interface

**What to build:**
- [ ] **Postmark adapter** (`packages/providers/email/postmark.ts`)
  - Use `postmark` npm package (free, no platform cost)
  - Implement: `sendEmail`, `sendBatch`, `verifyDomain`, `getDeliveryStatus`
  - Map Postmark webhooks to `NormalizedEmailEvent`

- [ ] **Amazon SES adapter** (`packages/providers/email/ses.ts`)
  - Use `@aws-sdk/client-ses` (pay-per-use, most cost-effective at scale)
  - Implement same interface
  - SNS webhook for bounce/complaint notifications

- [ ] **Mailgun adapter** (`packages/providers/email/mailgun.ts`)
  - Use `mailgun.js` npm package
  - Same interface implementation

- [ ] Update `createEmailProvider()` factory in `packages/providers/email/index.ts`
- [ ] Add integration tests for each adapter

**Cost note:** All adapters use developer's own ESP account (BYO-sender). No platform cost.

#### Task 2.2: ESP Engagement Metrics Pipeline
**Priority:** P1 | **Effort:** 2 days | **Milestone:** M4

**What exists:**
- `apps/web/app/api/webhooks/esp/[provider]/[integrationId]/route.ts` — receives ESP webhooks
- `NormalizedEmailEvent` type handles bounce/complaint/unsubscribed

**What to build:**
- [ ] Extend `NormalizedEmailEvent` to include `opened`, `clicked`, `link_clicked` events
- [ ] Map ESP engagement webhooks to merchant metrics:
  - `email_open_rate` = opens / sends (per merchant per period)
  - `email_click_rate` = clicks / opens
  - `email_engagement_score` = weighted composite
- [ ] Feed engagement data into **deliverability score** computation
- [ ] Add engagement sparkline to campaign detail page

#### Task 2.3: Response Caching Layer
**Priority:** P1 | **Effort:** 1.5 days | **Milestone:** M14

**What to build:**
- [ ] Redis cache wrapper in `apps/web/lib/cache.ts`
  - Use existing Upstash Redis connection
  - Pattern: `cache.get(key, ttlSeconds, fetchFn)`
  - Cache invalidation on write operations

- [ ] Cache these endpoints (read-heavy, rarely change):
  - `GET /api/v1/apps` — TTL 5 min
  - `GET /api/v1/merchants` (list) — TTL 2 min, invalidate on sync
  - `GET /api/v1/segments` — TTL 5 min
  - `GET /api/v1/workflows` — TTL 5 min
  - Dashboard aggregates — TTL 10 min

- [ ] Add `Cache-Control` headers to API responses
- [ ] Cache key scoped by `orgId` to prevent cross-tenant leakage

#### Task 2.4: Ingest Backpressure & Timeout
**Priority:** P1 | **Effort:** 1 day | **Milestone:** M14

**What to build:**
- [ ] Add request timeout: 30s max for ingest endpoints (`apps/web/app/api/ingest/`)
- [ ] Queue depth check before accepting events:
  - Check pg-boss queue depth for `sync/app.requested`
  - If depth > 1000, return `503 Service Unavailable` with `Retry-After: 30`
- [ ] Add gzip compression support in Next.js config
- [ ] Monitor: emit `ingest.backpressure.shed` metric when 503 returned

#### Task 2.5: Event Timeline UI
**Priority:** P1 | **Effort:** 2 days | **Milestone:** M3

**What to build:**
- [ ] `EventTimeline` component (`packages/ui/src/event-timeline.tsx`)
  - Vertical timeline with event type icons
  - Grouped by date
  - Expandable event detail (full payload view)
  - Filter by event type, app, date range

- [ ] Server action: `getEventTimeline(merchantId, cursor, filters)`
  - Query: `events` table WHERE `merchantId = ?` AND `orgId = ?`
  - Order: `occurredAt DESC`
  - Pagination: cursor-based, 50 per page

- [ ] Wire into merchant detail page Timeline tab (Task 1.1)
- [ ] Also show mini-timeline (last 5 events) on queue item "Act" modal

---

### Sprint 3 — Developer Experience & Config (Days 31-45)
**Theme:** Empower developers to self-configure

#### Task 3.1: Custom Field Schema Registry
**Priority:** P2 | **Effort:** 3 days | **Milestone:** M13

**What exists:**
- `merchants.customAttributes` JSONB column (unstructured)

**What to build:**
- [ ] Migration: `custom_field_definitions` table
  ```
  id, orgId, name, displayName, fieldType (text|number|date|dropdown|multi_select|boolean),
  options (jsonb, for dropdown/multi_select), required, defaultValue, createdAt
  ```
- [ ] API: `GET/POST/PATCH/DELETE /api/v1/custom-fields`
- [ ] Validation: when writing `customAttributes`, validate against field definitions
- [ ] UI: `/settings/custom-fields` — CRUD interface for field definitions
- [ ] UI: Custom field editor on merchant detail page (dynamic form from definitions)
- [ ] Segment integration: `customFields.{fieldName}` as segment condition

#### Task 3.2: Bulk Merchant Operations
**Priority:** P1 | **Effort:** 2 days | **Milestone:** M2

**PRD reference:** MR-005, MR-006, MR-007

**What to build:**
- [ ] Checkbox selection on merchant list (individual + select all matching filter)
- [ ] Bulk action toolbar: "Tag", "Remove tag", "Add to segment", "Export CSV", "Send campaign"
- [ ] `POST /api/v1/merchants/bulk` — accepts action + merchantIds array
- [ ] CSV export: `GET /api/v1/merchants/export?format=csv` — honors current filters
- [ ] Saved views: `saved_views` table (orgId, name, filters JSON, isShared, createdBy)
- [ ] Saved view UI: save current filter state, load saved views, share URL

#### Task 3.3: No-Code Event Type Registration
**Priority:** P2 | **Effort:** 2 days | **Milestone:** M13

**What to build:**
- [ ] Migration: `event_type_definitions` table
  ```
  id, orgId, appId, eventType, displayName, description, category,
  propertySchema (jsonb — defines expected properties), createdAt
  ```
- [ ] UI: `/apps/[id]/events` — register event types with property schema editor
- [ ] Validation: when ingesting events, optionally validate properties against schema
- [ ] Auto-discovery: suggest event type definitions from observed events (scan `events` table for distinct `eventType` values)

#### Task 3.4: Template WYSIWYG Editor Foundation
**Priority:** P2 | **Effort:** 3 days | **Milestone:** M13

**What exists:**
- `packages/email-renderer/` — Handlebars template interpolation
- Template CRUD at `/templates`

**What to build:**
- [ ] Integrate `@tiptap/react` (free, MIT licensed) for WYSIWYG email editing
  - Rich text: bold, italic, links, images, dividers
  - Variable insertion: `{{merchant.name}}`, `{{metrics.revenue}}` etc.
  - Preview: toggle between editor ↔ rendered HTML
- [ ] Template block system: header, content, CTA, footer (drag to reorder)
- [ ] Save templates as MJML (already have MJML rendering pipeline)
- [ ] Template gallery: 5 pre-built templates (welcome, milestone, report, survey, winback)

**Cost note:** Tiptap core is free/MIT. No paid service needed.

---

### Sprint 4 — AI-Native & Future-Proofing (Days 46-60)
**Theme:** Make AppThrive AI-ready and extensible

#### Task 4.1: MCP Server
**Priority:** P1 | **Effort:** 4 days | **Milestone:** M15

**PRD reference:** §1.4 "AI-native from day one — every action exposable via MCP for Claude/ChatGPT control"

**What to build:**
- [ ] New package: `packages/mcp-server/`
  - Implement MCP protocol (Model Context Protocol)
  - Expose tools for: list merchants, get merchant detail, list queue items, act on queue item, send message, create segment, get scores, list workflows, trigger workflow
  - Auth: PAT-based (reuse existing REST API auth)

- [ ] MCP resources:
  - `merchant://` — merchant profiles with scores
  - `queue://` — current action queue items
  - `campaign://` — campaign history
  - `workflow://` — workflow definitions

- [ ] MCP tools (map to existing server actions):
  - `get_merchants(filters)` → calls existing merchant list logic
  - `get_merchant_detail(id)` → calls existing detail logic
  - `act_on_queue_item(id, action)` → calls existing queue action logic
  - `send_message(merchantId, templateId, esp)` → calls existing send logic
  - `create_segment(conditions)` → calls existing segment logic
  - `get_dashboard_summary()` → aggregates scores + queue + metrics

- [ ] Documentation page: `/docs/mcp` — setup instructions for Claude Desktop, Cursor, etc.

**Cost note:** MCP is an open protocol. No external service needed. Uses existing AppThrive auth + API.

#### Task 4.2: Vector Embeddings Foundation (pgvector)
**Priority:** P2 | **Effort:** 3 days | **Milestone:** M15

**What to build:**
- [ ] Enable `pgvector` extension on Neon (free, built-in)
- [ ] Migration: `merchant_embeddings` table
  ```
  id, orgId, merchantId, embeddingType ('profile'|'activity'|'communication'),
  embedding vector(1536), metadata jsonb, updatedAt
  ```
- [ ] Embedding generation job (pg-boss cron, weekly):
  - Compose merchant profile text: name + country + plan + key metrics + score summary
  - Generate embedding via org's connected AI provider (BYO-key)
  - Store in `merchant_embeddings`

- [ ] Semantic search endpoint: `GET /api/v1/merchants/search?q=natural+language+query`
  - Convert query to embedding → cosine similarity search → return ranked results
  - Example: "merchants who installed last week but haven't configured features"

- [ ] "Similar merchants" widget on merchant detail page
  - Find 5 nearest neighbors by profile embedding
  - Show as "Merchants like this" sidebar card

**Cost note:** pgvector is free (Neon built-in). Embedding generation uses org's BYO AI key.

#### Task 4.3: Data Export Pipeline
**Priority:** P2 | **Effort:** 2 days | **Milestone:** M15

**What to build:**
- [ ] `POST /api/v1/export` — async export with job status polling
  - Supported formats: CSV, JSON, JSONL (for ML training)
  - Entities: merchants, events, metrics, scores, queue_history
  - Filters: date range, app, segment
  - PII handling: option to anonymize (hash emails, redact names)
  - Delivery: generate presigned URL, email when ready

- [ ] Export job handler in `packages/jobs/handlers/export.ts`
  - Chunked processing (10K rows per chunk)
  - Gzip compression for large exports
  - Auto-cleanup after 24 hours

- [ ] UI: `/settings/exports` — request export, view history, download

#### Task 4.4: Data Source Adapter Registry
**Priority:** P2 | **Effort:** 2 days | **Milestone:** M15

**What to build:**
- [ ] `DataSourceAdapter` interface in `packages/providers/data-source-adapter.ts` (see Part 2)
- [ ] Built-in adapters:
  - `ShopifyPartnerAdapter` — refactor existing sync logic to conform to interface
  - `AppThriveSdkAdapter` — refactor existing SDK ingest logic
  - `GenericWebhookAdapter` — configurable payload mapping via JSON schema
- [ ] Adapter registration in `/settings/integrations`
- [ ] Unmapped event logging: when an event doesn't match any adapter, log to `unmapped_events` table with payload for debugging

---

### Sprint 5 — Launch Readiness (Days 61-75)
**Theme:** Polish, test, and prepare for public launch

#### Task 5.1: Admin Panel Completion
**Priority:** P1 | **Effort:** 4 days | **Milestone:** M16

**PRD reference:** §15.3 (15 admin pages)

**What exists:** ~9 pages built in `apps/admin/`

**What to build:**
- [ ] `/admin/health` — system health dashboard
  - pg-boss queue depth, failed jobs (last 24h), active workers
  - ESP delivery rates (per provider)
  - API response times (p50, p95, p99 from Vercel Analytics)
  - Sync status per org (last sync time, failures)

- [ ] `/admin/abuse` — flagged accounts
  - High BASAI on free tier
  - High bounce rates (>5%)
  - Excessive API calls
  - Manual flagging capability

- [ ] `/admin/feature-flags` — per-org feature toggles
  - Table: `feature_flags` (orgId, featureKey, enabled, updatedBy, updatedAt)
  - UI: toggle grid with org search

- [ ] `/admin/announcements` — platform-wide banners
  - Create announcement (markdown, severity, target: all/plan/org)
  - Display in customer app header when active

- [ ] Impersonation flow (PRD §15.2):
  - "View as customer" button on account detail
  - Creates audit-logged session with read-only access
  - Banner in customer app: "You are being viewed by AppThrive support"

#### Task 5.2: Additional Workflow Playbooks
**Priority:** P2 | **Effort:** 2 days | **Milestone:** M6

**PRD reference:** §5.8.7 (10 playbooks, only 3 shipped)

**What to build (7 remaining):**
- [ ] Trial conversion nudge (5 touches, days 3-14)
- [ ] Billing failure dunning (3 attempts, 7 days, exit on resolution)
- [ ] Inactive merchant re-engagement (after 14 days no usage)
- [ ] Feature announcement (one-shot to segment)
- [ ] Upgrade nudge for plan-limit hitters
- [ ] NPS survey trigger (90 days post-install + high health)
- [ ] App review request (30 days + high health + high satisfaction)

Each playbook: workflow JSON + email template(s) + documentation

#### Task 5.3: EFOLI Dogfood Validation
**Priority:** P0 | **Effort:** 5 days | **Milestone:** M16

**Runbook:** `docs/runbooks/efoli-dogfood-week-1.md`

**What to validate:**
- [ ] Both apps (EmbedUp + DiscountRay) syncing continuously without errors
- [ ] Score computation produces meaningful differentiation across 603 merchants
- [ ] Action Queue generates relevant items each morning
- [ ] Morning digest email arrives at 08:00 BST
- [ ] Weekly report computes and dispatches for a test segment
- [ ] Queue "Act" → email send via connected ESP works end-to-end
- [ ] Workflow execution (Welcome Series) triggers on new install
- [ ] AI auto-handle proposes reasonable actions
- [ ] API endpoints return correct data via PAT
- [ ] Outbound webhook delivers to configured endpoint

**Acceptance:** All 10 flows work with real data for 5 consecutive business days

#### Task 5.4: CI/CD Hardening
**Priority:** P1 | **Effort:** 2 days | **Milestone:** M14

**Known issues from PROJECT-STATUS.md:**
- [ ] Fix typescript-eslint 8.58 plugin crash (bump past 8.58 or scope-disable)
- [ ] Run `prettier --write` on ~200 pre-existing files
- [ ] Fix compliance test flake (`PII_HASH_KEY` race condition)
- [ ] Add `EXPLAIN ANALYZE` CI check for slow queries (optional)
- [ ] Add Lighthouse CI for WCAG 2.1 AA (a11y score ≥95)

---

### Sprint 6 — Scale & Polish (Days 76-90)
**Theme:** Optimize for growth, prepare marketing launch

#### Task 6.1: Usage Dashboard & Billing Polish
**Priority:** P2 | **Effort:** 2 days | **Milestone:** M12

**What to build:**
- [ ] Usage stats card on `/settings/billing`:
  - Merchants synced: X / Y limit
  - API calls (30d): X / Y limit
  - Team seats: X / Y limit
  - Workflows active: X / Y limit
  - Storage (events retained): X days / Y limit
- [ ] "Upgrade" CTA when approaching limits (>80% usage)
- [ ] BASAI metering: count billable active merchants per month, display on billing page
- [ ] Overage warning emails (at 80%, 90%, 100% of plan limit)

#### Task 6.2: Dead Letter Queue UI
**Priority:** P2 | **Effort:** 1 day | **Milestone:** M3

**What exists:** `outbound_webhook_deliveries` with `status='dead'` entries

**What to build:**
- [ ] `/settings/webhooks/[id]/dead-letter` — list failed deliveries
  - Show: event type, payload preview, failure reason, attempt count, timestamps
  - Actions: retry single, retry all, dismiss
- [ ] Retry handler: re-publish to pg-boss with fresh attempt counter

#### Task 6.3: Audit Log Search UI
**Priority:** P2 | **Effort:** 1 day | **Milestone:** M11

**What exists:** Audit log table + export endpoint

**What to build:**
- [ ] `/settings/audit-log` page in customer app (not just admin)
  - Search by: actor, action type, date range, entity
  - Filter: config changes, sends, imports, API access
  - Pagination: cursor-based, 50 per page

#### Task 6.4: Performance Optimization Pass
**Priority:** P1 | **Effort:** 2 days | **Milestone:** M14

**What to do:**
- [ ] Add compound indexes where missing:
  - `metricThresholds(orgId, metricName, isActive)`
  - `events(orgId, merchantId, eventType, occurredAt)`
  - `queue_items(orgId, status, priority, createdAt)`
- [ ] Query optimization: identify and fix any N+1 patterns in server actions
- [ ] Add `connection: { max: 10 }` pool config to Drizzle Neon client
- [ ] Measure and log query times for top 20 endpoints (Axiom structured logging)

#### Task 6.5: Security Hardening
**Priority:** P1 | **Effort:** 1.5 days | **Milestone:** M14

**What to build:**
- [ ] PAT scope enforcement: add `scopes` column to `personal_access_tokens` table
  - Scopes: `read:merchants`, `write:merchants`, `read:queue`, `write:queue`, `read:events`, `write:events`, `admin`
  - Enforce at middleware level
- [ ] API key IP restriction (optional per org)
- [ ] Webhook secret rotation reminder (90-day nag in UI)
- [ ] CSRF enhancement: require `Origin` header for non-safe methods (not just `sec-fetch-site`)
- [ ] Input sanitization: ensure JSONB payloads in `customAttributes` can't inject into template rendering

---

## Part 4: PRD Update Requirements

The following PRD sections need updates based on current implementation reality:

| Section | Update Needed |
|---------|--------------|
| §5.3 Background sync | Document pg-boss (not Inngest) as production backend |
| §5.4.1 Event ingestion | Add batch endpoint spec (100 events/call) |
| §5.8.7 Playbooks | Mark 3 as shipped, 7 as planned |
| §8 API Surface | Update to 18 endpoints (was "30+ planned") |
| §9 Integrations | Add AI provider integration spec (Phase 21) |
| §11 Billing | Mark mock adapter as current, document live-swap steps |
| §12 NFRs | Add pgvector, MCP server, data export to V1+ scope |
| §13 Release Plan | Update timeline to reflect actual shipped dates |
| §14 Open Questions | Mark Q4 as RESOLVED (AI auto-handle shipped) |

---

## Part 5: Architecture Decisions for Agents

### Avoid External Paid Services

| Need | Solution | Cost |
|------|----------|------|
| Background jobs | pg-boss (Postgres-native) | $0 |
| Rate limiting | Upstash Redis (already connected) | Existing plan |
| Vector search | pgvector (Neon built-in) | $0 |
| Email rendering | React Email + MJML | $0 |
| AI inference | BYO-key (org provides their own) | $0 to platform |
| MCP server | Self-hosted in monorepo | $0 |
| WYSIWYG editor | Tiptap (MIT licensed) | $0 |
| Template engine | Handlebars (already used) | $0 |
| PDF generation | React PDF or html-pdf-node | $0 |
| Alerting | Sentry (already connected) | Existing plan |

### Key Technical Constraints

1. **Vercel serverless**: 10s default / 60s Pro function timeout. Long operations must use pg-boss jobs.
2. **Neon Postgres**: Serverless with auto-scaling. PgBouncer enabled. Max connections = plan-dependent.
3. **No `localStorage` in artifacts**: Use React state or server-side storage.
4. **Clerk auth**: All auth flows go through Clerk. No custom auth.
5. **Money = bigint cents USD**: Never floats, never local currency for computation.
6. **orgId everywhere**: Every DB query MUST be scoped to current org.

### File Organization for New Work

```
New package:     packages/{name}/src/index.ts + package.json
New API route:   apps/web/app/api/v1/{resource}/route.ts
New UI page:     apps/web/app/(app)/{feature}/page.tsx
New server action: apps/web/app/(app)/{feature}/actions.ts
New job handler: packages/jobs/handlers/{name}.ts
New DB schema:   packages/db/schema/{name}.ts + update index.ts
New migration:   packages/db/migrations/0020_{description}.sql
New test:        {package}/__tests__/{name}.test.ts
```

---

## Part 6: Sprint Dependency Graph

```
Sprint 1 (Foundation)
├── Task 1.1: Merchant 360 ──────────────────────┐
├── Task 1.2: Merchant Tags ─────────────────────┤
├── Task 1.3: Stripe Live Keys                    ├── Sprint 3 (Dev Config)
├── Task 1.4: DB Partitioning                     │   ├── Task 3.1: Custom Fields
└── Task 1.5: Per-Endpoint Rate Limits            │   ├── Task 3.2: Bulk Operations
                                                   │   ├── Task 3.3: Event Registration
Sprint 2 (Communication)                          │   └── Task 3.4: Template Editor
├── Task 2.1: ESP Adapters                         │
├── Task 2.2: ESP Engagement Metrics              │
├── Task 2.3: Response Caching                     ├── Sprint 5 (Launch)
├── Task 2.4: Backpressure                         │   ├── Task 5.1: Admin Panel
└── Task 2.5: Event Timeline ─────────────────────┤   ├── Task 5.2: Playbooks
                                                   │   ├── Task 5.3: EFOLI Dogfood ← ALL
Sprint 4 (AI-Native)                               │   └── Task 5.4: CI Hardening
├── Task 4.1: MCP Server                           │
├── Task 4.2: Vector Embeddings                    ├── Sprint 6 (Scale)
├── Task 4.3: Data Export                          │   ├── Task 6.1: Usage Dashboard
└── Task 4.4: Adapter Registry                     │   ├── Task 6.2: DLQ UI
                                                   │   ├── Task 6.3: Audit Log UI
                                                   │   ├── Task 6.4: Performance
                                                   │   └── Task 6.5: Security
```

**Critical path:** Sprint 1 → Sprint 5 (Task 5.3 EFOLI Dogfood depends on all Sprint 1 tasks)

---

## Part 7: Success Metrics

### Pre-Launch (EFOLI Dogfood)
- [ ] 603 merchants scored and queue-generating daily for 5 consecutive days
- [ ] Zero data integrity issues (no cross-org leakage, no orphan records)
- [ ] API response times: p50 <200ms, p95 <500ms, p99 <2s
- [ ] Zero unhandled errors in Sentry for 48 hours

### Launch (First 50 Beta Users)
- [ ] Onboarding completion rate >80% (5-step wizard)
- [ ] Time to first queue item: <24 hours after app connection
- [ ] Weekly report delivery rate >95%
- [ ] NPS >40 from beta cohort

### Scale (1000+ Orgs)
- [ ] Database handles 10K events/minute sustained
- [ ] No partition-related query slowdowns
- [ ] AI auto-handle adoption >30% of queue items on Pro+ orgs
- [ ] MCP integration used by >10% of orgs

---

_This document is the authoritative sprint plan for taking AppThrive from v0.5 to production launch. Each task is designed to be independently assignable to an agent. When starting a task, read the referenced source files first, then implement according to the conventions in CLAUDE.md and CONTRIBUTING.md._
