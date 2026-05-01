# Levantamento de Requisitos Funcionais (RF) - ETL Olist

Este documento descreve as funcionalidades necessárias para o pipeline de processamento de pedidos do marketplace, garantindo a rastreabilidade e a corretude dos dados.

## 📋 Tabela de Requisitos Funcionais

| ID | Título | Descrição | Prioridade | Fonte/Rastreabilidade |
| :--- | :--- | :--- | :--- | :--- |
| **RF-01** | Extração Diária | O sistema deve extrair aproximadamente 100 mil pedidos de arquivos CSV diariamente. | Essencial | `00_problem.md` (RF01) |
| **RF-02** | Validação de Integridade | O sistema deve validar o schema (colunas, tipos) e a consistência de cada linha do CSV antes da carga. | Essencial | `00_problem.md` (RF02) |
| **RF-03** | Carga Idempotente (Upsert) | O sistema deve realizar a carga no Postgres utilizando a lógica de *Upsert* (inserir ou atualizar) baseada na PK do pedido. | Essencial | `00_problem.md` (RF03 / Modo de Falha: Reprocesso) |
| **RF-04** | Registro de Metadados | O sistema deve gravar logs de auditoria para cada lote (Data, Qtd Linhas Lidas, Inseridas, Erros). | Essencial | `00_problem.md` (RF04) |
| **RF-05** | Visualização em Dashboard | O sistema deve disponibilizar os dados processados para consulta via dashboards no Grafana. | Essencial | `00_problem.md` (Fluxo Crítico: Dashboards) |
| **RF-06** | Rastreabilidade de SLA | O sistema deve registrar o tempo de início e fim de cada execução para monitoramento de performance. | Importante | `00_problem.md` (Observação de SLA) |
| RF-07 | Tratamento de Quarentena | O sistema deve isolar linhas com erro de validação em uma tabela de quarentena sem interromper o processo total. | Importante | `00_problem.md` (Modo de Falha: Arquivo Corrompido) |
| **RF-08** | Notificação de Alerta Crítico | O sistema deve disparar notificações (Slack/Webhook) imediatamente em caso de falha catastrófica ou quebra de SLA. | Essencial | `00_problem.md` (Risco: Sofrer Silenciosamente) |
| **RF-09** | Reconciliação de Dados | O sistema deve comparar o total de registros lidos na origem com a soma de (Inseridos + Quarentena) e logar a diferença. | Essencial | `00_problem.md` (Garantia de Confiança) |
| **RF-10** | Gestão de Arquivos Processados | O sistema deve mover arquivos CSV processados com sucesso para uma pasta de `/archive` para evitar reprocessamento acidental. | Importante | `00_problem.md` (Risco: Custo AWS / Eficiência) |
| **RF-11** | Reprocessamento sob Demanda | O sistema deve permitir que um operador dispare o reprocessamento de uma data específica via parâmetro. | Importante | `00_problem.md` (Modo de Falha: Banco Indisponível) |

## ✅ Critérios de Sucesso (Validação)

*   **Validação de RF-01/RF-02:** O sistema deve rejeitar arquivos com colunas faltantes e logar o erro.
*   **Validação de RF-03:** Executar o mesmo arquivo duas vezes e confirmar que o `COUNT(*)` no Postgres permanece idêntico.
*   **Validação de RF-04:** Consultar a tabela de metadados e encontrar o histórico de execuções dos últimos 7 dias.
*   **Validação de RF-07:** Inserir propositalmente uma linha inválida no CSV e verificar se ela foi movida para a quarentena enquanto as outras foram carregadas.
*   **Validação de RF-09:** Se a origem tem 100k linhas e o destino + quarentena tem 99k, o sistema deve gerar um alerta de "Data Loss Detected".
*   **Validação de RF-10:** Verificar se após o job, o arquivo original não consta mais na pasta de entrada (Landing Zone).

---
*Documento gerado automaticamente via Skill de Engenharia de Requisitos.*
