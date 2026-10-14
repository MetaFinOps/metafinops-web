# MetaFinOps SaaS Platform - Implementation Plan

## Tech Stack
- **Backend:** Python + FastAPI (async)
- **Frontend:** React + Next.js 16 (App Router) + shadcn/ui + Tailwind CSS
- **Database:** Cloud SQL (PostgreSQL) + BigQuery (analytics warehouse)
- **Auth:** Clerk (built-in orgs, RBAC, pre-built UI, 10K free MAUs)
- **Task Queue:** Celery + Redis (Memorystore)
- **Hosting:** GCP (Cloud Run, Cloud SQL, BigQuery, Pub/Sub, Memorystore)
- **CI/CD:** GitHub Actions → Cloud Build → Cloud Run

## MVP Scope: Cloud Cost Dashboard
Multi-cloud cost ingestion (AWS/GCP/Azure), cost visualization dashboards, anomaly detection, and alerting.

---

## Phase 1: Project Foundation (Steps 1-5)

### Step 1 - Monorepo Scaffold
Create the project structure:
```
metafinops-platform/
├── backend/          # FastAPI app
├── frontend/         # Next.js app
├── infrastructure/   # Terraform IaC
├── shared/           # Shared types
├── docker-compose.yml
├── Makefile
└── README.md
```

### Step 2 - Backend Foundation
- FastAPI app with project structure (api/, core/, models/, schemas/, crud/, services/, workers/)
- Pydantic v2 settings, structured logging, exception handlers
- Async SQLAlchemy 2.0 + Alembic migrations
- PostgreSQL schema: organizations, users, org_members, cloud_accounts, cost_daily, anomalies, alert_rules, alert_history
- Row-Level Security (RLS) for multi-tenant data isolation
- Health check and OpenAPI docs

### Step 3 - Frontend Foundation
- Next.js 16 with App Router, TypeScript, Tailwind CSS
- shadcn/ui components + dashboard layout (sidebar, nav, org switcher)
- Auth pages (login/signup) with Clerk integration
- Dashboard shell with placeholder pages: Overview, Costs, Anomalies, Alerts, Settings
- Recharts + Tremor for data visualization components

### Step 4 - Authentication (Clerk)
- Clerk integration: frontend `<ClerkProvider>`, `<SignIn>`, `<OrganizationSwitcher>`
- Backend JWT validation middleware extracting user_id + org_id
- Webhook endpoint to sync user/org creation into PostgreSQL
- RBAC: owner, admin, editor, viewer roles

### Step 5 - Docker + Local Dev Environment
- docker-compose.yml: PostgreSQL, Redis, FastAPI, Celery worker, Next.js
- Makefile with common commands (dev, test, migrate, seed)
- Environment variable management (.env files)

---

## Phase 2: Cloud Cost Data Pipeline (Steps 6-9)

### Step 6 - Cloud Account Connection
- API endpoints for connecting AWS/GCP/Azure accounts
- Credentials stored in GCP Secret Manager
- Frontend: Settings → Cloud Accounts page with connection flow
- Account status tracking (pending, active, error)

### Step 7 - Cost Data Ingestion Workers
- Celery tasks for each provider:
  - AWS: boto3 Cost Explorer API
  - GCP: BigQuery Billing Export
  - Azure: Cost Management API
- Data normalization to unified format (aligned with FOCUS standard)
- Write to PostgreSQL (recent data) + BigQuery (full history)
- Cloud Scheduler triggers ingestion every 6 hours

### Step 8 - Cost API Endpoints
- GET /costs/summary - KPI totals (total spend, delta, top services)
- GET /costs/daily - Time series data
- GET /costs/by-service - Service breakdown
- GET /costs/by-provider - Provider comparison
- GET /costs/by-region - Region breakdown
- GET /costs/by-tag - Tag-based grouping
- All endpoints support: date_from, date_to, provider, account_id, granularity filters

### Step 9 - Cost Dashboard UI
- Overview page: KPI cards (total spend, month-over-month change, top 5 services)
- Daily cost trend chart (Recharts line/area chart)
- Cost breakdown donut/bar charts (by service, provider, region)
- Cost explorer with filters (date range, provider, service, tags)
- Data table with sorting, filtering, CSV export

---

## Phase 3: Analytics & Alerting (Steps 10-13)

### Step 10 - Anomaly Detection
- Statistical anomaly detection (Z-score + Prophet/statsforecast)
- Celery task runs daily, analyzes cost patterns per service
- Detects: spikes, drops, trend changes
- Severity levels: low, medium, high, critical
- API: GET /anomalies, PATCH /anomalies/:id (acknowledge/resolve)

### Step 11 - Anomaly Dashboard UI
- Anomaly list with severity badges, status filters
- Anomaly detail view with cost chart showing expected vs actual
- Acknowledge/resolve workflow

### Step 12 - Alert Rules Engine
- Alert types: budget threshold, anomaly-triggered, forecast-based
- Notification channels: email (SendGrid), Slack (webhook)
- API: CRUD for alert rules, alert history
- Celery task evaluates rules after anomaly detection

### Step 13 - Alerts UI
- Alert rules management page (create/edit/delete)
- Alert history with status tracking
- Test notification button
- Slack integration setup flow

---

## Phase 4: Polish & Deploy (Steps 14-16)

### Step 14 - Infrastructure (Terraform)
- GCP Terraform modules: Cloud Run (API + workers), Cloud SQL, BigQuery, Memorystore, Pub/Sub, Secret Manager, Cloud Scheduler
- Staging + production environments
- Cloud Armor WAF, Cloud Monitoring

### Step 15 - Testing & Security
- Backend: pytest with unit + integration tests
- Frontend: Jest + React Testing Library
- API load testing
- Security: rate limiting, input validation, CORS, CSP headers

### Step 16 - Production Launch
- Deploy to GCP (staging first, then production)
- Custom domain + SSL
- Monitoring dashboards (Cloud Monitoring)
- Onboarding flow, documentation, empty states

---

## Key Libraries

**Backend (Python):**
fastapi, uvicorn, sqlalchemy[asyncio], alembic, pydantic, celery[redis], boto3, azure-mgmt-costmanagement, google-cloud-bigquery, httpx, pandas, scikit-learn, sendgrid, slack-sdk, pyjwt

**Frontend (Node.js):**
next, shadcn/ui, tailwindcss, recharts, @tremor/react, @tanstack/react-query, @tanstack/react-table, @clerk/nextjs, react-hook-form, zod, zustand, lucide-react, date-fns

---

## Estimated Timeline
- Phase 1 (Foundation): Weeks 1-4
- Phase 2 (Data Pipeline): Weeks 5-8
- Phase 3 (Analytics): Weeks 9-12
- Phase 4 (Polish & Launch): Weeks 13-16
- **Total: ~16 weeks**

## Starting Now: Step 1 - Monorepo Scaffold
We'll create a new repo `metafinops-platform` and set up the full project structure with backend (FastAPI), frontend (Next.js), Docker Compose for local dev, and the Makefile.
