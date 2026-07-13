# Self-Service Reporting & Analytics Platform

## ⚠️ Proprietary Work & Copyright Notice

This case study represents proprietary methodologies and NDA-compliant frameworks.

**This project is NOT open-source.**

© 2026 Rohail K. Malhi. All rights reserved.

You are welcome to read and review these materials to understand my professional capabilities. However, you are **strictly prohibited** from copying, adapting, or utilizing these artifacts, structures, or content in any form. See [LICENSE](LICENSE).

---

**A cloud-native, self-service reporting product that lets non-technical business users pull data from any source, merge and shape it with drag-and-drop, and publish pixel-perfect reports — without writing a line of SQL.**

> **Confidentiality note.** This is a sanitized portfolio overview. The product name, client identity, proprietary business rules, and internal source are withheld under NDA. Everything here describes capabilities and engineering approach at a level safe for public sharing. Screenshots are from a demo/test environment with client and product logos redacted (see `images/REDACTION.md`).

> 📄 **Client-facing case study (C-S-R):** [`self_service_reporting_platform_case_study.pdf`](self_service_reporting_platform_case_study.pdf) — a polished, shareable PDF with Challenge → Solution → Result, embedded screenshots (logos redacted).

---

## The problem

Large organizations sit on data spread across operational databases, warehouses, and flat files. The people who actually need reports — administrators, analysts, department heads — usually *can't* write SQL, so every report request becomes a ticket to an overworked technical team. Turnaround is measured in days, and the backlog never clears.

The platform removes the technical team from the critical path. A business user connects to their data sources once, then builds and merges datasets and designs formatted reports entirely through a visual interface. What used to be a SQL-and-ETL task for engineers becomes a self-service action for the person who needs the answer.

---

## Screenshots

> Demo/test environment. See `images/REDACTION.md` for items to mask before external sharing.

| | |
|---|---|
| **Data Exchanger** — drag-and-drop dataset merge & join | ![Data Exchanger](images/03-data-exchanger-merge.png) |
| **Report Builder** — group-by and aggregate functions, no SQL | ![Report Builder](images/04-report-builder-aggregates.png) |
| **Template Builder** — visual, pixel-perfect report designer | ![Template Builder](images/01-report-template-builder.png) |
| **Data binding** — bind live data fields into the template | ![Data binding](images/02-template-data-binding.png) |
| **Sections & layout** — repeating sections, columns, print rules | ![Sections](images/05-template-sections.png) |

---

## What it does

The whole product is one guided journey — from raw data to a finished, formatted report — with no SQL at any step. Heavy processing is offloaded to the data lake so the interface stays responsive.

![Self-service reporting workflow: Connect → Build Collection → Data Exchanger → Report Builder → Template Builder → Export, with heavy compute offloaded to a Glue + Spark data lake](images/diagram-workflow.png)

<details>
<summary>Mermaid source (renders live on GitHub)</summary>

```mermaid
flowchart LR
    A["1 · Connect<br/>data source"] --> B["2 · Build<br/>Collection"]
    B --> C["3 · Data Exchanger<br/>merge · join · filter"]
    C --> D["4 · Report Builder<br/>group · aggregate"]
    D --> E["5 · Template Builder<br/>pixel-perfect layout"]
    E --> F(["6 · Export<br/>CSV · print-ready PDF"])
    C -. "heavy compute" .-> LK["Glue + Spark<br/>data lake"]
    LK -. results .-> D

    classDef step fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef out fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef lk fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    class A,B,C,D,E step;
    class F out;
    class LK lk;
```

</details>

### Connect to any data source
- Save reusable connection profiles for **relational databases (JDBC)**, **CSV/flat files**, and **third-party reporting data blocks**.
- Connection details are stored once and shared across the organization, so end users pick a source instead of re-entering credentials.

### Collections — reusable datasets
- A **Collection** packages a dataset together with its metadata (field names, types, descriptions), so a curated dataset can be reused across many reports.
- Collections become the building blocks that non-technical users combine, rather than raw tables they'd have to understand.

### Data Exchanger — visual merge & shape
- **Drag-and-drop join** of two or more collections onto a canvas, with linking done visually instead of by hand-writing joins.
- Apply **filters, sorting, grouping, and aggregate functions** (COUNT, SUM, etc.) and derive **calculated columns** — all through a guided UI.
- Preview the resulting dataset before committing it to a report.

### Report & Template Builder
- Turn a shaped dataset into a **Report** — a saved, re-runnable output.
- A **pixel-perfect Template Builder** for formatted output: drag in headers, footers, images, text, tables, shapes, and repeating sections; position elements precisely; and **bind live data fields** (including global fields like date and page number) into the layout.
- Sections support repeating vs. non-repeating rendering, multi-column layouts, and print rules — the foundation for production documents such as transcripts and statements.
- Export to **CSV** and **print-ready PDF**.

### Governance, access & multi-tenancy
- **Enterprise SSO / directory integration** for authentication; users must be explicitly authorized for the resources they access.
- **Sites** provide multi-tenant isolation, letting an onboarded organization partition departments, campuses, or business units.
- **Groups** grant read-only, shared access to collections and reports across sets of users.
- Fine-grained **roles and permissions** on top of user accounts.

### Notifications
- Event- and action-driven notifications keep users informed of long-running jobs and report/collection changes.

---

## Architecture

A serverless, event-driven system on AWS: a React single-page app on the edge, a serverless API tier, and a Spark-based data-lake pipeline for heavy data processing.

![System architecture: a React SPA on the edge (CloudFront, S3, Cognito) calls a serverless API (API Gateway, Lambda) that reads/writes PostgreSQL metadata and drives a Glue + Spark + Athena data-lake pipeline over S3; KMS/IAM/SES/SNS/CloudWatch/VPC/CodePipeline as cross-cutting concerns](images/diagram-architecture.png)

<details>
<summary>Mermaid source (renders live on GitHub)</summary>

```mermaid
flowchart TB
    subgraph EDGE["🌐 Client · Edge"]
        U["Business User<br/>(browser)"]
        SPA["React SPA<br/>Redux · Redux-Saga"]
    end
    subgraph APIT["⚙️ Serverless API"]
        GW["API Gateway"]
        L["Lambda<br/>Node.js · Python"]
        AZ["AuthZ &amp;<br/>permission checks"]
    end
    subgraph APPDATA["🗄️ Application Data"]
        PG[("RDS / Aurora<br/>PostgreSQL<br/>metadata · ACLs")]
    end
    subgraph LAKE["📊 Data-Lake Pipeline"]
        EXT[/"External sources<br/>JDBC · CSV · data blocks"/]
        GL["AWS Glue<br/>Catalog · Jobs · Crawler"]
        SP["Spark / PySpark"]
        S3[("S3")]
        ATH["Athena"]
    end
    SEC["🔐 Cross-cutting<br/>KMS · IAM · SES/SNS · CloudWatch · VPC · CodePipeline"]

    U --> SPA
    SPA -->|"HTTPS · Cognito"| GW
    GW --> L --> AZ
    AZ --> PG
    L --> GL
    EXT --> GL --> SP --> S3
    S3 --> ATH --> L
    SEC -. governs .- APIT

    classDef edge fill:#e8f0fe,stroke:#3b6fd4,color:#12336e;
    classDef api fill:#efeafb,stroke:#5b3fa0,color:#331a63;
    classDef data fill:#e6f6f1,stroke:#0e9f7a,color:#0a4f3c;
    classDef lake fill:#fdf0e2,stroke:#d3852a,color:#7a4708;
    classDef sec fill:#f1f0f4,stroke:#6b6880,color:#33313f;
    class U,SPA edge;
    class GW,L,AZ api;
    class PG data;
    class EXT,GL,SP,S3,ATH lake;
    class SEC sec;

    style EDGE fill:#f4f8ff,stroke:#c3d6f5,color:#12336e;
    style APIT fill:#f8f5fd,stroke:#d9cdf0,color:#331a63;
    style APPDATA fill:#f2fbf8,stroke:#c2e9dd,color:#0a4f3c;
    style LAKE fill:#fef9f2,stroke:#f0dcc0,color:#7a4708;
```

</details>

**Serverless-first.** The compute tier is AWS Lambda behind API Gateway, packaged and deployed with the Serverless Framework — no servers to manage, and cost/scaling that follow actual usage.

**Two data planes.** Application metadata (users, sites, collections, report definitions, permissions) lives in managed PostgreSQL (RDS / Aurora). The *actual* reporting data flows through a **data-lake pipeline** — AWS Glue for cataloging and ETL jobs, Spark/PySpark for distributed transformation on S3, and Athena as the query layer — so large dataset merges and aggregations run at scale instead of inside the request path.

**Edge-hosted SPA.** The React app is served from S3 via CloudFront; a managed identity provider handles authentication and directory federation.

**Secure by construction.** Encryption via KMS, least-privilege IAM roles per function, and VPC isolation for data-tier access. (A dedicated post-MVP effort benchmarked field-level encryption approaches for sensitive report data.)

### Technology

| Layer | Stack |
|---|---|
| **Frontend** | React · Redux · Redux-Saga · React Final Form · Reactstrap |
| **Backend** | Node.js · Sequelize · Python · Serverless Framework |
| **Data pipeline** | AWS Glue (Catalog · Jobs · Crawler) · Spark / PySpark · Athena · Boto3 |
| **Database** | Amazon RDS & Aurora (PostgreSQL) |
| **Infra / platform** | Lambda · API Gateway · S3 · CloudFront · Cognito · KMS · IAM · SES · SNS · SSM · VPC · Service Catalog |
| **Observability** | CloudWatch (metrics · logs · dashboards) |
| **CI/CD** | CodeBuild · CodePipeline · Git-based workflow |
| **QA** | Python · Pytest · Boto3 · Postman · Allure reporting |
| **Reporting output** | Server-side PDF generation (template-driven), CSV export |

---

## Engineering highlights

- **No-SQL analytics for non-engineers.** The hard part isn't drawing a UI — it's translating drag-and-drop joins, filters, group-bys, and calculated columns into correct, performant data operations against a Spark/Athena backend, and giving users a faithful preview before they commit.
- **Scale outside the request path.** Heavy dataset merges and aggregations are pushed into a Glue/Spark data-lake pipeline rather than executed synchronously, keeping the interactive app responsive while still handling large volumes.
- **Pixel-perfect, data-bound documents.** A visual template engine binds live data fields into positioned layouts with repeating sections and print rules — the basis for production documents (e.g. transcripts and statements) rendered server-side to PDF.
- **Multi-tenant governance.** Sites, groups, roles, and per-request authorization combine to isolate tenants and control exactly who can see which collections and reports.
- **Fully serverless delivery.** Serverless Framework + CodePipeline/CodeBuild give repeatable, infrastructure-as-code deployments with usage-based scaling and no server fleet to maintain.
- **Test discipline at delivery scale.** A Python/Pytest automation suite (with API testing via Postman and Allure reporting) backed a multi-sprint delivery that closed out hundreds of tracked defects across frontend, backend, and data-lake layers before release.

---

## At a glance

A serverless AWS platform that turns multi-source data into self-service reporting for non-technical users — visual dataset merging (Data Exchanger), no-SQL grouping and aggregation (Report Builder), and a pixel-perfect, data-bound Template Builder that outputs CSV and print-ready PDF — backed by a Glue/Spark/Athena data lake, multi-tenant governance, and end-to-end CI/CD.

---

> *Notice: This case study has been modified to comply with confidentiality agreements. The resulting framework and artifacts remain the strict intellectual property of Rohail K. Malhi and may not be duplicated or repurposed.*
