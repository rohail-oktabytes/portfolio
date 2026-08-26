# Secure Healthcare Data-Access Portal Platform

## ⚠️ Proprietary Work & Copyright Notice

This case study represents proprietary methodologies and NDA-compliant frameworks.

**This project is NOT open-source.**

© 2026 Rohail K. Malhi. All rights reserved.

You are welcome to read and review these materials to understand my professional capabilities. However, you are **strictly prohibited** from copying, adapting, or utilizing these artifacts, structures, or content in any form. See [LICENSE](LICENSE).

---

**A secure, multi-tenant web portal platform that gives health-plan staff, providers, and third-party developers self-service access to member and clinical data. Two Angular single-page portals — an internal admin console and an external developer/sandbox portal — sit behind an OIDC identity provider and a *private*, authorizer-gated API, with a standards-based FHIR clinical-data explorer, role-based administration, OAuth application onboarding, and two analytics suites. The whole estate is provisioned as reusable infrastructure-as-code and shipped through a cross-cloud CI/CD pipeline, on a foundation of shared, versioned Angular design-system and authentication libraries.**

> **Confidentiality note.** This is a sanitized portfolio overview. The client identity, product names, brand assets, internal hostnames, cloud account identifiers, and proprietary source are withheld under NDA. Everything here describes engineering capabilities and architecture at a level safe for public sharing. Healthcare data standards referenced (FHIR, HITRUST, OIDC) are public industry standards and reveal no client data; internal identifiers, credentials, and any confidential content are intentionally omitted.

---

## Challenge

A healthcare-data organization needed to expose sensitive **member and clinical data** — protected health information (PHI) — to multiple audiences through the web, without compromising security, compliance, or trust:

- **Different audiences, one data estate.** Internal health-plan staff needed an administrative console; external partners and third-party developers needed a self-service portal to register applications and explore APIs against a sandbox. Building and operating these as separate, divergent codebases would have been slow and inconsistent.
- **PHI raises the bar on everything.** Every screen, API call, and audit event touches regulated healthcare data. Access had to be strongly authenticated, finely authorized per user and per tenant, and auditable end-to-end — with the API itself unreachable from the public internet.
- **Clinical data is only useful if it speaks a standard.** Consumers expected **HL7 FHIR** resources — patients, conditions, encounters, medications, labs, procedures, care plans — not a bespoke schema, so the portal needed a genuine FHIR data explorer rather than ad-hoc screens.
- **Many environments, many tenants, one artifact.** The same application had to run across development, QA, UAT, and production — split across separate cloud accounts — and serve multiple customer tenants, ideally from a single, immutable build rather than a per-environment rebuild.
- **Delivery had to be repeatable and governed.** Infrastructure and releases needed to be codified, promoted through approval gates, and reproducible — not hand-clicked in a console.

They needed a portal platform that was **secure and compliant by construction, standards-based, multi-tenant, and delivered as code** — one that could scale from one internal console to a public developer ecosystem without rework.

---

## Solution

A **two-portal platform over a shared front-end foundation**, engineered so that security, multi-tenancy, and repeatable delivery are structural rather than bolted on.

### Two portals, one build artifact
- **Admin portal** — an internal console for health-plan/provider staff: identity administration, the FHIR clinical-data explorer, and analytics.
- **Developer & sandbox portal** — an external surface for self-service registration and API testing against a sandbox tenant.
- **Runtime configuration, not build-time.** A single immutable Angular build serves *every* environment. On startup the app derives its **stage and tenant from the request subdomain**, then fetches its backend service URLs and OIDC client configuration from a config API — so one artifact promotes unchanged from dev to production, and sandbox vs. production is a runtime distinction.

### Security & identity by construction
- **OIDC with PKCE.** Authentication runs through an enterprise identity provider using the authorization-code + **PKCE** flow — no client secret in the browser. Access and ID tokens are JWKS-signed.
- **Claim-based RBAC.** A fine-grained **permission claim** in the signed token drives both route guards (every feature route is gated) and menu rendering — the UI only ever shows what a user is authorized to use. Silent token refresh and an idle-timeout warning manage the session.
- **Private, authorizer-gated API.** The backend API gateway is **private** — reachable only through an in-VPC endpoint, never the public internet. A **token Lambda authorizer** independently re-verifies every request's JWT (signature, issuer, audience) server-side, so client-side checks are never trusted alone.
- **No secrets in code.** All identity and environment configuration is injected at deploy time via secrets-manager → infrastructure dynamic references; nothing sensitive is committed to source.
- **Compliance-grade auditing.** PHI interactions are forwarded to a **HITRUST-aligned** compliance-audit trail.

### A standards-based FHIR clinical-data explorer
- **Real FHIR R4, tenant-scoped.** Patient search and a consolidated patient profile open onto **15 HL7 FHIR R4 resource types** — conditions, encounters, medications, allergies, diagnostic reports, procedures, immunizations, care plans, care teams, goals, devices, documents, and more — each with paging and code/date search, every query scoped to the signed-in customer tenant.

### Self-service administration & analytics
- **Identity & access administration.** Full RBAC — users, roles, and permission sets — plus **OAuth client-application registration** (redirect URIs, scopes) so partners can onboard their own integrations.
- **Two analytics suites.** An **API-usage** dashboard suite (status-code, response-time, and data-volume trends, sliced by API and by resource) gives developers operational insight; a **member-demographics** suite (enrollment by age, gender, race, county, and coverage type) gives the business population-health views — both built on a reusable filter-and-chart component pattern.

### Delivered as code, on a shared foundation
- **Reusable infrastructure-as-code.** A nested-stack pattern provisions the whole hosting estate — private origin buckets, CDN distributions, web-application-firewall ACLs, access logs — from a handful of parameterized, reusable templates, and publishes bucket names and distribution IDs to a parameter store for the pipeline to consume.
- **Cross-cloud CI/CD.** The delivery pipeline builds the SPA once and promotes it through **gated environments** (dev → QA → UAT → production, each behind manual approval) into the cloud — syncing to the origin bucket and invalidating the CDN — and separately deploys the serverless API stack per stage.
- **Shared Angular libraries.** A published **design-system library** (40+ UI components, dialogs, config-driven tables, theming) and a companion **authentication library** (session lifecycle, token, and guard services) are packaged and reused across applications — so both portals share one consistent, maintained foundation, and the frontend build is portable across CDN, container, Kubernetes, and PaaS targets.

---

## Architecture

Two Angular single-page portals are delivered from private object storage through a CDN (with a web-application firewall and SPA deep-link routing). Authenticated calls go to a **private** API gateway, gated by a token Lambda authorizer that verifies OIDC-issued JWTs; VPC-bound serverless functions serve configuration and proxy audit/telemetry, with all secrets injected at deploy time. Behind them sit the systems of record — a member/access REST API, a FHIR R4 clinical API, and a compliance-audit service.

![System architecture](images/diagram-architecture.png)

<details>
<summary>Diagram source (Mermaid)</summary>

```mermaid
flowchart TB
    subgraph CLIENT["Web clients - Angular SPAs (one build, every stage)"]
        direction LR
        ADM["Admin portal<br/>staff console"]:::client ~~~ DEV["Developer &amp; sandbox<br/>portal"]:::client ~~~ BOOT["Runtime config boot<br/>stage + tenant from subdomain"]:::client
    end
    subgraph EDGE["Edge &amp; delivery"]
        direction LR
        CDN["CDN distribution<br/>TLS - SPA routing"]:::data ~~~ WAF["Web application<br/>firewall"]:::sec ~~~ S3[("Private object storage<br/>CDN-identity only")]:::data
    end
    subgraph IDE["Identity &amp; API edge"]
        direction LR
        OIDC["OIDC identity provider<br/>PKCE - JWKS-signed tokens"]:::amber ~~~ APIGW["Private API gateway<br/>VPC-endpoint only"]:::api ~~~ AUTHZ["Token Lambda authorizer<br/>verify JWT"]:::api
    end
    subgraph COMP["Serverless compute - VPC-bound"]
        direction LR
        CFG["Identity + environment<br/>config"]:::api ~~~ AUD["Compliance-audit<br/>proxy"]:::api ~~~ LOG["Telemetry sink"]:::api
    end
    SEC["Secrets Manager -> IaC dynamic references (no secrets in code)"]:::amber
    subgraph DATA["Backend data plane - systems of record"]
        direction LR
        REST["Member &amp; access<br/>REST API"]:::data ~~~ FHIR["FHIR R4<br/>clinical API"]:::data ~~~ AUDIT["Compliance audit<br/>(HITRUST-aligned)"]:::sec
    end
    CLIENT --> EDGE --> IDE --> COMP
    COMP --> SEC
    COMP --> DATA
    classDef client fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef api fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef data fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef amber fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    classDef sec fill:#f1f0f4,stroke:#6b6880,color:#33313f;
    style CLIENT fill:#f4f8ff,stroke:#c3d6f5;
    style EDGE fill:#f2fbf8,stroke:#c2e9dd;
    style IDE fill:#f8f5fd,stroke:#d9cdf0;
    style COMP fill:#f8f5fd,stroke:#d9cdf0;
    style DATA fill:#f2fbf8,stroke:#c2e9dd;
```

</details>

### Authentication & runtime configuration

One static build boots, resolves its environment and identity config from the subdomain, authenticates via OIDC + PKCE, then lets a signed permission claim drive the client UI — while the private API gateway re-verifies every token server-side.

![Authentication and runtime configuration flow](images/diagram-auth-flow.png)

<details>
<summary>Diagram source (Mermaid)</summary>

```mermaid
flowchart TB
    A(["Load SPA from CDN<br/>same artifact, every stage"]):::edge
    B["Derive stage &amp; tenant<br/>from subdomain"]:::client
    C["Fetch runtime config<br/>env URLs + OIDC client config"]:::client
    D["Login with PKCE<br/>code exchanged for tokens"]:::amber
    E["Decode claims<br/>tenant - roles - userPermissions"]:::amber
    F["Route guards + RBAC menus<br/>silent refresh + idle timeout"]:::api
    G["Bearer interceptor<br/>attach access token"]:::api
    H{"Private API gateway<br/>+ Lambda authorizer<br/>re-verify JWT"}:::sec
    I(["Serve from backend<br/>data plane"]):::done
    A --> B --> C --> D --> E --> F --> G --> H
    H -->|valid token| I
    classDef edge fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef client fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef api fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef amber fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    classDef sec fill:#f1f0f4,stroke:#6b6880,color:#33313f;
    classDef done fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
```

</details>

### Product capability map

A single tenant-scoped console spanning identity administration, FHIR clinical-data exploration, and two analytics suites.

![Product capability map](images/diagram-capabilities.png)

<details>
<summary>Diagram source (Mermaid)</summary>

```mermaid
flowchart TB
    T["Every capability scoped to the signed-in customer tenant"]:::sec
    subgraph IAM["Identity &amp; access administration"]
        direction LR
        U["User management"]:::api ~~~ R["Roles &amp; permissions<br/>(RBAC)"]:::api ~~~ O["OAuth app<br/>registration"]:::api
    end
    subgraph CLIN["FHIR clinical-data explorer"]
        direction LR
        P["Patient search<br/>&amp; profile"]:::data ~~~ FR["15 FHIR R4<br/>resource types"]:::data ~~~ PG["Paged, searchable<br/>reads"]:::data
    end
    subgraph AN["Operational &amp; population analytics"]
        direction LR
        USG["API usage<br/>analytics"]:::client ~~~ DEMO["Member<br/>demographics"]:::client ~~~ CH["Interactive<br/>charts"]:::client
    end
    T --> IAM --> CLIN --> AN
    classDef client fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef api fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef data fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef sec fill:#f1f0f4,stroke:#6b6880,color:#33313f;
    style IAM fill:#f8f5fd,stroke:#d9cdf0;
    style CLIN fill:#f2fbf8,stroke:#c2e9dd;
    style AN fill:#f4f8ff,stroke:#c3d6f5;
```

</details>

### Delivery & platform engineering

Reusable nested-stack infrastructure-as-code provisions the hosting estate and publishes service-discovery parameters; a cross-cloud pipeline builds once and promotes through gated environments — all on a shared, versioned Angular design-system and auth library.

![Delivery and platform engineering](images/diagram-delivery.png)

<details>
<summary>Diagram source (Mermaid)</summary>

```mermaid
flowchart TB
    subgraph IAC["Reusable infrastructure as code"]
        direction LR
        BS["Bootstrap<br/>IAM + template bucket"]:::sec ~~~ MOD["Reusable modules<br/>storage - CDN - params"]:::data ~~~ EST["Provisioned estate<br/>buckets - CDN - WAF"]:::data ~~~ SSM["Service-discovery<br/>params (SSM)"]:::data
    end
    subgraph PIPE["Cross-cloud delivery pipeline"]
        direction LR
        BLD["Build once<br/>optimized SPA artifact"]:::amber ~~~ PRM["Promote with gates<br/>dev-qa-uat-prod"]:::amber ~~~ DSPA["Deploy SPA<br/>sync + CDN invalidate"]:::amber ~~~ DSRV["Deploy serverless<br/>per stage"]:::amber
    end
    subgraph LIB["Shared Angular library workspace"]
        direction LR
        DS["Design-system<br/>library"]:::api ~~~ AL["Auth library<br/>session + guards"]:::api ~~~ PORT["Write-once,<br/>deploy-anywhere"]:::client
    end
    IAC --> PIPE --> LIB
    classDef client fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef api fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef data fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef amber fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    classDef sec fill:#f1f0f4,stroke:#6b6880,color:#33313f;
    style IAC fill:#f2fbf8,stroke:#c2e9dd;
    style PIPE fill:#fef9f2,stroke:#f0dcc0;
    style LIB fill:#f8f5fd,stroke:#d9cdf0;
```

</details>

### Technology

| Layer | Stack |
|---|---|
| **Frontend** | Angular SPA · Angular Material + CDK + Flex-Layout · RxJS · lazy-loaded feature modules · Chart.js analytics · runtime (subdomain-driven) environment configuration |
| **Shared libraries** | Published design-system component library (40+ components, dialogs, config-driven tables, theming) + companion OIDC authentication library · `ng-packagr` · multi-project workspace |
| **Identity & authorization** | OIDC identity provider (Okta) · authorization-code + PKCE · JWKS-signed JWTs · claim-based RBAC (route guards + menu gating) · silent session refresh + idle-timeout |
| **API edge** | Private API Gateway (VPC-endpoint only) · token/request Lambda authorizer (JWT signature / issuer / audience verification) · CORS · Bearer-token HTTP interceptor |
| **Serverless compute** | AWS Lambda (Node.js) in-VPC · X-Ray tracing · identity-config, environment-discovery, compliance-audit-proxy & telemetry functions · AWS SAM |
| **Healthcare standards** | HL7 **FHIR R4** clinical resources (15 resource types) · **HITRUST**-aligned compliance auditing · PHI-scoped, per-tenant access |
| **Edge & hosting** | CloudFront (TLS/ACM, HTTP/2, geo-restriction, SPA 403/404 → index.html) · WAFv2 web ACLs · private, versioned, SSE-encrypted S3 origins via origin-access identity |
| **Config & secrets** | Secrets Manager → infrastructure dynamic references (zero secrets in code) · runtime config API for per-stage service discovery |
| **Infrastructure as code** | CloudFormation nested-stack pattern · reusable storage / CDN / parameter modules · per-environment parameter files · SSM Parameter Store service discovery |
| **CI/CD** | Cross-cloud pipeline (Azure DevOps → AWS) · build-once / promote-through-gated-environments (dev → QA → UAT → prod) · S3 sync + CloudFront invalidation · SAM deploy · multi-account (non-prod / prod) |
| **Quality** | Karma + Jasmine unit tests with coverage · Protractor e2e · TSLint + codelyzer · Prettier · Husky pre-commit lint |

---

## Engineering highlights

- **One immutable build, every environment and tenant.** The SPA resolves its stage, tenant, backend URLs, and identity config *at runtime* from the request subdomain and a config API — so a single artifact promotes unchanged from dev to production, and sandbox vs. production is a runtime distinction, not a rebuild.
- **Defense-in-depth authorization.** A signed permission claim drives client-side route guards and menu rendering, *and* a private, VPC-only API gateway backed by a token Lambda authorizer independently re-verifies every JWT server-side — the client is never the sole gatekeeper of PHI.
- **Private by default.** The API is unreachable from the public internet (VPC-endpoint only); the SPA origins are private buckets exposed solely through the CDN's origin-access identity; secrets are injected at deploy time, never committed.
- **Standards-based clinical data.** A genuine FHIR R4 explorer over 15 resource types — with paging and code/date search, tenant-scoped — rather than bespoke, one-off clinical screens.
- **A shared front-end foundation.** A versioned design-system library and a companion authentication library are packaged and consumed across applications, giving both portals one consistent, maintained base — and making the auth/session lifecycle reusable rather than re-implemented per app.
- **Compliance and observability built in.** HITRUST-aligned audit forwarding for PHI interactions, X-Ray tracing on serverless functions, and a browser-to-cloud telemetry bridge make behavior traceable.
- **Repeatable, governed delivery.** Reusable nested-stack IaC provisions the entire hosting estate and hands bucket/distribution identifiers to the pipeline through a parameter store; a cross-cloud pipeline builds once and promotes through manual-approval gates across separate cloud accounts.
- **Portable frontend.** The same optimized Angular build is targetable at a CDN, a container (Nginx), Kubernetes, or a PaaS static host — hosting is a deployment choice, not a code dependency.

---

## At a glance

A secure, multi-tenant **healthcare data-access portal platform** delivering two Angular single-page portals — an internal admin console and an external developer/sandbox portal — from one immutable build that resolves its environment, tenant, and identity configuration at runtime. Authentication runs on **OIDC + PKCE** with a signed permission claim driving **claim-based RBAC** across route guards and menus, while a **private, VPC-only API gateway** and a **token Lambda authorizer** re-verify every request server-side. The product spans self-service identity administration (users, roles, OAuth app registration), a standards-based **HL7 FHIR R4** clinical-data explorer across 15 resource types, and two analytics suites (API-usage and member demographics) — all tenant-scoped, PHI-aware, and **HITRUST-aligned** for audit. The entire estate is provisioned as reusable **nested-stack infrastructure-as-code** (private S3 origins, CloudFront, WAF) and shipped by a **cross-cloud CI/CD** pipeline that builds once and promotes through gated dev → QA → UAT → prod environments across multiple cloud accounts — on a foundation of shared, versioned Angular **design-system** and **authentication** libraries.

---

> *Notice: This case study has been modified to comply with confidentiality agreements. The resulting framework and artifacts remain the strict intellectual property of Rohail K. Malhi and may not be duplicated or repurposed.*
