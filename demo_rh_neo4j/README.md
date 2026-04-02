# 👥 Neo4j para Recursos Humanos — Skills, Alocação e Desenvolvimento

> Material de pré-venda Neo4j para o setor de RH, demonstrando capacidades e diferenciais para Gestão de Talentos, Mapeamento de Competências e Planejamento de Carreira.

---

## 📋 Sobre o Projeto

Este repositório contém scripts Cypher, consultas de demonstração e documentação de pré-venda para o domínio de **Recursos Humanos**.

O material cobre os casos de uso de **Funcionários, Skills, Alocação, Desenvolvimento e Treinamento** — com foco em Gestão de Talentos, Mapeamento de Competências e Planejamento de Carreira.

---

## 📊 Visão Geral do Modelo

| Entidade    | Detalhe                                   |
|-------------|-------------------------------------------|
| Pessoa      | first_name, last_name, gender             |
| Empresa     | empresas empregadoras                     |
| Competencia | skills técnicas e gerenciais              |
| Cargo       | Software Engineer, Product Manager        |

**Relacionamentos:**

| Relacionamento   | Semântica                                      |
|------------------|------------------------------------------------|
| TEM_COMPETENCIA  | Pessoa possui uma Competencia                  |
| TRABALHOU_PARA   | Vínculo profissional passado com Empresa       |
| AMIGO_DE         | Rede social interna entre Pessoas              |
| TRABALHA_PARA    | Vínculo ativo com Empresa                      |
| E_NECESSARIO     | Cargo requer uma Competencia específica        |

---

## 🗂️ Modelo de Dados

```
(Cargo)-[:E_NECESSARIO]--------→ (Competencia)
                                       ↑
(Pessoa)-[:TEM_COMPETENCIA]───────────/
(Pessoa)-[:TRABALHA_PARA]──────→ (Empresa)
(Pessoa)-[:TRABALHOU_PARA]─────→ (Empresa)
(Pessoa)-[:AMIGO_DE]───────────→ (Pessoa)
```

---

## 🛠️ Competências Mapeadas

| Competencia              | Categoria |
|--------------------------|-----------|
| People Management        | Gestão    |
| General Management       | Gestão    |
| Scala Programming        | Técnica   |
| Java Programming         | Técnica   |
| Project Management       | Gestão    |
| Financial Management     | Gestão    |
| Python Programming       | Técnica   |
| Product Management       | Gestão    |
| Visual Basic Programming | Técnica   |
| Carpenter                | Técnica   |

---

## 🚀 Casos de Uso

| # | Caso de Uso                              | Valor de Negócio                                                                     |
|---|------------------------------------------|--------------------------------------------------------------------------------------|
| 1 | **Matching de Candidatos**               | Identifica quem possui 100% das skills de uma vaga — em milissegundos                |
| 2 | **Gap Analysis e PDI**                   | Aponta exatamente quais competências faltam para cada pessoa chegar ao próximo nível |
| 3 | **Influenciadores da Rede Interna**      | Descobre os conectores organizacionais para mentoria e gestão de mudança             |
| 4 | **Trajetória e Retenção**                | Mapeia histórico de carreira completo e detecta padrões de turnover                  |
| 5 | **Recomendação de Times e Treinamentos** | Monta equipes com cobertura complementar de skills e prioriza treinamentos           |

---

## 🔍 Consultas de Demonstração

### Matching de Candidatos para Software Engineer
```cypher
MATCH (cargo:Cargo {name: 'Software Engineer'})-[:E_NECESSARIO]->(req:Competencia)
WITH collect(req.name) AS reqs, count(req) AS total
MATCH (p:Pessoa)-[:TEM_COMPETENCIA]->(c:Competencia)
WHERE c.name IN reqs
WITH p, collect(DISTINCT c.name) AS match, total
WHERE size(match) = total
RETURN p.first_name + ' ' + p.last_name AS candidato, match
```

### Gap Analysis — O que falta para uma promoção
```cypher
WITH 'Rodolfo Gregory' AS nomePessoa, 'Product Manager' AS nomeCargo
MATCH (cargo:Cargo {name: nomeCargo})-[:E_NECESSARIO]->(req:Competencia)
WITH nomePessoa, collect(req.name) AS requeridas
MATCH (p:Pessoa) WHERE p.first_name + ' ' + p.last_name = nomePessoa
OPTIONAL MATCH (p)-[:TEM_COMPETENCIA]->(possui:Competencia)
WITH p, collect(DISTINCT possui.name) AS tem, requeridas
RETURN
  p.first_name + ' ' + p.last_name AS pessoa,
  [s IN requeridas WHERE s IN tem]     AS skills_ok,
  [s IN requeridas WHERE NOT s IN tem] AS gaps_para_desenvolver
```

### Top Influenciadores da Rede Interna
```cypher
MATCH (p:Pessoa)-[:AMIGO_DE]-(amigo:Pessoa)
WITH p, count(DISTINCT amigo) AS conexoes
OPTIONAL MATCH (p)-[:TRABALHA_PARA]->(e:Empresa)
RETURN p.first_name + ' ' + p.last_name AS influenciador,
       e.name AS empresa, conexoes
ORDER BY conexoes DESC LIMIT 10
```

### Perfil Completo de um Profissional (Visualização)
```cypher
MATCH (p:Pessoa {first_name: 'Francisco', last_name: 'Morales'})
OPTIONAL MATCH (p)-[r1:TEM_COMPETENCIA]->(c:Competencia)
OPTIONAL MATCH (p)-[r2:TRABALHA_PARA]->(e:Empresa)
OPTIONAL MATCH (p)-[r3:AMIGO_DE]-(amigo:Pessoa)
RETURN p, r1, c, r2, e, r3, amigo
```

### Talentos Mais Versáteis (Multi-skill)
```cypher
MATCH (p:Pessoa)-[:TEM_COMPETENCIA]->(c:Competencia)
WITH p, count(c) AS num_skills
WHERE num_skills >= 4
OPTIONAL MATCH (p)-[:TRABALHA_PARA]->(e:Empresa)
RETURN p.first_name + ' ' + p.last_name AS talento,
       e.name AS empresa, num_skills
ORDER BY num_skills DESC LIMIT 10
```

### Caminhos entre Dois Profissionais pela Rede Interna
```cypher
MATCH (a:Pessoa {first_name: 'Ellis', last_name: 'Blair'}),
      (b:Pessoa {first_name: 'Harriet', last_name: 'Kennedy'})
MATCH path = shortestPath((a)-[:AMIGO_DE*..6]-(b))
RETURN [n IN nodes(path) | n.first_name + ' ' + n.last_name] AS caminho,
       length(path) AS graus_de_separacao
```

---

## 📁 Documentação

| Arquivo | Descrição |
|---------|-----------|
| [`Neo4j_PreVenda_RH.docx`](./Neo4j_PreVenda_RH.docx) | Documento completo de pré-venda com modelo de dados, casos de uso, scripts comentados, queries de demo e comparativo Neo4j vs SQL para RH |

O documento inclui:
- ✅ Inventário completo do modelo de dados (nós, relacionamentos, propriedades)
- ✅ Texto explicativo para audiência não-técnica de RH e negócios
- ✅ 4 scripts Cypher reutilizáveis e parametrizados
- ✅ 6 consultas prontas para demonstração ao vivo
- ✅ Tabela comparativa Neo4j vs SQL/HRIS tradicional para argumentação em pré-venda

---

## ⚖️ Neo4j vs. SQL para RH — Resumo

| Capacidade               | SQL / HRIS Tradicional            | Neo4j                                    |
|--------------------------|-----------------------------------|------------------------------------------|
| Matching de candidatos   | JOINs múltiplos entre tabelas     | 1 query — resultado em milissegundos     |
| Gap Analysis             | Cálculo manual em planilhas       | Diferença de conjuntos nativa            |
| Rede social interna      | Não modelável em tabelas          | Grafo social nativo                      |
| Trajetória de carreira   | Histórico em logs desconexos      | Timeline navegável                       |
| Recomendação de times    | Processo manual e subjetivo       | Algoritmo de cobertura de skills         |
| Visualização nativa      | Requer BI externo                 | Neo4j Bloom incluído                     |

---

## 🛠️ Tecnologias

- **Neo4j** — banco de dados de grafo
- **Cypher** — linguagem de consulta
- **Neo4j GDS** (Graph Data Science) — algoritmos de centralidade e comunidade
- **Neo4j Browser / Bloom** — visualização interativa

---
