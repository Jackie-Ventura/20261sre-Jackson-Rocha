# Requisitos Não Funcionais (RNF) - ETL Olist

Este documento define os critérios de qualidade para o sistema de processamento de pedidos, mapeados sob a norma ISO 25010, com indicadores mensuráveis (SLI/SLO).

---

## 1. Mapeamento por Atributos ISO 25010

### 1.1 Confiabilidade (Reliability)
Focado em garantir que o sistema opere corretamente mesmo sob falhas parciais.
*   **RNF-01 (Tolerância a Falhas):** O pipeline deve ser capaz de realizar até 3 tentativas de conexão com o Postgres antes de falhar o job.
*   **RNF-02 (Disponibilidade de Dados):** Os dados no banco analítico devem estar disponíveis para consulta 99.9% do tempo após o processamento.

### 1.2 Eficiência de Desempenho (Performance Efficiency)
Garante o cumprimento da janela de tempo do negócio.
*   **RNF-03 (Time-to-Insight):** O processamento de um lote de 100k linhas deve ser concluído em no máximo 15 minutos (95th percentile).
*   **RNF-04 (Vazão de Ingestão):** O sistema deve sustentar uma taxa de ingestão de pelo menos 150 linhas por segundo.

### 1.3 Segurança (Security)
Proteção contra acessos indevidos e corrupção.
*   **RNF-05 (Proteção de Credenciais):** 100% das chaves de acesso (DB, AWS) devem ser rotacionadas e acessadas via Secrets Manager, com zero segredos em texto puro.
*   **RNF-06 (Integridade de Dados):** A soma do campo `price` no destino deve ser idêntica à soma na origem com erro tolerável de 0.00%.

### 1.4 Manutenibilidade (Maintainability)
Facilidade de diagnóstico e evolução.
*   **RNF-07 (Tempo de Diagnóstico - MTTR):** Em caso de falha, os logs devem permitir identificar a causa raiz (ex: erro de schema) em menos de 5 minutos de análise.
*   **RNF-08 (Observabilidade):** 100% das execuções do job devem emitir métricas de `job_status` e `rows_processed` para o Prometheus.

### 1.5 Compatibilidade (Compatibility)
Interoperabilidade entre sistemas.
*   **RNF-09 (Coexistência):** O processo de ingestão não deve consumir mais que 20% das conexões simultâneas totais do banco Postgres para não impactar o Grafana.

### 1.6 Usabilidade / Operabilidade (Usability)
Foco na facilidade de operação pelo time de dados/SRE.
*   **RNF-10 (Tempo de Resposta Dashboard):** As consultas analíticas no Grafana devem retornar resultados em menos de 3 segundos para janelas de 24h.

### 1.7 Portabilidade (Portability)
Facilidade de deploy.
*   **RNF-11 (Infras-as-Code):** 100% da infraestrutura necessária (S3, RDS, Orquestrador) deve ser provisionável via scripts (Terraform/CloudFormation) em menos de 30 min.

### 1.8 Adequação Funcional (Functional Suitability)
Precisão dos cálculos.
*   **RNF-12 (Precisão):** O arredondamento de valores monetários deve seguir o padrão bancário (Half Even) para evitar distorções de centavos em 100k pedidos.

---

## 2. Matriz de Requisitos (SLI/SLO/MoSCoW)

| ID | Atributo ISO | SLI (Indicador) | SLO (Meta) | Fonte de Medição | Prioridade |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **RNF-01** | Confiabilidade | Qtd de Retries tentados | Max 3 | Logs do Orquestrador | **Must** |
| **RNF-03** | Performance | Duração total do Job | < 15 min | CloudWatch Metrics | **Must** |
| **RNF-05** | Segurança | Audit log de secrets | 0 Hardcoded Keys | Git Secret Scanner | **Must** |
| **RNF-06** | Segurança | Diff `SUM(price)` | 0.00% | Job Reconciliation Report | **Must** |
| **RNF-08** | Manutenibilidade | Cobertura de métricas | 100% dos Jobs | Grafana Health Check | **Should** |
| **RNF-09** | Compatibilidade | % Conexões DB usadas | < 20% | Postgres `pg_stat_activity` | **Should** |
| **RNF-10** | Usabilidade | Latência de Query | < 3s | Grafana Query Inspector | **Could** |
| **RNF-12** | Adequação | Divergência Centavos | 0 | Teste de Unidade / Auditoria | **Must** |

---

## 3. Premissas e Fontes
1.  **Premissa de Infra:** O banco Postgres RDS (m5.large ou superior) estará ativo durante a janela de processamento.
2.  **Premissa de Rede:** A latência entre a Landing Zone (S3) e o Worker (EC2/Fargate) é desprezível dentro da mesma região AWS.
3.  **Fonte de Medição Principal:** Métricas extraídas via Prometheus (node_exporter/custom metrics) e Logs estruturados em JSON.

---
*Documento elaborado seguindo os eixos SWEBOK e ISO 25010.*
