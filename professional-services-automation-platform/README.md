# Professional Services Automation Platform

## ⚠️ Proprietary Work & Copyright Notice

This case study represents proprietary methodologies and NDA-compliant frameworks.

**This project is NOT open-source.**

© 2026 Rohail K. Malhi. All rights reserved.

You are welcome to read and review these materials to understand my professional capabilities. However, you are **strictly prohibited** from copying, adapting, or utilizing these artifacts, structures, or content in any form. See [LICENSE](LICENSE).

---

**A multi-tenant B2B SaaS that runs the back office of professional-services teams — projects, people, time, billing, and documents — on one integrated system.**

> **Confidentiality note.** This is a sanitized portfolio overview. The product name, client identity, and proprietary business rules are withheld under NDA. Everything here describes capabilities and engineering approach at a level safe for public sharing. All screenshots use fictional demo data — no real client information.

> 📄 **Client-facing case study (C-S-R):** [`professional-services-automation-platform_case_study.pdf`](professional-services-automation-platform_case_study.pdf) — a polished, shareable PDF with Challenge → Solution → Result and embedded screenshots.

---

## The problem

Professional-services teams (agencies, consultancies, software studios) run their operations across a patchwork of disconnected tools: one app for time, another for projects, a spreadsheet for capacity, a separate system for invoicing, and email for everything else. Data is re-keyed between them, margins are guessed at, and invoicing lags weeks behind delivery.

This platform collapses that stack into a single, workspace-isolated system: a team plans work, tracks time against it, approves it, bills it, and stores the paperwork — without leaving the product, and without any of their data ever touching another tenant's.

---

## Screenshots

> Fictional demo workspace. No real client data.

### Role-aware dashboard

The dashboard renders a different operational view depending on the signed-in user's role — resolved from their **permissions**, so custom roles resolve correctly too. A filter bar (date range · project scope) drives every KPI and chart, and the whole surface is theme-aware.

| | |
|---|---|
| **Leadership** (Owner / Admin) — org KPIs with trend deltas, revenue trend vs. target, project-status mix, budget health, and the approvals queue | ![Leadership dashboard](images/01-dashboard-leadership.png) |
| **Manager** — approval queue with one-click review, team capacity vs. weekly cap (over-allocation flagged), project budgets, and team activity | ![Manager dashboard](images/01c-dashboard-manager.png) |
| **Member** — my hours vs. goal, my week, timesheet status, and prioritized tasks — shown here in **dark theme** | ![Member dashboard](images/01d-dashboard-member-dark.png) |
| **The same dashboard adapts to light or dark** (Leadership, dark) | ![Leadership dashboard in dark theme](images/01b-dashboard-leadership-dark.png) |

### Delivery lifecycle — track → approve → bill

The operational spine of a services firm: log time against tasks, submit it for approval, invoice from the approved time, and report on utilization and margin.

| | |
|---|---|
| **Timesheet** — a member's weekly grid: hours per task per day, billable flags, and submit-for-approval | ![Weekly timesheet](images/07-timesheet-member.png) |
| **Timesheet approvals** — a manager's review queue with one-click approve / reject and utilization at a glance | ![Timesheet approvals](images/08-timesheet-approvals.png) |
| **Invoicing** — generate an invoice from approved time: line items, tax, discount, and lifecycle status (draft → sent → paid) | ![Invoicing](images/09-invoicing.png) |
| **Reports & analytics** — utilization vs. target, project profitability (revenue vs. cost), and margin by project | ![Reports and analytics](images/10-reports-analytics.png) |

### Core screens

| | |
|---|---|
| **Projects** — portfolio with search, filter, sort | ![Projects](images/02-projects.png) |
| **Project detail** — overview, budget & billing defaults | ![Project detail](images/03-project-detail.png) |
| **Clients** — company records & contacts | ![Clients](images/04-clients.png) |
| **People** — the workspace roster: role, status, engagement type, and billing rate | ![People](images/12-people.png) |
| **Roles & permissions** — RBAC roles and their permission scope | ![Roles and permissions](images/05-people-roles.png) |
| **Documents** — files with visibility controls | ![Documents](images/06-documents.png) |
| **Workspace settings** — general, branding, email, and integrations | ![Workspace settings](images/13-settings.png) |
| **Sign in** — email/password with a workspace-aware, permission-driven portal behind it | ![Sign in](images/11-login.png) |

---

## What it does

The platform is one integrated back office for a services team — from setting up a client, through delivering and tracking the work, to billing it and reporting on it — with a strict tenant boundary and server-enforced permissions at every step.

![Professional-services delivery workflow: set up client and project, assign teams and people, track time against tasks, submit and approve timesheets, invoice from approved time, and report on utilization and resource planning — with Documents scoped to project, client, or workspace](images/diagram-workflow.png)

<details>
<summary>Mermaid source (renders live on GitHub)</summary>

```mermaid
flowchart LR
    A["1 · Set up<br/>client &amp; project"] --> B["2 · Assign<br/>teams &amp; people"]
    B --> C["3 · Track time<br/>against tasks"]
    C --> D["4 · Submit &amp;<br/>approve timesheets"]
    D --> E["5 · Invoice<br/>from approved time"]
    E --> F(["6 · Reports &amp;<br/>resource planning"])
    B -. "files · visibility" .-> DOC["Documents<br/>project · client · workspace"]

    classDef step fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef out fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef doc fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    class A,B,C,D,E step;
    class F out;
    class DOC doc;
```

</details>

### Identity, workspaces & access control
- **Multi-tenant workspaces** — every organization gets an isolated workspace; a strict tenancy boundary guarantees no data ever crosses between them.
- **Full authentication suite** — email/password sign-up and login, email verification, password reset, magic links, and secure, expiring team invitations (single and bulk).
- **Role-based access control** — Owner, Admin, Manager, and Member roles out of the box, plus fully custom roles. Access is expressed as fine-grained `resource:action` permissions, enforced on the server for every request.
- **Privilege-escalation protection** — a member can never grant a role or permission beyond what they themselves hold.

### Clients & contacts
- Company records with addresses, tax identifiers, default currency, payment terms, and billing cycles.
- Multiple contacts per client, with designated primary and billing contacts and independent contact lifecycle.
- Lifecycle management (active / archived) with history preserved.

### Projects & delivery
- Rich projects: client linkage, codes, descriptions, industry/tech-stack metadata, tags, budgets, and billing configuration.
- A governed **project lifecycle** (Planning → Active → On-Hold → Completed → Archived) with rules about which transitions are allowed and who may perform them.
- **Functional teams within a project** (e.g. Backend, Frontend, Mobile) with per-team pricing — either the sum of each member's rate, or a fixed lump sum for the whole team.
- Member assignment with project-specific roles and billing-rate overrides, plus bulk assignment.
- Tasks with status, priority, assignees, estimates, and due-date rules.

### Time, approvals & billing
- Time tracking against tasks and projects, with billable flags, categories, and per-entry notes.
- Weekly **timesheet submission and approval workflows** — submit, review, comment, approve or reject, with entries locked once approved.
- A **rate hierarchy** (user default → project default → per-member override) with historical rate preservation, plus cost-vs-bill tracking for margin analysis.
- **Invoicing** — generate from approved time, group and edit line items, apply tax and discounts, brand with the tenant's logo and template, deliver by email or shareable link, and track partial payments through a full invoice lifecycle.

### Resource planning & reporting
- A capacity timeline showing allocation vs. availability, with over-allocation warnings.
- Operational dashboards and enterprise reports — utilization, workload, profitability, and detailed audit trails — with multi-level filtering, grouping, and export.

### Documents
- Upload, version, preview, and manage files, scoped to a project, a client, or the whole workspace.
- **Granular visibility** — restricted, workspace-wide, or shared with a specific list of people — enforced consistently on every read and download.
- In-browser preview for common document formats, with full revision history and restore.

### Communication & configuration
- An in-app notification center plus email delivery for the events that matter (approvals due, budgets breached, invoices sent, payments received).
- **Bring-your-own email** — each workspace configures its own provider (SMTP, or a modern transactional API), with credentials encrypted at rest and a built-in connection test.
- Workspace branding, currency, and invoice templates.

### Integrations
- Native time-tracking integration with the team's issue tracker and calendar-driven time suggestions, so tracking happens where the work already lives.

---

## Architecture

A cleanly layered, API-first system built for correctness and multi-tenant safety.

![System architecture: a workspace-aware Next.js portal calls a FastAPI REST API over HTTPS with a JWT and an X-Tenant-Slug header; requests pass through tenant-context and auth middleware, then a thin Router (owns the transaction) into a Service (business logic) and a tenant-scoped Repository; the repository reads and writes PostgreSQL (shared schema, tenant_id discriminator) while the service writes documents and PDFs to S3-compatible object storage; JWT, RBAC, and email are cross-cutting concerns that govern the API tier](images/diagram-architecture.png)

<details>
<summary>Mermaid source (renders live on GitHub)</summary>

```mermaid
flowchart TB
    subgraph EDGE["🌐 Client · Edge"]
        U["User<br/>(browser)"]
        SPA["Next.js Portal<br/>React · TypeScript · Tailwind"]
    end
    subgraph APIT["⚙️ REST API · FastAPI"]
        MW["Tenant context &amp;<br/>auth middleware"]
        R["Router<br/>(HTTP · owns txn)"]
        S["Service<br/>(business logic)"]
        RP["Repository<br/>(tenant-scoped access)"]
    end
    subgraph DATA["🗄️ Application Data"]
        PG[("PostgreSQL<br/>shared schema · tenant_id")]
    end
    subgraph STORE["📦 Object Storage"]
        OS[("S3-compatible<br/>documents · PDFs")]
    end
    SEC["🔐 Cross-cutting<br/>JWT · RBAC · Email (Resend / SMTP)"]

    U --> SPA
    SPA -->|"HTTPS · JWT · X-Tenant-Slug"| MW
    MW --> R --> S --> RP
    RP --> PG
    S --> OS
    SEC -. governs .- APIT

    classDef edge fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef api fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef data fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef store fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    classDef sec fill:#f1f0f4,stroke:#6b6880,color:#33313f;
    class U,SPA edge;
    class MW,R,S,RP api;
    class PG data;
    class OS store;
    class SEC sec;

    style EDGE fill:#f4f8ff,stroke:#c3d6f5,color:#12336e;
    style APIT fill:#f8f5fd,stroke:#d9cdf0,color:#331a63;
    style DATA fill:#f2fbf8,stroke:#c2e9dd,color:#0a4f3c;
    style STORE fill:#fef9f2,stroke:#f0dcc0,color:#7a4708;
```

</details>

**Multi-tenancy.** A shared database with a `tenant_id` discriminator on every domain table. The tenant is derived from the request's workspace subdomain and carried as a header; the API cross-checks it against the caller's signed token and active membership on every request. Data-access is auto-scoped to the tenant at the repository layer, so cross-tenant reads are structurally impossible rather than merely discouraged.

**Layered backend.** HTTP routers stay thin and own the transaction boundary; services hold all business logic and are framework-agnostic; repositories own data access and are automatically tenant-scoped and soft-delete-aware. Typed domain exceptions convert to a single, uniform JSON error envelope.

**Frontend.** A workspace-aware Next.js portal (App Router, React, TypeScript, Tailwind) that renders strictly against the permissions the server reports — the UI shows only what the user may do, while the server remains the single source of truth for what they *can* do.

### Technology

| Layer | Stack |
|---|---|
| **Backend** | Python · FastAPI · Pydantic v2 · SQLAlchemy 2.0 · Alembic |
| **Database** | PostgreSQL (shared-schema multi-tenancy, UUIDv7 keys, soft deletes) |
| **Auth** | JWT access/refresh (HS256) · Argon2 password hashing · RBAC |
| **Storage** | S3-compatible object storage for documents & generated PDFs |
| **Email** | Pluggable providers (SMTP / transactional API) with encrypted credentials |
| **Frontend** | Next.js · React · TypeScript · Tailwind CSS · shadcn/ui |
| **Tooling** | uv · pytest · Playwright · Black · Ruff · Mypy · Bandit · pre-commit |
| **Delivery** | Dockerized services · CI gates · automated database migrations |

---

## Engineering highlights

- **Tenant isolation you can prove.** Isolation is enforced at the data-access layer and covered by a dedicated cross-tenant test suite: foreign-workspace reads return *not found* (no existence leak), workspace/identity mismatches are rejected, and a revoked member's still-valid token is locked out immediately.
- **Security-first RBAC.** Every authenticated endpoint declares the permission it requires; privilege-escalation guards prevent a user from ever granting access beyond their own.
- **Correctness under concurrency.** Money- and ownership-sensitive flows (billing rates, the last remaining owner of a workspace, uniqueness constraints) are guarded with explicit invariants and race-safe write paths.
- **Deep automated testing.** Layered unit and integration suites exercise happy paths, negative cases, boundary values, multi-tenant isolation, RBAC matrices, and data-integrity rules; end-to-end browser tests drive the real UI against a live backend.
- **Consistent quality gates.** Formatting, linting, type-checking, security scanning, and tests run on every change.

---

## Document Intelligence (RAG) — on the roadmap

The platform already knows a great deal about how a company works: its policies, its clients, its projects, the requirements each project was built against, and the solutions the teams actually delivered. Today that knowledge lives across documents, project records, and delivery history. The next step is to make it **conversational**.

**Document Intelligence** is a retrieval-augmented, chat-style assistant built into the platform. Any stakeholder — leadership, a project manager, a new engineer, a client-facing lead — can simply *ask*, in plain language, and get a grounded, cited answer drawn from the workspace's own knowledge:

- *"What's our remote-work and expense policy?"* → answered from company policy documents.
- *"What were the acceptance criteria for the payments module on Project X?"* → answered from that project's requirements.
- *"How did the team implement authentication on the client portal?"* → answered from the solution documentation the delivery team produced.

**How it works (intended design).**
- **Grounded in the workspace's own content** — company policies, project requirements, delivery/solution docs, and structured operational data are indexed into a vector store.
- **Retrieval-augmented generation** — a question retrieves the most relevant passages, which are handed to a language model to compose an answer *with citations back to the source documents*, so answers are traceable and trustworthy rather than hallucinated.
- **Tenancy- and permission-aware retrieval** — this is the critical difference from a generic chatbot: retrieval is filtered by the same multi-tenant and document-visibility rules the rest of the platform enforces. A user can only ever get answers from content they are already allowed to see, and no workspace's knowledge is ever reachable from another.
- **Role-shaped answers** — the assistant surfaces information appropriate to the asker: a manager querying delivery status, an engineer querying implementation detail, an executive querying policy — each served from the sources they're entitled to.

**Why it matters.** It turns the operational data a services company already accumulates into an institutional memory that answers questions in seconds — onboarding new hires faster, keeping delivery aligned to requirements, and making company knowledge searchable by conversation instead of by folder.

*This capability is in the design pipeline and slated for a dedicated design cycle.*

---

## At a glance

A production-grade, multi-tenant SaaS spanning identity, project delivery, time, billing, resource planning, reporting, and document management — engineered with a strict tenancy boundary, server-enforced RBAC, a cleanly layered API, and a deep automated test suite — and evolving toward a permission-aware, RAG-based knowledge assistant.

---

> *Notice: This case study has been modified to comply with confidentiality agreements. The resulting framework and artifacts remain the strict intellectual property of Rohail K. Malhi and may not be duplicated or repurposed.*
