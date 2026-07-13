# Portfolio — Data Architecture & Engineering

Engineer with a hybrid background (electronics & automation + master's degree + data) focused on
**data architecture** and **analytics engineering**. This portfolio collects real projects
—recreated with synthetic data and no confidential information— documented as **architecture
decisions**: the problem, the constraints, the alternatives evaluated, and the rationale behind each
technical choice.

> Each case reads in ~5 minutes and follows the same structure (see [ADR_TEMPLATE.md](ADR_TEMPLATE.md)).

---

## Cases

| # | Case | Domain | Main stack |
|---|------|--------|------------|
| 01 | [Social media analytics data lake](cases/01-social-media-data-lake/) | Data architecture · medallion | S3 · Apache Iceberg · Glue · Athena · Step Functions · Terraform |
| 02 | Agentic analytics over the data warehouse *(WIP)* | AI over data · text-to-SQL + semantic search | Snowflake Cortex (Analyst / Search) · semantic layer · Vue 3 |
| 03 | Cloud architecture & IaC at scale *(WIP)* | Cloud / end-to-end platform | ECS Fargate · ALB · RDS · CloudFront · Terraform |
| 04 | Multi-platform ingestion / ETL *(WIP)* | Data engineering | Python · APIs · retries · data quality · AWS |

---

## How it's organized

```
portfolio/
├── README.md              ← this index
├── ADR_TEMPLATE.md        ← architecture-decision template
├── cases/
│   └── 01-.../
│       ├── README.md      ← the case, ADR style
│       └── diagrams/      ← architecture diagrams (SVG)
└── docs/                  ← website source (GitHub Pages)
```

## Approach per case

1. **Problem** — what was needed and why it mattered.
2. **Context & constraints** — what shaped the design.
3. **Architecture** — diagram and end-to-end flow.
4. **Technology choices & rationale** — chose X / rejected Y.
5. **Cost & scalability**.
6. **Results / impact**.
7. **Possible improvements**.

---

## Confidentiality note

All cases are based on real projects but **recreated with generic domains and synthetic data**. They
contain no real names, production identifiers, credentials or internal information of any
organization.
