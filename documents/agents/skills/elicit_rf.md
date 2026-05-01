# Skill: Elicitação de Requisitos Funcionais (RF)

Esta skill capacita o agente a extrair, refinar e documentar requisitos funcionais focados em pipelines de dados e sistemas analíticos.

## 🎯 Objetivo
Transformar necessidades de negócio e fluxos críticos em descrições técnicas acionáveis do que o sistema deve "fazer".

## 📋 Critérios de Aceite para um RF
Um Requisito Funcional bem definido nesta arquitetura deve:
1.  **Identificador Único:** Seguir o padrão `RF-XX`.
2.  **Descrição Clara:** Verbo de ação + objeto + condição (ex: "O sistema deve validar a integridade do CSV...").
3.  **Prioridade:** (Essencial, Importante, Desejável).
4.  **Rastreabilidade:** Apontar para qual item do `00_problem.md` ele atende.
5.  **Critério de Sucesso:** Como validar que o requisito foi entregue.

## 🛠️ Workflow de Elicitação
1.  **Análise de Entrada:** Ler `specs/00_problem.md` e identificar verbos de ação nos "Fluxos Críticos".
2.  **Decomposição:** Quebrar fluxos complexos em passos atômicos.
3.  **Formatação:** Aplicar o template abaixo.

## 📄 Template de Saída
| ID | Título | Descrição | Prioridade | Fonte/Rastreabilidade |
| :--- | :--- | :--- | :--- | :--- |
| RF-01 | Ingestão Diária | O sistema deve capturar arquivos CSV da landing zone (S3) a cada 24h. | Essencial | Fluxo Crítico: Ingestão |
| RF-02 | Validação de Schema | O sistema deve validar se o CSV possui as colunas `order_id`, `price` e `date`. | Essencial | Modo de Falha: Arquivo Corrompido |

## 🚫 O que NÃO é um RF
-   "O sistema deve ser rápido" (Isso é RNF).
-   "O banco deve ser Postgres" (Isso é Restrição de Design).
-   "O código deve ser limpo" (Isso é Padrão de Engenharia).
