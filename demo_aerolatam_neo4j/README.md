# ✈️ Neo4j Aerolatam — Mapeamento de Rotas e Redes de Conexão

> Material de pré-venda Neo4j para o setor de aviação, gerado com dados reais do Grupo LATAM via conexão MCP.

---

## 📋 Sobre o Projeto

Este repositório contém scripts Cypher, consultas de demonstração e documentação de pré-venda produzidos a partir da inspeção direta do banco de dados **Aerolatam** via **Neo4j MCP Connection**.

O material cobre o caso de uso de **Mapeamento de Rotas e Redes de Conexão**, com foco em Logística, Supply Chain e Operações Aéreas — demonstrando as capacidades e diferenciais do Neo4j para audiências técnicas e de negócios.

---

## 📊 Dados do Banco

| Entidade         | Volume       |
|------------------|-------------|
| Rotas            | 125.527     |
| Aeroportos       | 612         |
| Países           | 32          |
| Estados/Províncias | 488       |
| Municípios       | 488         |
| Empresas Aéreas  | 10 (Grupo LATAM) |
| Relacionamentos  | ~504.000    |

**Algoritmos de grafo pré-computados** armazenados nos nós `Aeroporto`:
- `betweennessCentrality` — criticidade como ponto de passagem
- `eigenvectorCentrality` — importância considerando vizinhos influentes
- `inDegreeCentrality` / `outDegreeCentrality` — volume de voos
- `louvainId` — cluster/comunidade operacional

---

## 🗂️ Modelo de Dados

```
(Pais) <-[:ESTA_EM_PAIS]- (Estado) <-[:PARTE_DE_ESTADO]- (Municipio)
                                                               ↑
(Aeroporto) -[:SERVE_MUNICIPIO]-----------------------------→ |
(Aeroporto) -[:LOCALIZADO_EM_MUNICIPIO]------------------→ (Municipio)
(Aeroporto) -[:ASSOCIADO_A_PAIS]-------------------------→ (Pais)
(Aeroporto) -[:CONECTADO_A {conexoes}]-------------------→ (Aeroporto)
(Aeroporto) -[:TEM_ROTA_ORIGEM]-------------------------→ (Rota)
(Rota)      -[:TEM_ROTA_DESTINO]------------------------→ (Aeroporto)
(Rota)      -[:ASSOCIADA_A_EMPRESA_AEREA]---------------→ (Empresa Aerea)
```

---

## 🚀 Casos de Uso

| # | Caso de Uso | Valor de Negócio |
|---|-------------|-----------------|
| 1 | **Hubs Críticos** | Identificar aeroportos cuja indisponibilidade causa maior impacto na rede |
| 2 | **Roteamento com Escalas** | Encontrar a rota mais curta entre quaisquer dois pontos, com N conexões |
| 3 | **Comunidades Operacionais** | Descobrir clusters naturais de operação via algoritmo Louvain |
| 4 | **Cobertura Territorial** | Mapear municípios atendidos e identificar gaps de mercado |
| 5 | **Performance por Empresa** | Comparar volume, sobreposições e rotas exclusivas por subsidiária |

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

### Rotas por Empresa do Grupo LATAM
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

## 📁 Documentação

| Arquivo | Descrição |
|---------|-----------|
| [`Neo4j_PreVenda_Aerolatam.docx`](./Neo4j_PreVenda_Aerolatam.docx) | Documento completo de pré-venda com modelo de dados, casos de uso, scripts comentados, queries de demo e comparativo Neo4j vs SQL |

O documento inclui:
- ✅ Inventário completo do modelo de dados (nós, relacionamentos, propriedades)
- ✅ Texto explicativo para audiência não-técnica
- ✅ 4 scripts Cypher reutilizáveis e parametrizados
- ✅ 6 consultas prontas para demonstração ao vivo
- ✅ Tabela comparativa Neo4j vs SQL para argumentação em pré-venda

---

## 🏆 Destaque: Top Hubs da Rede LATAM

| Aeroporto | ICAO | Betweenness | Cluster |
|-----------|------|-------------|---------|
| Guarulhos | SBGR | 12.544 | Brasil (206) |
| Santiago | SCEL | 7.615 | Chile (508) |
| Lima Jorge Chávez | SPJC | 7.584 | Peru (498) |
| Bogotá El Dorado | SKBO | 4.416 | Colombia/EUA (609) |
| Miami | KMIA | 3.818 | Colombia/EUA (609) |

> Guarulhos é **65% mais crítico** que o segundo colocado — confirmado pelos dados reais do grafo.

---

## 🛠️ Tecnologias

- **Neo4j** — banco de dados de grafo
- **Cypher** — linguagem de consulta
- **Neo4j GDS** (Graph Data Science) — algoritmos de centralidade e comunidade
- **Neo4j MCP** — conexão e inspeção do banco via Model Context Protocol
- **Neo4j Browser / Bloom** — visualização interativa

---

## 📌 Empresas do Grupo LATAM no Banco

| Empresa | ICAO |
|---------|------|
| LATAM Brasil | TAM |
| LATAM Airlines | LAN |
| LATAM Colombia | ARE |
| LATAM Chile | LXP |
| LATAM Peru | LPE |
| LATAM Paraguay | LAP |
| LATAM Ecuador | LNE |
| LATAM Cargo | LCO |
| LATAM Cargo Brasil | LTG |
| LATAM Cargo Colombia | LAE |

---
