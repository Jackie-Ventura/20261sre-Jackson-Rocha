# Problema: ETL de Pedidos Marketplace Olist

## 1. Contexto do Negócio
O negócio Olist precisa processar ~100 mil pedidos do marketplace diariamente, carregar em um banco analítico e gerar dashboards. O ETL deve ser confiável: idempotente, observável e resiliente a falhas parciais. Não podemos perder dados, nem duplicá-los, nem sofrer silenciosamente.

## 2. Mapeamento do Framework

### 2.1 Stakeholders
*   **Operação Olist (Negócio):** Consome os números para decisões táticas; exige precisão absoluta.
*   **Time de Dados:** Responsável pela arquitetura do ETL e correção de bugs; exige manutenibilidade.
*   **Clientes internos do dashboard:** Gerentes e diretores que precisam de disponibilidade e baixa latência.
*   **Plataforma / SRE:** Garante a infraestrutura e o cumprimento do SLA; exige observabilidade.

### 2.2 Fluxos Críticos
*   **Ingestão diária CSV → Postgres:** Processo de extração e carga dos dados brutos.
*   **Consulta de dashboards Grafana:** Visualização das métricas de negócio.
*   **Observação de SLA:** Monitoramento do tempo de execução e sucesso do pipeline.

### 2.3 Modos de Falha
*   **Arquivo corrompido ou parcial:** Dados malformados que podem quebrar o processamento.
*   **Reprocesso duplicando linhas:** Falha na idempotência inflacionando métricas.
*   **Queda de EC2 durante o run:** Interrupção brusca da infraestrutura.
*   **Banco indisponível:** Falha de conexão com o banco analítico (Postgres).

### 2.4 Riscos Sistêmicos
*   **Perda silenciosa de linhas:** Dados que não chegam ao dashboard sem gerar erro explícito.
*   **Dashboard mostrando dado stale (obsoleto):** Exibição de dados defasados por falha na atualização.
*   **Custo AWS explodindo:** Ineficiência técnica gerando gastos desnecessários.
*   **Dívida técnica de IA mal revisada:** Lógicas de ETL complexas e pouco testadas.

---

## 3. Requisitos de Engenharia (SWEBOK / ISO 25010)

### 3.1 Os 8 Eixos de Qualidade Não Funcional
1.  **Confiabilidade (Reliability):** Foco em Idempotência e Tolerância a falhas.
2.  **Manutenibilidade (Maintainability):** Foco em Observabilidade e facilidade de diagnóstico.
3.  **Adequação Funcional:** Correção e completude das regras de transformação.
4.  **Eficiência de Desempenho:** Processamento dentro da janela de tempo (SLA).
5.  **Segurança:** Integridade e proteção contra corrupção de dados.
6.  **Operabilidade:** Monitoramento eficaz via Grafana/Logs.
7.  **Compatibilidade:** Interoperabilidade entre CSV e Postgres.
8.  **Portabilidade:** Facilidade de deploy entre ambientes.

### 3.2 Requisitos Funcionais (RF)
*   **RF01:** Extração diária de arquivos CSV (~100k linhas).
*   **RF02:** Validação de esquema e integridade de cada linha.
*   **RF03:** Carga de dados no Postgres utilizando o padrão *Upsert* para garantir idempotência.
*   **RF04:** Registro de metadados e auditoria de cada lote processado.

### 3.3 Requisitos Não Funcionais (RNF)
*   **RNF01 (Idempotência):** Múltiplas execuções do mesmo lote não devem gerar duplicatas.
*   **RNF02 (Observabilidade):** Alerta ativo (Slack/PagerDuty) em caso de falha.
*   **RNF03 (Resiliência):** Capacidade de retomar o processamento após falha de infraestrutura.
*   **RNF04 (Integridade):** Verificação de conciliação (Reconciliation Check) entre origem e destino.
*   **RNF05 (Performance):** Ingestão completa em no máximo 15 minutos.

---

## 4. Garantia de Confiança do Negócio
Para que o negócio confie nos números, o sistema deve garantir a **Determinística do Processo**. Isso é alcançado através da **Rastreabilidade** (saber o status de cada pedido) e da **Consistência** (provar que o dado no dashboard é o reflexo exato da fonte), eliminando o sofrimento silencioso através da observabilidade ativa.
