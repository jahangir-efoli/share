# AppThrive: Shopify App Lifecycle CRM
## Developer Milestone Document

### Primary Goal
Build a multi-tenant CRM for Shopify app organizations to manage merchant lifecycle, app usage, communication, and retention.

---

## 1. Core Data Ownership Rules

### 1.1 Tenant Model
- Multi-tenant platform
- Each organization = workspace
- Workspace can own multiple apps
- All data belongs to exactly one workspace

### 1.2 Store Ownership Model
- Same Shopify store can exist in multiple workspaces
- Records must remain isolated
- No cross-workspace data access

### 1.3 App-Scoped Operational Model
Within a workspace:
- One store → multiple app installations
- Each app has independent:
  - plan
  - trial
  - events
  - lifecycle state

### 1.4 Same-Workspace Aggregation
Allowed only within the same workspace across its apps.

### 1.5 Security Rule
Ownership resolution order:
1. Workspace  
2. App  
3. Role/Permission  

---

## 2. Milestones Overview

| Milestone | Name | Priority | Outcome |
|----------|------|----------|--------|
| M0 | Tenant Isolation | P0 | Secure foundation |
| M1 | Historical Sync | P0 | Backfill data |
| M2 | Merchant Identity | P0 | Store + app model |
| M3 | Event Pipeline | P0 | Real-time updates |
| M4 | Messaging Infra | P0 | Transactional email |
| M5 | Automation Engine | P1 | Lifecycle flows |
| M6 | Segmentation | P1 | Campaign targeting |
| M7 | Reporting | P1 | Merchant insights |
| M8 | Dev Config Layer | P1 | App-level control |
| M9 | SaaS Readiness | P2 | Monetization & scale |

---

## M0 — Tenant Isolation

### Goal
Secure multi-tenant foundation.

### Requirements
- workspace_id on all records
- RBAC roles:
  - owner, admin, developer, marketing, support, read-only
- API filtering by workspace
- encrypted secrets
- audit logs

### Acceptance
- No cross-tenant access
- All data tied to workspace

---

## M1 — Historical Data Sync

### Goal
Import large-scale historical data asynchronously.

### Features
- Background jobs
- Chunk processing
- Retry system
- Import UI with progress

### Entities
- stores
- app installs
- subscriptions
- events

### Acceptance
- Large imports don’t block UI
- Dedup works per workspace

---

## M2 — Tenant-Scoped Merchant Identity

### Goal
Workspace-scoped store + app relationships.

### Core Models

#### Store
- workspace_id
- shopify_domain
- metadata (location, plan, etc.)
- tags, custom fields

#### App Installation
- app_id
- store_id
- plan
- trial
- status

### Rules
- No cross-workspace merge
- App data remains scoped

---

## M3 — Event & Webhook Pipeline

### Goal
Reliable real-time updates.

### Features
- webhook ingestion
- idempotency
- retry + DLQ
- event timeline

### Acceptance
- No duplicate processing
- Events update state correctly

---

## M4 — Messaging Infrastructure

### Goal
Transactional email system.

### Features
- Provider abstraction (SendGrid, Resend, SES)
- Domain verification
- Suppression handling
- Event-based triggers

### Safeguards
- rate limits
- unsubscribe handling
- bounce tracking

---

## M5 — Lifecycle Automation Engine

### Goal
Event-driven automation flows.

### Triggers
- install/uninstall
- trial ending
- inactivity

### Actions
- send email
- tagging
- scoring
- delays

### Safety
- frequency caps
- quiet hours
- no duplicate flows

---

## M6 — Segmentation & Campaigns

### Goal
Targeted communication.

### Features
- dynamic segments
- campaign scheduling
- preview counts

### Filters
- plan
- app
- activity
- location

---

## M7 — Reporting & Health Scoring

### Goal
Merchant insights.

### Reports
- weekly/monthly
- app performance
- recommendations

### Scores
- activation
- engagement
- churn risk

---

## M8 — Developer Configuration Layer

### Goal
No-code app configuration.

### Configurable
- events
- templates
- flows
- reports

---

## M9 — SaaS Readiness

### Goal
Commercial product readiness.

### Features
- plan limits
- usage tracking
- onboarding wizard
- admin tools

---

## 3. Cross-Cutting Requirements

### Observability
- logs (events, jobs, emails)
- alerts

### Idempotency
- imports
- webhooks

### Audit Logs
- config changes
- sends
- imports

---

## 4. Priority

### P0 (Core)
- M0–M4

### P1 (Value)
- M5–M8

### P2 (Scale)
- M9

---

## 5. Delivery Phases

### Phase A
M0–M3

### Phase B
M4

### Phase C
M5–M7

### Phase D
M8–M9

---

## 6. Engineering Rule

> All merchant data is workspace-owned.  
> All operational data is app-scoped.  
> No cross-workspace exposure is allowed.

