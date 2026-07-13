# Cloud architecture & IaC for containerized web apps at scale

**Role:** Data / Solutions Architect · **Year:** 2025–2026 · **Status:** Production (national-scale event)

> **Confidentiality note:** case based on a real project, recreated with a generic domain and
> synthetic values. It contains no account IDs, VPC IDs, ARNs, real domains or internal information.

**One-line summary:** A reusable, environment-parametrized **Terraform** platform that deploys
containerized web applications on **AWS ECS Fargate** behind a shared **ALB**, with a **CloudFront +
S3** frontend, **RDS PostgreSQL** (and optional DocumentDB), secrets, DNS/TLS and a WAF — proven on a
high-traffic, national-scale event and reused as the blueprint for a second production app.

---

## 1. Problem

The organization needed to ship several web applications to production quickly and repeatably — one
of them for a **national-scale event** with large, spiky traffic — without hand-clicking
infrastructure in the AWS console for each one. Each app had the same backbone (SPA + containerized
API + relational DB) but slightly different needs (extra services, media storage, SSR). The goal was
a **single infrastructure blueprint** that any app could instantiate per environment (sandbox / QA /
prod) with a config file.

The architectural challenge was designing infrastructure that is **generic enough to reuse** yet
**specific enough to run real apps**, and that stays cheap when idle but absorbs national-scale
spikes.

## 2. Context & constraints

- Multiple apps, one small platform team → infra must be **reusable and reproducible**, not bespoke.
- One workload faced **national-scale, bursty traffic** (a live event) → must scale out and back in.
- Cost discipline: no always-on over-provisioning; pay for what's used.
- Security baseline: private database, secrets never in code, HTTPS everywhere, WAF at the edge.
- Multiple environments (sandbox / QA / prod) that must be **identical in shape**, differing only in sizing.
- Mixed frontends: mostly static SPAs, with room for a server-rendered service on the same balancer.

## 3. Proposed architecture

A modular Terraform codebase where each module owns one concern, composed per environment from a
`.tfvars` file. Traffic splits between a CDN-fronted static SPA and a containerized API.

![Architecture](diagrams/architecture.svg)

**Request flow:**

1. Users hit **CloudFront**. Static assets come from a private **S3** bucket (origin locked down with
   OAC). A CloudFront behavior for `/api/*` forwards API calls to the ALB, so the SPA and API share
   one domain.
2. The **public ALB** (HTTP→HTTPS redirect) is **shared**: host- and path-based rules route to
   different **target groups**, so several apps/services live behind one balancer.
3. The API runs as **ECS Fargate** tasks in **private subnets** (no public IP). Images come from
   **ECR**; configuration and secrets are injected from **Secrets Manager**.
4. **RDS PostgreSQL** is private — ingress on `5432` only from the ECS task security groups (plus
   optional admin CIDRs via VPN). A document store (**DocumentDB**) is available for apps that need it.
5. **Route 53 + ACM** issue certificates and records for the SPA domain and `api.<domain>`; a **WAF**
   sits at the edge. A **bastion via SSM** gives break-glass DB access without exposing SSH.
6. Additional services (e.g. an SSR container) attach to the **same ALB** with their own target group
   and host rule — no new balancer required.

**On reusability:** every resource name derives from `project_name` + `environment`, so the same code
produces isolated sandbox/QA/prod stacks. Adding a second app = a new `.tfvars` + (optionally) an
entry in the `additional_ecs_services` map. Remote state lives in S3 per environment.

**Concrete app instance:** the blueprint runs a real app — an Angular SPA + FastAPI API on
PostgreSQL, with **Keycloak** as the identity provider (SPA gets a JWT, the API validates it against
Keycloak's JWKS), **SES** for transactional email, **S3 presigned URLs** for media, and **k6** load
tests to validate it before the traffic peak.

## 4. Technology choices & rationale

| Decision | Chosen | Rejected | Why |
|---|---|---|---|
| Compute | **ECS Fargate** | EKS / self-managed EC2 | Containers without managing servers or a Kubernetes control plane; right-sized for a small team. |
| Edge / frontend | **CloudFront + S3 (OAC)** | Serve SPA from the container | Cheap, cached, global; origin stays private; frontend deploys are just an S3 sync + cache invalidation. |
| Ingress | **One shared ALB, host/path rules** | One ALB per app | Fewer moving parts and lower cost; multiple apps/services behind a single balancer. |
| Relational DB | **RDS PostgreSQL (private)** | DB on a container / public DB | Managed backups/patching; no public exposure; access limited to task security groups. |
| Config & secrets | **Secrets Manager + env** | Secrets in image / repo | Secrets never in code; rotated centrally; injected at task start. |
| IaC | **Terraform, modular, per-env tfvars** | Console / copy-paste stacks | Reproducible, reviewable, environment-parity; modules encode the org's standards. |
| Identity | **Keycloak (JWT + JWKS)** | Roll-your-own auth | Standard OIDC IdP; API validates tokens statelessly against JWKS. |
| Pre-launch validation | **k6 load tests** | Hope it holds | Quantify capacity and tune scaling before a national-scale spike. |

## 5. Cost & scalability

Fargate scales task count with load and scales back down after the peak, so the national-scale event
was absorbed without permanent over-provisioning. CloudFront offloads static traffic from the origin
entirely. The shared ALB and a single ECR repo keep fixed costs low. Environments are sized
independently via tfvars (small sandbox, production-grade prod). The main scaling levers are ECS
desired/max count and RDS instance class; both are variables, not code changes.

## 6. Results / impact

- One Terraform blueprint deploys a full app stack per environment with a single `apply`.
- Proven under national-scale, bursty traffic during a live event.
- Reused as the foundation for a second production application with minimal new code.
- Security baseline (private DB, edge WAF, secrets, HTTPS) is built in, not bolted on.

## 7. Possible improvements

- **CI/CD**: pipeline that plans on PR and applies on merge, with per-env approvals.
- **Autoscaling policies** tuned from k6 baselines (target tracking on CPU/RPS).
- **Observability**: centralized logs/metrics/traces and dashboards per service.
- **Blue/green or canary** deploys on ECS for zero-downtime releases.
- Move remote-state backend config into the tooling so it's fully variable-driven.

---

**Stack:** `Terraform` · `AWS ECS Fargate` · `ALB` · `CloudFront + S3` · `RDS PostgreSQL` · `DocumentDB` · `ECR` · `Secrets Manager` · `Route 53 + ACM` · `WAF` · `Keycloak` · `SES` · `k6`
