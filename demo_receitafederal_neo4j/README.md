# 🏢 Neo4j para o CNPJ — Sócios, Grupos Econômicos, Compliance e Doações Eleitorais

> Material de pré-venda Neo4j para análise do Cadastro Nacional da Pessoa Jurídica, demonstrando capacidades e diferenciais para Compliance, Detecção de Fraude, PPE, Inteligência de Mercado e Transparência Eleitoral.

---

## 📋 Sobre o Projeto

Este repositório contém scripts Cypher, consultas de demonstração e documentação de pré-venda baseados no grafo do **CNPJ completo da Receita Federal**, enriquecido com dados eleitorais, societários, geoespaciais e de sanções.

O material cobre os casos de uso de **Sócios e Empresas, Grupos Econômicos, Compliance, Detecção de Fraude, PPE e Doações Eleitorais** — demonstrando as capacidades e diferenciais do Neo4j para audiências técnicas e de negócios no setor público e privado.

---

## 📊 Visão Geral do Modelo

| Entidade              | Volume         | Detalhe                                          |
|-----------------------|----------------|--------------------------------------------------|
| Empresa               | 52,9 milhões   | cnpj, nome, natureza jurídica, porte, capital social, centralidade |
| Estabelecimento       | 55,8 milhões   | cnpj, nome fantasia, data início atividade       |
| Situacao Cadastral    | 55,8 milhões   | código, data, motivo — status ativo/inativo      |
| CNAE                  | 55,8 milhões   | código primário/secundário, atividade econômica  |
| Endereco / CEP        | 54,6 milhões   | logradouro, bairro, CEP, coordenadas POINT       |
| Simples / MEI         | 34,7 mi / 14,9 mi | regime tributário com datas de opção/exclusão |
| Email                 | 33,3 milhões   | e-mail de contato do estabelecimento             |
| Sociedade             | 18,4 milhões   | cnpj sócio, qualificação, data entrada           |
| Pessoa                | 15,2 milhões   | nome, CPF/identificador, faixa etária            |
| Lancamento Eleicao    | 3,2 milhões    | valor, data, descrição                           |
| Documento Eleicao     | 1,8 milhão     | tipo despesa, valor, data                        |
| Fornecedor Eleicao    | 1,2 milhão     | nome, CNPJ, CNAE, município, UF                  |
| Doador                | 137 mil        | nome, tipo (F/J), CNPJ/CPF, CNAE                 |
| Candidato Eleicao     | 22 mil         | nome, CPF, número, turno, sequencial             |

**Relacionamentos principais:**

| Relacionamento                    | Semântica                                              |
|-----------------------------------|--------------------------------------------------------|
| TEM_ESTABELECIMENTO               | Empresa possui Estabelecimento                         |
| TEM_CNAE / TEM_SITUACAO_CADASTRAL | Estabelecimento tem CNAE e situação cadastral          |
| TEM_ENDERECO / TEM_CEP            | Estabelecimento tem Endereço geocodificado             |
| ASSOCIADO_A_GEOLOCALIZACAO        | CEP mapeado para ponto geoespacial (POINT)             |
| TEM_SIMPLES / TEM_MEI             | Regime tributário da empresa                           |
| TEM_SOCIEDADE_PESSOA              | Pessoa é sócia de Sociedade                            |
| ASSOCIADA_A_EMPRESA               | Sociedade vinculada a Empresa                          |
| TEM_RELACAO_EMPRESA               | Empresa relacionada a outra (grupo econômico)          |
| ASSOCIADO_A_EMPRESA_DOADOR        | Doador eleitoral vinculado a Empresa pelo CNPJ         |
| ESTA_ASSOCIADO_A_EMPRESA_CANDIDATO| Candidato vinculado a Empresa eleitoral                |
| TEM_EMPRESA_FORNECEDOR_ELEICAO    | Fornecedor de campanha vinculado a Empresa CNPJ        |

---

## 🗂️ Modelo de Dados

```
(Pais) <-[:ESTA_EM_PAIS]- (Estado) <-[:PARTE_DE_ESTADO]- (Municipio)
                                                               ↑
(CEP)-[:ESTA_EM_MUNICIPIO]─────────────────────────────────→ |
(CEP)-[:ASSOCIADO_A_GEOLOCALIZACAO]────────────────────→ (Geolocalizacao)
(Endereco)-[:TEM_CEP]──────────────────────────────────→ (CEP)
(Estabelecimento)-[:TEM_ENDERECO]──────────────────────→ (Endereco)
(Estabelecimento)-[:TEM_CNAE]──────────────────────────→ (CNAE)
(Estabelecimento)-[:TEM_SITUACAO_CADASTRAL]────────────→ (Situacao Cadastral)
(Empresa)-[:TEM_ESTABELECIMENTO]───────────────────────→ (Estabelecimento)
(Empresa)-[:TEM_SIMPLES / TEM_MEI]─────────────────────→ (Simples / MEI)
(Empresa)-[:TEM_SOCIEDADE_EMPRESA]─────────────────────→ (Sociedade)
(Empresa)-[:TEM_RELACAO_EMPRESA]───────────────────────→ (Empresa)  ← grupo econômico
(Pessoa)-[:TEM_SOCIEDADE_PESSOA]───────────────────────→ (Sociedade)
(Sociedade)-[:ASSOCIADA_A_EMPRESA]─────────────────────→ (Empresa)
(Doador)-[:ASSOCIADO_A_EMPRESA_DOADOR]─────────────────→ (Empresa)
(Doador)-[:ASSOCIADO_A_CANDIDATO_ELEICAO_DOADOR]───────→ (Candidato Eleicao)
(Candidato)-[:ESTA_ASSOCIADO_A_EMPRESA_CANDIDATO]──────→ (Empresa)
```

---

## 🚀 Casos de Uso

| # | Caso de Uso                               | Valor de Negócio                                                                                 |
|---|-------------------------------------------|--------------------------------------------------------------------------------------------------|
| 1 | **Mapeamento de Sócios e Redes**          | Encontra todas as empresas de uma pessoa e suas qualificações societárias em segundos             |
| 2 | **Identificação de Grupos Econômicos**    | Mapeia estruturas de controle, holdings e consórcios com qualquer profundidade de aninhamento     |
| 3 | **Detecção de Fraude e Compliance**       | Detecta sócios em comum entre concorrentes em licitações e vínculos com sanções                   |
| 4 | **Pessoa Politicamente Exposta (PPE)**    | Cruza candidatos com o CNPJ revelando conflitos de interesse e vínculos empresariais              |
| 5 | **Análise de Doações Eleitorais**         | Conecta doadores, candidatos e fornecedores eleitorais ao cadastro CNPJ em uma única query        |
| 6 | **Inteligência de Mercado Geoespacial**   | 52 milhões de endereços geocodificados — densidade empresarial por região, CNAE e porte           |

---

## 🔍 Consultas de Demonstração

### Rede Societária Completa de uma Pessoa
```cypher
WITH 'JOSE CARLOS DA SILVA' AS nomePessoa
MATCH (p:Pessoa {pessoaNome: nomePessoa})
-[:TEM_SOCIEDADE_PESSOA]->(s:Sociedade)
-[:ASSOCIADA_A_EMPRESA]->(e:Empresa)
RETURN p.pessoaNome AS pessoa, e.empresaNome AS empresa,
       e.empresaCnpj AS cnpj, s.sociedadeQualinome AS qualificacao
ORDER BY e.empresaNome
```

### Top Empresas por Centralidade (Hubs da Rede)
```cypher
MATCH (e:Empresa)
WHERE e.empresaDegreecent > 500
RETURN e.empresaNome AS empresa, e.empresaCnpj AS cnpj,
       e.empresaDegreecent AS centralidade, e.empresaNjdesc AS tipo
ORDER BY e.empresaDegreecent DESC LIMIT 10
```

### Sócios em Comum entre Duas Empresas
```cypher
WITH '61065751' AS cnpj1, '08343492' AS cnpj2
MATCH (e1:Empresa {empresaCnpj: cnpj1})<-[:ASSOCIADA_A_EMPRESA]-(s1:Sociedade)
      <-[:TEM_SOCIEDADE_PESSOA]-(p:Pessoa)
      -[:TEM_SOCIEDADE_PESSOA]->(s2:Sociedade)
      -[:ASSOCIADA_A_EMPRESA]->(e2:Empresa {empresaCnpj: cnpj2})
RETURN e1.empresaNome AS empresa1, e2.empresaNome AS empresa2,
       p.pessoaNome AS socio_em_comum, 'ALERTA: Socio compartilhado' AS status
```

### Doadores de um Candidato e seus Vínculos Empresariais
```cypher
WITH 'WELTON YUDI ODA' AS nomeCandidato
MATCH (ce:`Candidato Eleicao` {candidatoeleicaoNome: nomeCandidato})
<-[:ASSOCIADO_A_CANDIDATO_ELEICAO_DOADOR]-(d:Doador)
OPTIONAL MATCH (d)-[:ASSOCIADO_A_DOADOR_ELEICAO]->(doacao:Doacao)
RETURN d.doadorNome AS doador, sum(doacao.doacaoValor) AS total_doado
ORDER BY total_doado DESC
```

### Grupo Econômico — Empresas Relacionadas
```cypher
WITH '61065751' AS cnpjRaiz
MATCH (raiz:Empresa {empresaCnpj: cnpjRaiz})
MATCH path = (raiz)-[:TEM_RELACAO_EMPRESA*1..4]->(filial:Empresa)
RETURN raiz.empresaNome AS holding, filial.empresaNome AS empresa_relacionada,
       filial.empresaCnpj AS cnpj, length(path) AS nivel
ORDER BY nivel, filial.empresaNome
```

### Empresas de Saúde por Município
```cypher
MATCH (e:Empresa)-[:TEM_ESTABELECIMENTO]->(est:Estabelecimento)
-[:TEM_CNAE]->(c:CNAE)-[:TEM_ENDERECO]->(end:Endereco)
WHERE c.cnaeCodigoprimario STARTS WITH '86'
RETURN end.enderecoMunicipio AS municipio, count(DISTINCT e) AS empresas
ORDER BY empresas DESC LIMIT 15
```

---

## 📁 Documentação

| Arquivo | Descrição |
|---------|-----------|
| [`Neo4j_PreVenda_CNPJ.pdf`](./Neo4j_PreVenda_CNPJ.pdf) | Documento completo de pré-venda com modelo de dados, 6 casos de uso, scripts comentados, queries de demo e comparativo Neo4j vs SQL |

O documento inclui:
- ✅ Inventário completo do modelo de dados com volumes reais
- ✅ Texto explicativo para audiência não-técnica (compliance, gestores públicos, investigadores)
- ✅ 4 scripts Cypher reutilizáveis e parametrizados
- ✅ 6 consultas prontas para demonstração ao vivo no Neo4j Browser
- ✅ Tabela comparativa Neo4j vs SQL para argumentação em pré-venda

---

## 📌 Empresas com Maior Centralidade no Grafo

| Empresa                        | CNPJ     | Centralidade | Tipo                   |
|--------------------------------|----------|--------------|------------------------|
| MEDICALMAIS SERVICOS EM SAUDE  | 21609217 | 1.108        | Soc. Empresária Ltda   |
| LASER FAST DEPILACAO           | 31237773 | 1.101        | Soc. Empresária Ltda   |
| ROSSI RESIDENCIAL SA           | 61065751 | 1.045        | S.A. Aberta            |
| MRV ENGENHARIA E PARTICIPACOES | 08343492 | 947          | S.A. Aberta            |
| CYRELA BRAZIL REALTY S.A.      | 73178600 | 664          | S.A. Aberta            |

---

## ⚖️ Neo4j vs. SQL para CNPJ — Resumo

| Capacidade                    | SQL / Bases Relacionais                | Neo4j                                        |
|-------------------------------|----------------------------------------|----------------------------------------------|
| Rede societária N níveis      | Recursão SQL — impraticável em escala  | `TEM_SOCIEDADE_PESSOA*1..N` — segundos       |
| Grupos econômicos             | Self-join em 52 mi de registros        | `TEM_RELACAO_EMPRESA` — resultado imediato   |
| Sócios em comum               | Múltiplos JOINs — inviável em produção | Caminho no grafo — milissegundos             |
| Cruzamento CNPJ × Eleições    | Sistemas isolados — integração manual  | Modelo unificado — query única               |
| Análise geoespacial           | Requer PostGIS ou ferramentas externas | POINT nativo — 52 mi de endereços            |
| Centralidade / Importância    | Não disponível nativamente             | `empresaDegreecent` pré-computado + GDS      |
| Visualização interativa       | BI externo necessário                  | Neo4j Bloom — explora o grafo sem código     |

---

## 🛠️ Tecnologias

- **Neo4j** — banco de dados de grafo
- **Cypher** — linguagem de consulta
- **Neo4j GDS** (Graph Data Science) — centralidade, detecção de comunidades, pathfinding
- **Neo4j Browser / Bloom** — visualização interativa do grafo CNPJ

---

