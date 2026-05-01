# Arquitetura do Sistema: ETL Olist Marketplace

Esta arquitetura segue o framework **RM-ODP** (Reference Model of Open Distributed Processing) e está alinhada com as restrições do AWS Academy Learner Lab.

---

## 1. Enterprise Viewpoint (Visão de Negócio)
O sistema tem como objetivo prover dados precisos e oportunos para a tomada de decisão tática da Olist, garantindo que ~100 mil pedidos diários sejam processados sem duplicidade ou perda silenciosa.
*   **Atores:** Operação Olist, Time de Dados, SRE.
*   **Objetivos:** Idempotência, Observabilidade e Baixo Custo Operacional.

## 2. Information Viewpoint (Visão de Informação)
Define os modelos de dados e estados do pipeline.
*   **Landing (S3):** Arquivos CSV brutos (Imutáveis).
*   **Audit/Metadata (Postgres):** Tabela `job_runs` para controle de estado e `reconciliation_logs` (Atende **RF-04, RF-06, RF-09**).
*   **Analytics (Postgres):** Tabela `orders` com PK em `order_id` (Atende **RF-03, RF-05**).
*   **Quarentena (Postgres):** Tabela `orders_quarantine` para registros malformados (Atende **RF-07, RNF-07**).

## 3. Computational Viewpoint (Visão Funcional)
Decomposição lógica em componentes independentes.
*   **Ingestor/Validator:** Componente que lê chunks do S3 e aplica regras de schema (Atende **RF-01, RF-02, RNF-12**).
*   **Upsert Engine:** Lógica de persistência atômica no Postgres (Atende **RF-03, RNF-01**).
*   **Observer/Notifier:** Emite métricas Prometheus e dispara webhooks em falhas (Atende **RF-08, RNF-08**).
*   **Archiver:** Move arquivos processados entre prefixos S3 (Atende **RF-10**).

## 4. Engineering Viewpoint (Visão de Infraestrutura)
Distribuição física dos componentes no ambiente AWS Academy.
*   **Computação:** Instância EC2 (t3.medium) executando Workers em Python via Docker. O orquestrador (Cron/Airflow local) gerencia o ciclo de vida.
*   **Armazenamento:** S3 Standard para Landing e Archive.
*   **Banco de Dados:** RDS Postgres (db.t3.micro/medium) para metadados e dados analíticos.
*   **Observabilidade:** Exportador customizado (Python) para Prometheus e dashboard no Grafana (Atende **RF-05, RNF-10**).

## 5. Technology Viewpoint (Visão Tecnológica)
Stack tecnológica escolhida.
*   **Linguagem:** Python 3.11+ (Pandas/SQLAlchemy/Pydantic).
*   **Banco:** PostgreSQL 15.
*   **Cloud:** AWS Academy Learner Lab (EC2, S3, RDS).
*   **Segurança:** AWS Secrets Manager + IAM Roles (Atende **RNF-05**).
*   **Provisionamento:** Scripts Terraform ou Bash/AWS-CLI (Atende **RNF-11**).

---

## 🏗️ Architecture Decision Records (ADR)

### ADR-01: Uso de RDS Postgres como Banco Analítico
*   **Contexto:** Restrição de "sem Redshift" e volume moderado (~100k/dia).
*   **Decisão:** Utilizar RDS Postgres tanto para metadados quanto para o banco analítico final.
*   **Consequência:** Simplifica a arquitetura e reduz custo, mas exige monitoramento de concorrência (conforme **RNF-09**).

### ADR-02: Estratégia de Upsert via SQL Nativo
*   **Contexto:** Necessidade de idempotência absoluta e resiliência a reprocessos.
*   **Decisão:** Utilizar a instrução `INSERT ... ON CONFLICT (order_id) DO UPDATE`.
*   **Consequência:** Garante que re-execuções não dupliquem dados (**RNF-06**) e mantém a performance necessária (**RNF-03**).

### ADR-03: Processamento Baseado em EC2/Docker (Foco em Custo/Lab)
*   **Contexto:** Restrição de uso do AWS Glue e limites do Learner Lab.
*   **Decisão:** Rodar o código de processamento em containers Docker dentro de uma EC2 t3.medium.
*   **Consequência:** Maior controle sobre as dependências e bibliotecas de validação, mas exige gestão manual de logs e escalabilidade vertical.

---
*Documento gerado conforme framework RM-ODP e restrições de projeto.*
