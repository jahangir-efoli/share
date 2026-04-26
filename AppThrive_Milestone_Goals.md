Below is a developer-ready milestone document using the corrected model:
multi-tenant
workspace-scoped data ownership
app-scoped operational data
optional same-workspace multi-app aggregation only
strict no cross-organization data exposure

Developer Milestone Document
Project: AppThrive: Shopify App Lifecycle CRM
Document Type: Development Milestone & Acceptance Specification
Primary Goal: Build a multi-tenant CRM for Shopify app organizations to manage merchant lifecycle, app usage, communication, and retention.

1. Core Data Ownership Rules
1.1 Tenant model
The platform supports multiple organizations.
Each organization is represented as a workspace.
A workspace can own multiple apps.
All data must belong to exactly one workspace.
1.2 Store ownership model
A Shopify store may appear in multiple workspaces across the platform.
Those records must remain logically and operationally separate.
No workspace can access another workspace’s store, install, event, or communication data.
1.3 App-scoped operational model
Within a workspace:
a store may have multiple app installations,
one installation per app relationship,
plans, trials, events, onboarding state, billing state, and lifecycle state are app-scoped.
1.4 Same-workspace aggregation
Allowed:
one workspace may view a store across its own apps,
only if the apps belong to that same workspace,
app-specific details must still remain distinguishable.
1.5 Security rule
Every query, API response, background job, import, export, webhook, message, template, and event must resolve ownership in this order:
workspace
app
role/permission

2. Milestone Overview

## M0 — Tenant Isolation & Platform Foundations
Goal
Establish secure workspace isolation, ownership boundaries, authentication, authorization, and technical patterns that every later milestone depends on.
Scope
workspace model
user auth
role-based access
ownership enforcement
audit log baseline
workspace-aware API filtering
secure secrets storage
environment separation
Functional Requirements
Users can sign up and belong to a workspace.
A workspace can have multiple users.
Roles must support at least:
owner
admin
developer
marketing
support
read-only
All records must contain workspace_id.
Sensitive app/provider credentials must be encrypted at rest.
Every API endpoint must enforce workspace ownership.
Audit logs must record:
login
role changes
provider connection
domain verification
export action
flow publish/update
campaign send/start/pause
bulk import action
Non-Functional Requirements
no cross-tenant reads
no cross-tenant writes
no shared object access by guessed IDs
secure session handling
rate limiting for auth endpoints
Acceptance Criteria
user from Workspace A cannot retrieve any entity from Workspace B
API access is filtered by workspace automatically
all created records are tied to workspace
audit logs are generated for critical actions
encrypted secrets are not exposed in logs or API responses
Technical Notes
Prefer row-level ownership enforcement in application logic and database constraints
Strongly consider UUIDs instead of sequential IDs
Add middleware that resolves workspace ownership early
Done When
secure workspace model exists
all core objects require workspace_id
permission checks are enforced consistently

## M1 — Historical Data Sync
Goal
Import historical data from Shopify Partner/API/app sources into the correct workspace and app context, at large scale, using background processing.
Scope
backfill imports
queue jobs
progress tracking
retries
failure reporting
deduplication
source attribution
Inputs
workspace app credentials
Shopify partner/app data source
app backend exports/APIs
CSV uploads where needed
date range filters
import options
Entities Covered
workspace apps
stores
app installations
subscriptions / plans
trial windows
app events
contacts (if available and allowed)
communication history (optional later)
Functional Requirements
Import jobs must run asynchronously.
Jobs must show:
queued
running
partial success
failed
completed
Large imports must be chunked.
Failed chunks must be retryable.
Duplicate stores within a workspace must be resolved safely.
Same store across different workspaces must remain separate.
Imported app relationships must preserve app ownership.
Completion/failure notifications must be sent to the initiating user.
Required UI/UX
import wizard
import history page
live status/progress
row/chunk failure summary
retry button
completion notification
Edge Cases
same store installed multiple apps in same workspace
same store appears in multiple workspaces
missing fields
stale or conflicting plan data
partial API outages
duplicate webhook/history overlap after import
Metrics
import success rate
duplicate resolution rate
average import duration
retry recovery rate
failed record percentage
Acceptance Criteria
historical data for one app can be imported end-to-end
large dataset import does not block UI
deduplication works within workspace boundaries
app-install relationships are preserved
user receives final import summary
Done When
old data from at least one connected app can be imported reliably at scale
imports are observable and retryable

## M2 — Tenant-Scoped Merchant Identity
Goal
Create a workspace-scoped merchant/store record model that supports multiple apps in the same workspace while keeping app-specific lifecycle data separate.
Scope
workspace store record
app installation relationship model
app-scoped plan/trial/activity
field source tracking
identity resolution within workspace only
Data Model Requirements
Workspace Store
Represents a store inside one workspace only.
Data sources: Shopify partner API + Sync from the App via API/webhooks
Fields may include:
workspace_id
Shopify Shop ID
myshopify_domain
primary_domain
Store_name
Shop owner
Country code
Country name
state/region
city
timezone
currency
phone
Email
Shopify Plan
Primary locale
Address1
Address2
Zip
Phone
Customer email
Timezone
Multi-location enabled
public social links
enrichment status
Custom fields (can be added by developers to store and retrieve any additional data they want)
Tags (for grouping or different identification)
Business summary (can be auto-generated by using AI)
Additional comments (comments can have multiple come from different sources, like the support team, old comments can be preserved with new ones, with the identity of who commented and the source)
last_enriched_at
App Installation
Represents one app-to-store relationship.
Fields may include:
workspace_id
app_id
store_id
install_status
install_date
uninstall_date
current_app_plan
trial_start
trial_end
billing_status
onboarding_status
last_active_at
Functional Requirements
A store can exist once per workspace store identity.
That store may link to multiple app installations inside the same workspace.
App events, Scores, Metrics, App Subscriptions, plans, trials, and onboarding data must remain app-scoped.
Workspace-level summary may show combined view across same-workspace apps only.
Each field should optionally track:
source type
source name
updated_at
confidence/verification state
Important Rules
no cross-workspace identity merge
no platform-wide unified store profile
no cross-organization aggregation
same-workspace aggregation is allowed only for that workspace’s apps
Metrics
store identity match accuracy within workspace
duplicate store resolution rate within apps
percentage of installs linked to correct workspace store
profile completeness score
Acceptance Criteria
same store using 3 apps in one workspace can be shown under one workspace store record
app-specific lifecycle states remain separate
same store in another workspace is not merged or visible
field-level source metadata can be stored
Done When
workspace store identity and app installation model are stable and queryable

## M3 — Real-Time Event & Webhook Pipeline
Goal
Keep store/app/install/plan/activity state current through Shopify webhooks and internal app event webhooks.
Scope
webhook ingestion
app event ingestion
signature verification
idempotency
retries
dead-letter queue
replay tools
event timeline
Supported Sources
Shopify partner API/webhooks
app backend webhooks
internal SDK event ingestion
external webhook/API event posts
Functional Requirements
Every event must include workspace and app ownership context.
Webhook authenticity must be verified where applicable.
Duplicate deliveries must not corrupt state.
Failed processing must be retried.
Permanently failed events must move to dead-letter state.
Admin tools must allow replay/reprocess.
Processed events must appear in a store/app timeline.
Event Types (may include events from Shopify partner API or Apps SDK/API/webhooks)
install / uninstall
plan upgraded / downgraded
trial started / trial ended
feature events
onboarding events
activity / inactivity markers
billing issue events
messaging events
Technical Requirements
append-only raw event storage
normalized derived state updates
idempotency key support
event versioning support
Metrics
event success rate
event processing latency
duplicate handling accuracy
retry success rate
freshness lag
Acceptance Criteria
live events update app installation state correctly
duplicate events do not create duplicate actions
failed events can be replayed
event history is visible per store and per app installation
Done When
real-time updates are reliable enough for lifecycle automation

## M4 — Messaging Infrastructure
Goal
Provide workspace-owned, app-aware transactional email, onboarding, campaign, event based infrastructure with provider abstraction, verified domains, suppression, and safeguards.
Scope
email provider integration
sending domains (can have different domains per app or one org level for all apps)
authentication checks
suppression handling
bounce/complaint tracking
transactional triggers
app-specific sender settings
Initial Supported Providers
Resend and SendGrid
Architecture must support more later like tosend, amazon, WhatsApp, Telegram or any other.
Recommended abstraction fields:
provider type
provider account
API key
stream/type
sender identity
domain mapping
webhook secret
status
Functional Requirements
A workspace can connect an email provider.
A workspace can verify one or more domains.
A workspace can verify or use different domains for different apps.
Each app can define:
sender name
sender email
template mapping
enabled transactional events
Transactional emails can be sent for:
app install
uninstall
plan upgrade
plan downgrade
trial ending
billing issue
onboarding milestone
Or any other (can be defined by app developers per app)
Delivery events must update CRM records.
Per apps suppression must block sends.
Workspace rate limits must exist.
Per-app send controls must exist.
Safeguards
no sends from unverified domain
unsubscribe honored where applicable (per apps and email types)
bounce handling
complaint tracking
provider failure fallback behavior defined
send logging required
Metrics
delivery rate
bounce rate
complaint rate
send success rate
event-to-email latency
verified domain adoption rate
Acceptance Criteria
one workspace can connect provider and verified domain
one app can send configured transactional emails
delivery/bounce/complaint events appear in logs
suppression prevents sending to blocked recipients
Done When
transactional email is production-safe for early customers

## M5 — Lifecycle Automation Engine
Goal
Enable app-specific lifecycle flows driven by events, time conditions, plan state, and app activity.
Scope
trigger engine
condition engine
action engine
wait/delay logic
branch logic
per-app flow ownership
flow execution logs
guardrails
Supported Triggers
app installed
app uninstalled
trial ending soon
onboarding incomplete
inactivity threshold
feature milestone reached
free plan limit reached
billing issue
segment entry
Supported Conditions
app equals X
current plan equals Y
trial days remaining
country/region
last active before/after
event count threshold
onboarding step completed/not completed
message already sent/not sent
Supported Actions
send email, messages (WhatsApp, Telegram, or SMS etc)
add/remove tag
create task
set score
delay/wait
branch
stop flow
webhook to external system
internal notification
Flow Safety Rules
no duplicate flow entry unless explicitly allowed
frequency caps
quiet hours by timezone
stop on uninstall if configured
stop on unsubscribe for marketing communications
conflict handling between flows
Prebuilt MVP Flows
onboarding sequence
trial ending flow
uninstall follow-up
inactivity rescue
free-to-paid upgrade assist
first-value milestone message
Metrics
flow trigger success rate
flow completion rate
onboarding completion uplift
trial-to-paid uplift
reactivation rate
error rate in flow execution
Acceptance Criteria
developer can configure per-app flow rules
flow logs show trigger, steps, and outcomes
safety controls prevent over-messaging
at least 5 prebuilt flows operate end-to-end
Done When
lifecycle automation is useful without custom engineering per app

## M6 — Segmentation & Campaigns
Goal
Allow workspaces to build merchant segments and run targeted campaigns using workspace-owned and app-scoped data.
Scope
filter builder
saved segments
exclusions
campaign send flow
preview/estimation
analytics
suppression/frequency controls
Functional Requirements
Segments can filter by:
app
current plan
trial state
country
install date
activity level
event occurrence
tag
communication history
Segments can be:
static
dynamic
Campaign flow must support:
draft
test
scheduled
running
paused
completed
User must preview recipient counts before send.
Campaign must enforce:
suppression
unsubscribe
workspace limits
per-app exclusions if defined
MVP Campaign Types
email only
one-time campaign
scheduled recurring campaign
Metrics
recipient count accuracy
open/click/reply rates
unsubscribe rate
complaint rate
conversion by campaign
error rate
Acceptance Criteria
segment can be built from app-scoped and workspace-owned fields
campaign preview count is accurate enough for operational use
send logs and performance metrics are visible
blocked recipients are excluded correctly
Done When
teams can run targeted campaigns safely

## M7 — Merchant Reporting & Health Scoring
Goal
Generate merchant-facing value reports and internal health intelligence using app-scoped activity and optional same-workspace multi-app context.
Scope
weekly reports
monthly reports
Custom reports (can be defined by apps)
health scores
merchant success blocks
internal action recommendations
Merchant-Facing Reports
Configurable per app:
usage summary
milestones completed
app impact summary
missed opportunities
recommended next actions
free plan limit proximity
feature suggestions
Internal Health Scores
At minimum:
activation score
engagement score
churn risk score
expansion readiness score
Rules
scores may use app-scoped data first
same-workspace multi-app signals may be added if explicitly enabled
no cross-workspace benchmarking exposure
Functional Requirements
Developers can define which report blocks belong to each app.
Reports can use variables and computed metrics.
Reports can be scheduled weekly/monthly or on custom dates/times.
Internal dashboard must show high-risk and high-opportunity stores.
Metrics
report generation success rate
report open/click rate
upgrades after reports
churn reduction among recipients
health score predictive usefulness
Acceptance Criteria
one app can send weekly success reports
internal dashboard can show at-risk stores
configurable report blocks work correctly
Done When
reports provide clear merchant value and internal actionability

## M8 — Developer Configuration Layer
Goal
Give app developers structured control over app-specific logic without code changes for every scenario.
Scope
app settings
event schema registration
template mapping
transactional rules
flow configuration
reporting blocks
sender identities
safety settings
Configurable Objects
Per app:
event definitions
event property schema
onboarding milestones
transactional email toggles
flow templates
report content blocks
default sender identity
rate/frequency rules
unsubscribe groups if supported
Functional Requirements
Developer can define app events and their schema.
Developer can mark certain events as lifecycle-relevant.
Developer can map events to:
transactional emails
flow triggers
score inputs
report blocks
Developer can enable/disable app-specific messaging logic.
Configuration changes must be versioned.
Metrics
time to configure new app
misconfiguration rate
publish success rate
rollback frequency
Acceptance Criteria
new app can be configured without engineering changes to the core platform
event-driven features can reference developer-defined app events
template mapping is editable and versioned
Done When
onboarding a new app is mostly configuration-driven

## M9 — SaaS Readiness & Commercial Controls
Goal
Prepare the product for external organizations with pricing limits, usage enforcement, support tooling, and onboarding.
Scope
billing hooks
plan limits
seat limits
app limits
store/install limits
usage counters
onboarding wizard
admin diagnostics
support tooling
Functional Requirements
Plans must support limits on:
number of apps
active store-app installations
email volume
events processed
seats
Limit warnings must be shown before hard stop.
Admin/support tools must allow:
inspect import status
inspect webhook failures
inspect send failures
inspect plan usage
New workspace onboarding must guide:
workspace setup
app connection
provider connection
initial import
first flow
Metrics
time to first value
onboarding completion rate
support ticket rate
plan upgrade rate
limit breach frequency
Acceptance Criteria
external workspace can onboard mostly self-serve
commercial limits are enforced
admins/support can troubleshoot common failures
Done When
product is viable as independent SaaS

3. Cross-Cutting Technical Requirements
3.1 Observability
Required across all milestones:
structured logs
job logs
event logs
send logs
import logs
error classification
alerting for repeated failures
3.2 Idempotency
Must exist for:
imports
webhooks
event ingestion
send operations where applicable
3.3 Auditability
Must log:
who changed flow/template/config
who started import/export
who sent/paused campaign
who connected provider/domain
3.4 Retry & Recovery
Must support:
retry transient failures
DLQ for permanent failures
manual replay for events/jobs
safe reprocessing
3.5 Permissions
Every sensitive action must check:
workspace membership
role permissions
app access if app-scoped restriction exists


4. Priority Labels for Engineering
# P0
Must exist before production pilot:
M0 Tenant Isolation
M1 Historical Sync
M2 Tenant-Scoped Merchant Identity
M3 Real-Time Event Pipeline
M4 Messaging Infrastructure
# P1
Core product value:
M5 Lifecycle Automation
M6 Segmentation & Campaigns
M7 Reporting & Health Scoring
M8 Developer Configuration Layer
# P2
Go-to-market / platform maturity:
M9 SaaS Readiness
advanced analytics
multi-provider failover
multichannel messaging beyond email

5. Suggested Delivery Order
Phase A — Foundation
# M0
# M1
# M2
# M3
Phase B — Communication Base
# M4
Phase C — Product Value
# M5
# M6
# M7
Phase D — Scale & Configuration
# M8
# M9

6. Sample Acceptance Test Scenarios
Scenario 1 — Same store in two workspaces
Workspace A imports store X from app A1
Workspace B imports same store X from app B1
Result: each workspace sees only its own record set
Scenario 2 — Same store, multiple apps in one workspace
Workspace A has apps A1 and A2
store X installs both
Result: one workspace store record, two app installations, separate app states
Scenario 3 — Historical import + webhook overlap
old data import includes install
webhook for same install arrives later
Result: no duplicate install record, state remains correct
Scenario 4 — Suppressed email recipient
contact unsubscribed earlier
flow or campaign attempts send
Result: send blocked and logged
Scenario 5 — Uninstall flow
app uninstall event received
app-specific follow-up sequence starts if enabled
only for that app/workspace
flow stops if recipient is suppressed

7. What the Development Agent Should Build First
First engineering slice should include:
workspace auth and permissions
app registration under workspace
workspace store + app installation schema
historical import queue with progress tracking
raw event ingestion pipeline
one email provider integration
one transactional template send flow
audit logs and basic admin diagnostics
That gives a stable base for later automation.

8. Product Success Criteria by Stage
Foundation success
data is secure
imports are reliable
event updates are trustworthy
Communication success
transactional sends are reliable
suppression and safeguards work
CRM success
app-specific lifecycle flows work
segments are accurate
reports are useful
SaaS success
onboarding is self-serve enough
limits and permissions are enforceable

9. Final Instruction for Engineering
Use this rule everywhere:
All merchant data is workspace-owned.
All operational state is app-scoped unless explicitly defined as same-workspace aggregate.
No cross-workspace merge, access, inference, or exposure is allowed.
