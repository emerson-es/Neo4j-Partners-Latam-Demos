# 🏥 Neo4j para Saúde — Jornada do Paciente, CRM, Fraude e Marketing

> Material de pré-venda Neo4j para o setor de saúde, demonstrando capacidades e diferenciais para Hospitais, Clínicas e Operadoras de Planos de Saúde.

---

## 📋 Sobre o Projeto

Este repositório contém scripts Cypher, consultas de demonstração e documentação de pré-venda para o domínio de **Saúde e Seguros de Saúde**.

O material cobre os casos de uso de **CRM, Jornada do Paciente, Detecção de Fraude e Marketing** — com foco em rastreabilidade clínica ponta a ponta, segmentação de risco de beneficiários, auditoria de sinistros e segurança do paciente.

---

## 📊 Visão Geral do Modelo

| Entidade       | Detalhe                                                                      |
|----------------|------------------------------------------------------------------------------|
| Patient        | Dados demográficos, localização (POINT), custos de saúde acumulados          |
| Encounter      | Atendimentos com classe, custo, cobertura, datas e flags de início/fim       |
| Organization   | Hospitais e clínicas com localização geoespacial                             |
| Provider       | Médicos e prestadores com especialidade e organização                        |
| Payer          | Planos de saúde (Medicare, Cigna, Anthem, UnitedHealthcare etc.)             |
| Condition      | Diagnósticos e condições clínicas com datas de início e encerramento         |
| Drug           | Medicamentos prescritos com custo e período de uso                           |
| CarePlan       | Planos de cuidado com status ativo/encerrado                                 |
| Observation    | Exames e observações clínicas com valor, unidade e data                      |
| Allergy        | Alergias registradas com categoria (alimento, medicamento, ambiente)         |
| Reaction       | Reações alérgicas com severidade (MILD, MODERATE, SEVERE)                   |
| Procedure      | Procedimentos realizados por atendimento                                     |
| Speciality     | Especialidades médicas dos provedores                                        |

**Relacionamentos:**

| Relacionamento         | Semântica                                                     |
|------------------------|---------------------------------------------------------------|
| NEXT                   | Encounter sequencia para o próximo — jornada temporal         |
| HAS_ENCOUNTER          | Patient possui Encounter                                      |
| HAS_OBSERVATION        | Encounter registra Observation (com date, unit, value)        |
| HAS_PROCEDURE          | Encounter inclui Procedure                                    |
| HAS_CONDITION          | Encounter registra Condition (diagnóstico)                    |
| HAS_DRUG               | Encounter prescreve Drug                                      |
| HAS_PAYER              | Encounter coberto por Payer                                   |
| HAS_CARE_PLAN          | Encounter gera CarePlan                                       |
| HAS_PROVIDER           | Encounter atendido por Provider                               |
| AT_ORGANIZATION        | Encounter ocorre em Organization                              |
| INSURANCE_START / END  | Patient inicia ou encerra vínculo com Payer                   |
| HAS_ALLERGY            | Patient tem Allergy registrada                                |
| HAS_REACTION           | Patient sofre Reaction com severidade                         |
| CAUSES_REACTION        | Allergy causa Reaction                                        |
| FIRST / LAST           | Patient aponta para primeiro e último Encounter               |

---

## 🗂️ Modelo de Dados

```
(Patient)-[:FIRST]──────────────────────────→ (Encounter)
(Patient)-[:HAS_ENCOUNTER]─────────────────→ (Encounter)
(Patient)-[:LAST]───────────────────────────→ (Encounter)
(Patient)-[:HAS_ALLERGY]────────────────────→ (Allergy)-[:CAUSES_REACTION]→ (Reaction)
(Patient)-[:INSURANCE_START / END]──────────→ (Payer)

(Encounter)-[:NEXT]─────────────────────────→ (Encounter)  ← jornada temporal
(Encounter)-[:AT_ORGANIZATION]──────────────→ (Organization)
(Encounter)-[:HAS_PROVIDER]─────────────────→ (Provider)-[:HAS_SPECIALITY]→ (Speciality)
(Encounter)-[:HAS_PAYER]────────────────────→ (Payer)
(Encounter)-[:HAS_CONDITION]────────────────→ (Condition)
(Encounter)-[:HAS_DRUG]─────────────────────→ (Drug)
(Encounter)-[:HAS_PROCEDURE]────────────────→ (Procedure)
(Encounter)-[:HAS_CARE_PLAN]────────────────→ (CarePlan)
(Encounter)-[:HAS_OBSERVATION]──────────────→ (Observation)

(Provider)-[:BELONGS_TO]────────────────────→ (Organization)
```

---

## 🚀 Casos de Uso

| # | Caso de Uso                                  | Valor de Negócio                                                                               |
|---|----------------------------------------------|-----------------------------------------------------------------------------------------------|
| 1 | **Jornada do Paciente e Readmissão**         | Rastreia a linha do tempo completa de cada paciente e detecta padrões que precedem eventos críticos |
| 2 | **Detecção de Fraude em Sinistros**          | Identifica atendimentos de alto custo sem cobertura — alertas de billing e uso indevido        |
| 3 | **CRM e Gestão de População**                | Segmenta beneficiários por risco para programas de gestão de doença crônica                    |
| 4 | **Marketing e Engajamento com a Jornada**    | Identifica pacientes com planos de cuidado abertos e histórico de churn de plano               |
| 5 | **Segurança do Paciente — Alertas de Alergia** | Cruza alergias registradas com prescrições atuais — alerta de reação adversa em tempo real   |

---

## 🔍 Consultas de Demonstração

### Condições Mais Frequentes na População
```cypher
MATCH (e:Encounter)-[:HAS_CONDITION]->(c:Condition)
RETURN c.description AS condicao, count(e) AS ocorrencias
ORDER BY ocorrencias DESC LIMIT 15
```

### Sinistros Suspeitos — Emergências sem Cobertura
```cypher
MATCH (p:Patient)-[:HAS_ENCOUNTER]->(e:Encounter)-[:HAS_PAYER]->(pay:Payer)
WHERE e.class = 'emergency' AND e.claimCost > 10000 AND e.coveredAmount = 0
OPTIONAL MATCH (e)-[:HAS_CONDITION]->(c:Condition)
RETURN p.first + ' ' + p.last AS paciente, pay.name AS payer,
       e.claimCost AS custo, collect(DISTINCT c.description) AS diagnosticos
ORDER BY e.claimCost DESC
```

### Segmentação de Alto Risco (CRM)
```cypher
MATCH (p:Patient)-[:HAS_ENCOUNTER]->(e:Encounter)
WITH p, sum(e.claimCost) AS custo_total, count(e) AS atendimentos
WHERE custo_total > 200000
OPTIONAL MATCH (p)-[:HAS_ENCOUNTER]->(ec:Encounter)-[:HAS_CONDITION]->(c:Condition)
RETURN p.first + ' ' + p.last AS paciente, atendimentos,
       round(custo_total*100)/100 AS custo_total_usd,
       collect(DISTINCT c.description)[0..5] AS condicoes
ORDER BY custo_total DESC
```

### Jornada Visual de um Paciente (Grafo)
```cypher
MATCH (p:Patient)-[:FIRST]->(inicio:Encounter)
MATCH path = (inicio)-[:NEXT*0..6]->(enc:Encounter)
OPTIONAL MATCH (enc)-[rc:HAS_CONDITION]->(c:Condition)
OPTIONAL MATCH (enc)-[rp:HAS_PAYER]->(pay:Payer)
RETURN path, rc, c, rp, pay LIMIT 50
```

### Planos de Cuidado Abertos — Lista de Engajamento
```cypher
MATCH (e:Encounter)-[:HAS_CARE_PLAN]->(cp:CarePlan)
WHERE cp.isEnd = false
MATCH (p:Patient)-[:HAS_ENCOUNTER]->(e)
RETURN p.first + ' ' + p.last AS paciente,
       cp.description AS plano_cuidado, cp.start AS inicio
ORDER BY cp.start LIMIT 20
```

### Cobertura por Plano de Saúde
```cypher
MATCH (e:Encounter)-[:HAS_PAYER]->(pay:Payer)
RETURN pay.name AS plano, count(e) AS atendimentos,
       round(avg(e.claimCost)*100)/100 AS custo_medio,
       round(avg(e.coveredAmount)*100)/100 AS cobertura_media,
       round(avg(e.claimCost - e.coveredAmount)*100)/100 AS gap_medio
ORDER BY atendimentos DESC
```

---

## 📁 Documentação

| Arquivo | Descrição |
|---------|-----------|
| [`Neo4j_PreVenda_Saude.pdf`](./Neo4j_PreVenda_Saude.pdf) | Documento completo de pré-venda com modelo de dados, casos de uso, scripts comentados, queries de demo e comparativo Neo4j vs SQL para saúde |

O documento inclui:
- ✅ Inventário completo do modelo de dados (14 tipos de nó, 15 tipos de relacionamento)
- ✅ Texto explicativo para audiência não-técnica (CMOs, gestores de operadoras, diretores clínicos)
- ✅ 4 scripts Cypher reutilizáveis e parametrizados
- ✅ 6 consultas prontas para demonstração ao vivo no Neo4j Browser
- ✅ Tabela comparativa Neo4j vs SQL/EMR tradicional para argumentação em pré-venda

---

## 📌 Planos de Saúde no Modelo

| Plano               | Atendimentos | Custo Médio (USD) | Cobertura Média (USD) |
|---------------------|-------------|--------------------|-----------------------|
| NO_INSURANCE        | 1.212        | $ 5.166,33         | $ 0,00                |
| Medicare            | 835          | $ 4.094,62         | $ 3.248,87            |
| Cigna Health        | 595          | $ 2.348,11         | $ 1,42                |
| Medicaid            | 581          | $ 7.684,58         | $ 7.255,13            |
| Anthem              | 534          | $ 3.076,27         | $ 0,77                |
| Blue Cross Blue Shield | 521       | $ 2.932,50         | $ 1.861,71            |
| UnitedHealthcare    | 511          | $ 3.541,89         | $ 0,00                |
| Aetna               | 447          | $ 2.310,78         | $ 0,17                |
| Humana              | 418          | $ 2.259,31         | $ 0,00                |

---

## ⚖️ Neo4j vs. SQL para Saúde — Resumo

| Capacidade                    | SQL / EMR Tradicional                         | Neo4j                                               |
|-------------------------------|-----------------------------------------------|-----------------------------------------------------|
| Jornada temporal do paciente  | Consultas recursivas complexas                | Relacionamento NEXT — linha do tempo nativa         |
| Detecção de fraude            | Regras batch noturnas                         | Padrões estruturais em tempo real                   |
| Segmentação de risco (CRM)    | Relatórios estáticos e snapshots              | Segmentação dinâmica por qualquer combinação clínica |
| Alerta de interação medicamentosa | Tabela de contra-indicações pré-configurada | Cruzamento dinâmico Allergy × Drug                 |
| Análise de readmissão         | JOINs entre tabelas de internação             | Caminho NEXT até internação — 1 query               |
| Dados geoespaciais            | Extensões externas necessárias                | POINT nativo em Patient, Organization, Provider     |

---

## 🛠️ Tecnologias

- **Neo4j** — banco de dados de grafo
- **Cypher** — linguagem de consulta
- **Neo4j GDS** (Graph Data Science) — algoritmos de centralidade e detecção de padrões clínicos
- **Neo4j Browser / Bloom** — visualização interativa da jornada do paciente

---
