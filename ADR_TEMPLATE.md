# Case template — Architecture Decision Record (ADR)

> Copy this file as `README.md` inside each case folder and fill in the sections.
> The goal is not to document code, but to **tell an architecture decision**: why it was solved
> this way and which alternatives were rejected. A technical recruiter should get it in 5 minutes.

---

# [Case name]

**Role:** [Data Architect / Analytics Engineer / …] · **Year:** [YYYY] · **Status:** [Production / PoC / Personal]

> **Confidentiality note:** case based on a real project, recreated with a generic domain and
> synthetic data. It contains no internal information, real names or production identifiers.

**One-line summary:** [What problem it solves and with what approach, in one sentence.]

---

## 1. Problem
What was needed and why it mattered (the business "why", in neutral terms).

## 2. Context & constraints
What shaped the design: data volumes, budget, acceptable latency, team, existing tech in the
organization, compliance requirements, etc.

## 3. Proposed architecture
Diagram + end-to-end flow description:

`![Architecture](diagrams/architecture.svg)`

Walk through the path of the data/request across each component.

## 4. Technology choices & rationale
For each relevant decision, the pattern: **I chose X because… / I rejected Y because…**
This is the section that best communicates architectural judgment.

| Decision | Chosen | Rejected alternatives | Why |
|---|---|---|---|
| | | | |

## 5. Cost & scalability
How it scales, where the bottlenecks are, and the cost levers (serverless vs. provisioned,
partitioning, caching, batch vs. streaming).

## 6. Results / impact
What was achieved, measurable and non-confidential.

## 7. Possible improvements
What I'd do differently or what's next: known tech debt, next iteration, GOLD layer, tests,
observability.

---

**Stack:** `tech1` · `tech2` · `tech3`
