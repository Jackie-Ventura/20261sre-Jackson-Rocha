# Arquitetura do Sistema: ETL Olist Marketplace (Versão 2.1 - OLAP & Resiliência)

Esta arquitetura segue o framework **RM-ODP** e está otimizada para alta performance analítica (OLAP) em ambiente **Codespace**, com total portabilidade para **AWS Academy**.

---

## 1. Pilares da Arquitetura (Fluxo de Dados)

O sistema é estruturado em quatro camadas fundamentais para garantir o SLA de 15 minutos e a integridade dos dados:

### 1.1 Object Storage (Camada de Persistência Bruta)
*   **Tecnologia:** **MinIO (Docker)** - Simula fielmente o AWS S3.
*   **Função:** Ponto de entrada único para os arquivos CSV.
*   **Estrutura de Buckets:**
    *   `landing/`: Recebe os arquivos `.csv` originais (Imutáveis).
    *   `archive/`: Armazena arquivos processados com sucesso (Segurança e Histórico).
    *   `quarantine/`: Arquivos com falhas catastróficas (ex: schema totalmente corrompido).

### 1.2 Ingestão (Camada de Processamento de Alta Performance)
*   **Tecnologia:** **Python + DuckDB**.
*   **Função:** Extração, Validação e Roteamento.
*   **Diferencial OLAP:** O DuckDB utiliza processamento vetorial, permitindo validar e transformar 100k linhas em menos de 5 minutos, garantindo ampla folga para o SLA total de 15 min.
*   **Lógica de Negócio:** Aplica regras de schema, sanitiza strings e converte moedas/datas antes da carga.

### 1.3 Análise OLAP (Camada de Inteligência e Armazenamento)
*   **Tecnologia:** **PostgreSQL 15 + BRIN Indexes**.
*   **Função:** Armazenamento analítico e suporte ao reprocessamento.
*   **Estratégias OLAP:**
    *   **Índices BRIN:** Otimizam buscas temporais massivas com baixo custo de disco.
    *   **JSONB (Quarentena):** Registros individuais que falharam na validação são salvos em formato JSONB, permitindo o **Reprocessamento Automatizado** sem perda da estrutura original.
    *   **Upsert:** Garante a **Idempotência** (sem duplicidade em caso de re-execução).

### 1.4 Visualização (Camada de Decisão)
*   **Tecnologia:** **Grafana**.
*   **Função:** Dashboards de saúde do negócio e do pipeline.
*   **Visibilidade:** Monitoramento em tempo real do volume de pedidos, status da quarentena e tempo de execução do ETL.

---

## 2. Visão de Engenharia (Infraestrutura)

| Componente | Desenvolvimento (Local/Codespace) | Produção (AWS Academy) |
| :--- | :--- | :--- |
| **Computação** | Python 3.11 em Docker | EC2 t3.medium / Fargate |
| **Storage** | MinIO (Container) | AWS S3 |
| **Banco de Dados** | PostgreSQL 15 (Container) | RDS PostgreSQL |
| **Observabilidade** | Grafana (Container) | Managed Grafana |

---

## 3. Architecture Decision Records (ADR)

### ADR-01: DuckDB para Validação Vetorial
*   **Contexto:** Necessidade de processar 100k linhas em menos de 15 minutos com baixo overhead de memória.
*   **Decisão:** Utilizar DuckDB como motor de transformação e validação.
*   **Consequência:** Ganho massivo de performance comparado ao Pandas puro e facilidade em lidar com tipos de dados complexos para OLAP.

### ADR-02: Índices BRIN (Block Range Index)
*   **Contexto:** Volume de dados crescente (~3M/mês) exigindo queries rápidas no Grafana.
*   **Decisão:** Utilizar BRIN em vez de B-Tree para colunas de timestamp.
*   **Consequência:** Reduz o tamanho do índice em até 99%, mantendo alta performance em consultas analíticas de datas.

### ADR-03: MinIO para Simulacro de S3
*   **Contexto:** Desenvolvimento em Codespace sem acesso direto à AWS.
*   **Decisão:** Utilizar MinIO via Docker.
*   **Consequência:** Garantia de que a lógica de "boto3" e buckets funcione perfeitamente quando migrada para a AWS real.

### ADR-04: Fluxo de Quarentena com JSONB
*   **Contexto:** Necessidade de reprocessamento automatizado de dados malformados.
*   **Decisão:** Salvar falhas de linha em tabela `orders_quarantine` com tipo JSONB.
*   **Motivo:** Permite o reprocessamento automatizado e evita o "sofrimento silencioso" (perda de dados sem rastro).

---
*Arquitetura validada para garantir Idempotência, Observabilidade e Resiliência.*
