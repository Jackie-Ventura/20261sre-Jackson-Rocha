# Project Context: Olist ETL Architecture & Specs

This project is a dedicated workspace for modeling the architecture and defining requirements for the Olist marketplace ETL pipeline. It follows the SWEBOK (Software Engineering Body of Knowledge) and ISO 25010 standards to ensure a high-quality, reliable data engineering solution.

## 📁 Directory Overview
The project is currently structured as a documentation and architectural repository:
- **`specs/`**: Contains the core specification and problem definition documents.
- **`README.md`**: Project identifier.

## 🔑 Key Files
### `specs/00_problem.md`
This is the **primary source of truth** for the project. It outlines:
- **Business Context**: Processing ~100k daily orders for analytical dashboards.
- **Stakeholders**: Business operations, data team, internal dashboard clients, and SRE.
- **Critical Flows**: CSV ingestion to Postgres, Grafana dashboards, and SLA monitoring.
- **Failure Modes & Risks**: Corrupted files, duplication, infrastructure failures, and "silent suffering" (data loss without alerts).
- **Requirements (SWEBOK/ISO 25010)**: Categorized into Functional (Ingestion, Validation, Upsert, Audit) and Non-Functional (Idempotency, Observability, Resilience, Performance, Security).

## 🛠️ Usage & Guidelines
- **Decision Alignment**: Every architectural proposal or implementation step MUST align with the constraints and requirements defined in `specs/00_problem.md`.
- **Core Principles**:
    - **Idempotency**: Retrying a job should never duplicate data. Use `UPSERT` strategies.
    - **Observability**: Failures must never be silent. Metrics and logs must be exposed for Grafana/Alerting.
    - **Resilience**: The system should handle partial failures and resume from checkpoints where possible.
- **Modeling Standards**: Use the eights axes of quality (Reliability, Maintainability, etc.) as a checklist for any proposed change.

## 🚀 Future Implementation (TODO)
While currently a documentation-first project, the following technologies are targeted:
- **Infrastructure**: AWS (EC2/Fargate, S3, RDS Postgres).
- **Orchestration**: Likely Airflow or AWS Step Functions.
- **Monitoring**: Prometheus and Grafana.
- **Validation**: Python (Pydantic / Great Expectations).
