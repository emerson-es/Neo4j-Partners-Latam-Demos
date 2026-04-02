# ✈️ Neo4j para Aviação — Mapeamento de Rotas e Redes de Conexão

> Material de pré-venda Neo4j para o setor de aviação, demonstrando capacidades e diferenciais para Logística, Supply Chain e Operações Aéreas.

---

## 📋 Sobre o Projeto

Este repositório contém scripts Cypher, consultas de demonstração e documentação de pré-venda para o domínio de **Aviação e Logística**.

O material cobre o caso de uso de **Mapeamento de Rotas e Redes de Conexão**, com foco em Logística, Supply Chain e Operações Aéreas — demonstrando as capacidades e diferenciais do Neo4j para audiências técnicas e de negócios.

---

## 📊 Visão Geral do Modelo

| Entidade        | Detalhe                                          |
|-----------------|--------------------------------------------------|
| Rota            | nvoo, origem, destino, cia aérea, horários, data |
| Aeroporto       | ICAO, nome, tipo, situação, coordenadas, centralidades (betweenness, eigenvector, degree), cluster (louvain) |
| Empresa Aerea   | ICAO, nome                                       |
| Pais            | nome                                             |
| Estado          | nome, UF                                         |
| Municipio       | nome, UF                                         |

**Relacionamentos:**

| Relacionamento           | Semântica                                  |
|--------------------------|--------------------------------------------|
| TEM_ROTA_ORIGEM          | Aeroporto origina uma Rota                 |
| TEM_ROTA_DESTINO         | Rota chega a um Aeroporto                  |
| ASSOCIADA_A_EMPRESA_AEREA| Rota pertence a uma Empresa Aérea          |
| CONECTADO_A              | Aeroporto conectado a outro (com peso)     |
| SERVE_MUNICIPIO          | Aeroporto atende um Município              |
| LOCALIZADO_EM_MUNICIPIO  | Aeroporto está sediado em um Município     |
| PARTE_DE_ESTADO          | Município pertence a um Estado             |
| ESTA_EM_PAIS             | Estado pertence a um País                  |
| ASSOCIADO_A_PAIS         | Aeroporto internacional vinculado a País   |

---

## 🗂️ Modelo de Dados

```
(Pais) <-[:ESTA_EM_PAIS]- (Estado) <-[:PARTE_DE_ESTADO]- (Municipio)
                                                               ↑
(Aeroporto) -[:SERVE_MUNICIPIO]-----------------------------→ |
(Aeroporto) -[:LOCALIZADO_EM_MUNICIPIO]-----------------→ (Municipio)
(Aeroporto) -[:ASSOCIADO_A_PAIS]------------------------→ (Pais)
(Aeroporto) -[:CONECTADO_A {conexoes}]------------------→ (Aeroporto)
(Aeroporto) -[:TEM_ROTA_ORIGEM]------------------------→ (Rota)
(Rota)      -[:TEM_ROTA_DESTINO]-----------------------→ (Aeroporto)
(Rota)      -[:ASSOCIADA_A_EMPRESA_AEREA]--------------→ (Empresa Aerea)
```

**Algoritmos de grafo pré-computados** nos nós `Aeroporto`:
- `betweennessCentrality` — criticidade como ponto de passagem na rede
- `eigenvectorCentrality` — importância considerando vizinhos influentes
- `inDegreeCentrality` / `outDegreeCentrality` — volume de voos recebidos e emitidos
- `louvainId` — cluster/comunidade operacional

---

## 🚀 Casos de Uso

| # | Caso de Uso                    | Valor de Negócio                                                            |
|---|--------------------------------|-----------------------------------------------------------------------------|
| 1 | **Hubs Críticos**              | Identificar aeroportos cuja indisponibilidade causa maior impacto na rede   |
| 2 | **Roteamento com Escalas**     | Encontrar a rota mais curta entre quaisquer dois pontos, com N conexões     |
| 3 | **Comunidades Operacionais**   | Descobrir clusters naturais de operação via algoritmo Louvain               |
| 4 | **Cobertura Territorial**      | Mapear municípios atendidos e identificar gaps de mercado                   |
| 5 | **Performance por Empresa**    | Comparar volume, sobreposições e rotas exclusivas por subsidiária           |

---

## 🔍 Consultas de Demonstração

### Top 5 Hubs Mais Críticos
```cypher
MATCH (a:Aeroporto)
WHERE a.betweennessCentrality > 0
RETURN a.aeroportoNome AS hub, a.aeroportoICAO AS icao,
       round(a.betweennessCentrality) AS criticidade
ORDER BY a.betweennessCentrality DESC LIMIT 5
```

### Caminho Mínimo entre Dois Aeroportos
```cypher
MATCH (sp:Aeroporto { aeroportoICAO: 'SBGR' }),
      (lim:Aeroporto { aeroportoICAO: 'SPJC' })
MATCH path = shortestPath((sp)-[:CONECTADO_A*..8]->(lim))
RETURN [n IN nodes(path) | n.aeroportoICAO] AS rota,
       length(path) AS escalas
```

### Rotas por Empresa Aérea
```cypher
MATCH (r:Rota)-[:ASSOCIADA_A_EMPRESA_AEREA]->(e:`Empresa Aerea`)
RETURN e.eaereaNome AS empresa, count(r) AS rotas
ORDER BY rotas DESC
```

### Rede de Conexões de um Hub (Visualização)
```cypher
MATCH (bog:Aeroporto { aeroportoICAO: 'SKBO' })
-[c:CONECTADO_A]->(vizinho:Aeroporto)
RETURN bog, c, vizinho
LIMIT 30
```

### Aeroportos por Cluster Operacional
```cypher
WITH 206 AS clusterId  // 206=Brasil, 609=Colombia/EUA, 508=Chile
MATCH (a:Aeroporto { louvainId: clusterId })
OPTIONAL MATCH (a)-[:ASSOCIADO_A_PAIS]->(p:Pais)
RETURN a.aeroportoNome AS aeroporto, a.aeroportoICAO AS icao,
       p.paisNome AS pais, a.inDegreeCentrality AS volume_rotas
ORDER BY a.inDegreeCentrality DESC
```

### Gaps de Mercado (Aeroportos Ativos sem Rota Direta)
```cypher
MATCH (a:Aeroporto)
WHERE a.aeroportoSituacao = 'ATIVO'
  AND NOT (a)-[:CONECTADO_A]->(:Aeroporto)
OPTIONAL MATCH (a)-[:ASSOCIADO_A_PAIS]->(p:Pais)
RETURN a.aeroportoNome AS aeroporto, a.aeroportoICAO AS icao,
       p.paisNome AS pais LIMIT 20
```

---

## 📁 D
