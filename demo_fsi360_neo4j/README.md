# 🏦 FSI 360 — Visão 360° do Cliente com Neo4j

> Plataforma de análise de clientes do setor financeiro (FSI) construída sobre **Neo4j**, que combina visão 360° do cliente, sistema de recomendação híbrido, detecção de redes de conluio e identificação de pagamentos circulares — tudo em um único grafo.

---

## 📋 Sobre o Projeto

O **FSI 360** é uma solução de demonstração que mostra como bancos de dados de grafos resolvem problemas críticos do setor financeiro que são difíceis (ou impossíveis) de endereçar com bancos de dados relacionais tradicionais.

A solução modela clientes (`Consumer`), suas transações (`Transaction`), produtos (`Product`), dispositivos (`Device`), localizações (`Location`), chamados de suporte (`SupportTicket`) e métricas (`MetricSnapshot`) como nós em um grafo, conectados por relacionamentos que capturam a riqueza das interações do mundo real.

### Por que Grafos?

Instituições financeiras lidam com dados inerentemente conectados — clientes compartilham dispositivos, fazem transações entre si, utilizam os mesmos produtos e interagem com suporte. Modelar essas conexões como um grafo permite:

- **Visão 360° real** — não apenas juntar tabelas, mas navegar relacionamentos em tempo real
- **Detecção de fraude por padrões topológicos** — identificar redes de conluio e pagamentos circulares que são invisíveis em SQL
- **Recomendações baseadas em jornada** — usar a similaridade de trajetórias completas, não apenas atributos isolados
- **Análise de jornada temporal** — encadear eventos com `FIRST → NEXT → ... → LAST` para entender o comportamento sequencial do cliente

---

## 🗂️ Modelo de Dados

O grafo FSI 360 é composto pelas seguintes entidades e relacionamentos:

### Nós (Labels)

| Label | Descrição | Propriedades-chave |
|---|---|---|
| `Consumer` | Cliente da instituição financeira | `consumer_id`, `email_domain`, `is_pro`, `registered_at`, `collusion_component_id` |
| `Transaction` | Transação financeira realizada | `paid_total_value`, `transaction_date`, `is_pix`, `historical_at` |
| `Product` | Produto financeiro | `product_id`, `product_name` |
| `ProductType` | Categoria de produto | `product_type_name` |
| `Device` | Dispositivo utilizado pelo cliente | `device_model`, `device_os`, `collusion_component_id` |
| `Location` | Localização geográfica do cliente | — |
| `SupportTicket` | Chamado de suporte | `ticket_id`, `severity`, `topic`, `resolved_at`, `is_problematic_case` |
| `MetricSnapshot` | Snapshot de métricas do cliente | — |
| `Date` | Nó de data (para agregações temporais) | `date` |
| `Email` | E-mail associado ao cliente | — |

### Relacionamentos

| Relacionamento | De → Para | Descrição |
|---|---|---|
| `FROM` | Consumer → Transaction | Cliente originou a transação |
| `TO` | Transaction → Consumer | Transação destinada a um cliente |
| `OF_PRODUCT` | Transaction → Product | Transação referente a um produto |
| `HAS_TYPE` | Product → ProductType | Produto pertence a uma categoria |
| `USES_PRODUCT` | Consumer → Product | Cliente utiliza um produto (`count` = quantidade) |
| `HAS_DEVICE` | Consumer → Device | Cliente usa um dispositivo |
| `AT_LOCATION` | Consumer → Location | Cliente está em uma localização |
| `HAS_EMAIL` | Consumer → Email | E-mail do cliente |
| `OPENED_TICKET` | Consumer → SupportTicket | Cliente abriu chamado de suporte |
| `TRANSACTION_DATE` | Transaction → Date | Data da transação |
| `FIRST` | Consumer → Event | Primeiro evento da jornada do cliente |
| `LAST` | Consumer → Event | Último evento da jornada do cliente |
| `NEXT` | Event → Event | Próximo evento na jornada (`weight` = 1/(1+dias)) |
| `SIMILAR_JOURNEY` | Consumer ↔ Consumer | Jornadas similares (`similarity_score`) |

---

## 📊 Dashboards

O projeto inclui um dashboard interativo para **Neo4j Aura Dashboards** com 5 páginas (cockpits):

### 1. Cockpit — Visão 360°

Visão completa de um cliente selecionado:

- **Grafo 360°** — visualização de todos os relacionamentos diretos do cliente (transações, produtos, dispositivos, chamados, etc.)
- **Total de Transações** e **Valor Movimentado** — KPIs do cliente
- **Portfólio de Produtos** — tabela com tipos de produto e contagem de transações
- **Movimentações** — gráfico de barras com evolução temporal do valor movimentado
- **Últimos Chamados** — tabela com os chamados de suporte mais recentes
- **Últimas 4 Interações** — grafo hierárquico mostrando os eventos mais recentes via cadeia `LAST → NEXT`
- **Clientes Similares** — tabela com clientes de jornada similar e seus scores de similaridade

### 2. Cockpit — Recomendações

Sistema de recomendação híbrido que combina 5 estratégias com pesos configuráveis:

| # | Estratégia | Peso | Descrição |
|---|---|---|---|
| 1 | Collaborative Filtering | 40% | Produtos comprados por clientes com jornadas similares (via embeddings FastRP + Cosine Similarity) |
| 2 | Popularidade | 20% | Produtos mais comprados na base geral |
| 3 | Segmentação por Perfil | 15% | Produtos populares entre clientes do mesmo perfil (PRO vs. não-PRO) |
| 4 | Categoria de Interesse | 15% | Produtos de categorias que o cliente já consome |
| 5 | Recência | 10% | Produtos com compras nos últimos 30 dias |

A similaridade de jornada considera a **topologia do grafo** (sequência e tipos de eventos), o **timing entre eventos** (peso = 1/(1+dias)) e a **proporção de tipos de eventos** (transações vs. suporte vs. métricas).

Inclui visualização completa da **jornada do cliente** como grafo hierárquico (`FIRST → NEXT → ... → LAST`).

### 3. Cockpit — Rede de Conluio

Detecção de redes fraudulentas usando algoritmos de grafos:

- **Redes de Conluio** — quantidade de redes identificadas (componentes com 5+ membros)
- **Sharing Devices** — clientes compartilhando dispositivos
- **Sharing Devices e Location** — clientes compartilhando dispositivos E localização
- **Total Movimentado** — volume financeiro movimentado pelas redes
- **Grafo da Rede** — visualização do componente de conluio selecionado
- **Score de Fraude** — tabela com ranking das redes por score ponderado:
  - Nº de membros × 25
  - Devices compartilhados × 15
  - Endereços compartilhados × 15
  - PIX internos × 10
  - Registros coordenados × 20
- **Ação Recomendada**: 🔴 Bloqueio Sugerido (≥275) | 🟠 Investigação (≥150) | 🟡 Monitorar

### 4. Cockpit — Fraude e PLD (Prevenção à Lavagem de Dinheiro)

Detecção de pagamentos circulares via PIX usando **Quantified Path Patterns (QPP)** do Cypher:

- **Pagamentos Circulares** — identifica ciclos de 4 a 5 níveis onde o dinheiro retorna ao remetente original, no mesmo dia, via PIX
- **Grafo dos Ciclos** — visualização interativa dos padrões circulares encontrados
- **Top Usuários em Ciclos** — ranking dos clientes que mais aparecem em ciclos suspeitos
- **Total Movimentado** — valor total envolvido nos pagamentos circulares

### 5. Timeline de Transações

Gráfico de linha mostrando a evolução do volume de transações ao longo do tempo.

---

## 🧠 Tecnologias e Algoritmos

- **Neo4j Aura** — banco de dados de grafos na nuvem
- **Neo4j Aura Dashboards** — dashboards interativos com Cypher
- **GDS (Graph Data Science)**:
  - **FastRP** — geração de embeddings de jornadas de clientes
  - **Cosine Similarity** — cálculo de similaridade entre jornadas
  - **Weakly Connected Components (WCC)** — identificação de redes de conluio
- **Quantified Path Patterns (QPP)** — detecção de ciclos de pagamento (Cypher 5+)

---

## ⚙️ Setup

### Pré-requisitos

- Acesso ao Aura console (pode ser a versão Free)
- Os dados do FSI 360 carregados no banco (database: `fsi360`)

### Importar o Dashboard

O dashboard está disponível no arquivo `FSI_360_Demo_Dashboard.json` deste repositório. Para importá-lo no Neo4j Aura:

1. Acesse sua instância no [Neo4j Aura Console](https://console.neo4j.io)
2. Navegue até a seção **Dashboards**
3. Conect se ao ambiente de parceceiros. (neo4j+s://neo4jlatampartners.com:7687, Seu usuário e Senha)
4. Clique em **Import** (ícone de upload)
5. Selecione o arquivo `FSI_360_Demo_Dashboard.json`
6. O dashboard será carregado com todas as 5 páginas e widgets configurados

> 📖 Para instruções detalhadas sobre importação de dashboards, consulte a documentação oficial:
> **[Neo4j Aura — Import Dashboards](https://neo4j.com/docs/aura/dashboards/import/)**

### Parâmetros do Dashboard

Após importar, o dashboard utiliza os seguintes parâmetros interativos:

| Parâmetro | Tipo | Valor Padrão | Descrição |
|---|---|---|---|
| `consumer_consumer_id` | string | `43e94ae2e18c261e` | ID do cliente selecionado |
| `PesoSimilaridade` | number | `0.4` | Peso do collaborative filtering no sistema de recomendação |
| `consumer_collusion_component_id` | string | — | ID do componente de conluio para visualização |
| `nivel` | number | — | Nível de profundidade para análise de ciclos |

---

## 📁 Estrutura do Repositório

```
.
├── README.md                          # Este arquivo
├── FSI_360_Demo_Dashboard.json        # Dashboard exportado (pronto para importar)
└── ...
```

---

## 📄 Licença

Este projeto é disponibilizado para fins de demonstração e aprendizado.
