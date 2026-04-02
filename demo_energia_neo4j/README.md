# ⚡ Energy 360 — Rede Elétrica Inteligente com Neo4j

> Plataforma de gestão e análise de redes elétricas construída sobre **Neo4j**, que combina monitoramento de ativos de geração e transmissão, análise de impacto na rede, visão 360° do cliente e roteamento inteligente entre equipamentos — tudo modelado como um grafo georreferenciado.

---

## 📋 Sobre o Projeto

O **Energy 360** é uma solução de demonstração que mostra como bancos de dados de grafos resolvem problemas críticos do setor de energia que são extremamente complexos com abordagens tradicionais.

Uma rede elétrica é, por natureza, um grafo: geradores conectam-se a barramentos (busbars), que se conectam a links de transmissão, que alimentam estações e, por fim, chegam às instalações dos clientes. Modelar essa topologia como um grafo permite consultas que seriam impraticáveis em bancos relacionais — como encontrar o caminho mais curto entre dois equipamentos, analisar o raio de impacto de uma falha ou rastrear toda a cadeia desde o gerador até o consumidor final.

### Por que Grafos para Energia?

- **Topologia nativa da rede** — a rede elétrica já é um grafo; modelá-la assim elimina JOINs complexos e permite navegação em tempo real
- **Análise de impacto** — determinar instantaneamente quais clientes são afetados por uma falha em qualquer ponto da rede
- **Roteamento inteligente** — encontrar o caminho mais curto entre equipamentos usando algoritmos nativos de grafos (Shortest Path)
- **Georreferenciamento** — cada nó possui coordenadas geográficas, permitindo visualizações em mapa e análises espaciais
- **Jornada do cliente** — encadear consumo, tickets e pesquisas de satisfação como uma jornada temporal conectada

---

## 🗂️ Modelo de Dados

O grafo Energy 360 modela a rede elétrica de ponta a ponta:

### Nós (Labels)

| Label | Descrição | Propriedades-chave |
|---|---|---|
| `Bus` | Barramento (busbar) da rede elétrica | `bus_id`, `country`, `under_construction`, `geometry` |
| `Generator` | Gerador de energia | `country`, `geometry` |
| `Link` | Link de transmissão entre barramentos | `country_1`, `length_m_new`, `geometry` |
| `Station` | Estação / subestação | — |
| `Sensor` | Sensor IoT instalado nos equipamentos | `id`, `type` (TEMPERATURE, etc.) |
| `Alert` | Alerta gerado por sensores | `id`, `type`, `status` (ACTIVE, etc.) |
| `MaintenanceRecord` | Registro de manutenção | `id`, `Type` (Corretivo/Preventivo), `description_en` |
| `Customer` | Cliente consumidor de energia | `name` |
| `Installation` | Instalação elétrica do cliente | — |
| `InstallationType` | Tipo de instalação | `name` |
| `Ticket` | Chamado de suporte do cliente | `createdDate` |
| `Survey` | Pesquisa de satisfação (NPS) | `npsGroup` (Promoter/Neutral/Detractor) |
| `Consumption` | Registro de consumo do cliente | — |

### Relacionamentos

| Relacionamento | De → Para | Descrição |
|---|---|---|
| `CONNECTED` | Bus ↔ Link ↔ Bus | Topologia da rede elétrica |
| `CONNECTED` | Generator → Bus | Gerador conectado a um barramento |
| `IN_STATION` | Bus → Station | Barramento pertence a uma estação |
| `LINK_HAS_INSTALLATION` | Link → Installation | Link alimenta uma instalação |
| `CUSTOMER_HAS_INSTALLATION` | Customer → Installation | Cliente possui uma instalação |
| `INSTALL_HAS_INSTALLTYPE` | Installation → InstallationType | Tipo da instalação |
| `CREATED_TICKET` | Customer → Ticket | Cliente abriu um chamado |
| `COMPLETED_SURVEY` | Customer → Survey | Cliente respondeu pesquisa NPS |
| `HAS_MAINTENANCE_RECORD` | Equipment → MaintenanceRecord | Equipamento possui registro de manutenção |
| `FIRST` | Installation → Consumption | Primeiro registro de consumo |
| `NEXT` | Consumption → Consumption | Próximo registro de consumo (jornada temporal) |

---

## 📊 Dashboards

O projeto inclui um dashboard interativo para **NeoDash** com 4 páginas:

### 1. Energy Network & Assets

Visão geral dos ativos da rede elétrica com KPIs e mapas georreferenciados:

- **Coverage** — gauge mostrando a cobertura da rede (96%)
- **Stations** (83), **Generators** (62), **Busbars** (97), **Links** (122) — contadores dos ativos
- **Network Size** — extensão total da rede em quilômetros (177.749 km)
- **Network Map** — mapa interativo com barramentos e geradores georreferenciados, conectados por links de transmissão
- **Maintenance Map** — mapa de calor (heatmap) mostrando a concentração geográfica das manutenções
- **Latest Maintenances** — tabela com registros de manutenção (corretiva e preventiva)
- **Mean Time Between Failures (MTBF)** — indicador de confiabilidade por equipamento
- **Sensors** — inventário de sensores IoT (362 sensores de temperatura)
- **Latest Alerts** — alertas ativos (HIGH_LEVEL_VIBRATION, HIGH_TEMPERATURE, etc.)

### 2. Network — Impact Analysis

Análise de impacto a partir de um endereço geográfico:

- **Geocodificação** — o usuário informa um endereço e o sistema localiza o barramento mais próximo usando `apoc.spatial.geocode`
- **Mapa de Análise** — visualização georreferenciada do ponto de impacto e rede ao redor
- **Shortest Path to Generator** — caminho mais curto entre o ponto selecionado e o gerador que o alimenta
- **Workspace Explore** — integração com Neo4j Workspace para análise visual aprofundada

### 3. Customer 360

Visão completa do cliente de energia:

- **Average NPS Score** — Net Promoter Score médio calculado via Cypher (Promoters − Detractors)
- **Connected Installations** — total de instalações conectadas à rede
- **Tickets** e **Surveys** — contadores de chamados e pesquisas
- **Tickets per Installation Type** — gráfico de pizza com distribuição de chamados por tipo de instalação
- **Evolution Tickets** — gráfico de barras com evolução mensal dos chamados
- **Ranking Customer NPS** — tabela com ranking de clientes por NPS
- **Customer Journey** — grafo mostrando a jornada completa do cliente: `Customer → Installation → FIRST → NEXT → ... → Consumption`

### 4. Shortest Path between Equipments

Roteamento inteligente entre dois equipamentos da rede:

- **Seletores de Busbar** — o usuário escolhe dois barramentos (Bus 1 e Bus 2)
- **Grafo do Caminho** — visualização do caminho mais curto entre os barramentos usando `shortestPath`
- **Georeferencing Details** — o mesmo caminho renderizado no mapa com coordenadas reais
- **Hops** — número de saltos (equipamentos intermediários) no caminho

---

## 🧠 Tecnologias e Algoritmos

- **Neo4j** — banco de dados de grafos
- **NeoDash** — dashboards interativos open-source para Neo4j
- **APOC** — procedimentos auxiliares:
  - `apoc.spatial.geocode` — geocodificação de endereços
  - `apoc.create.vRelationship` / `apoc.create.vNode` — nós e relacionamentos virtuais para visualização
- **Shortest Path** — algoritmo nativo do Cypher para roteamento entre equipamentos
- **Dados georreferenciados** — coordenadas `point()` em barramentos, geradores e links para visualização em mapa

---

## ⚙️ Setup

### Pré-requisitos

- Credenciais de acesso ao servidor Neo4j
- Navegador web moderno (Chrome, Firefox, Edge)

### Passo 1 — Acessar o NeoDash

Abra o NeoDash no navegador:

> 🔗 **[https://neodash.graphapp.io/](https://neodash.graphapp.io/)**

### Passo 2 — Conectar ao Servidor

Na tela de conexão do NeoDash, preencha os campos:

| Campo | Valor |
|---|---|
| **Connection URL** | `neo4j+s://neo4jlatampartners.com:7687` |
| **Username** | *(seu usuário fornecido pelo administrador)* |
| **Password** | *(sua senha fornecida pelo administrador)* |

Clique em **Connect**.

### Passo 3 — Selecionar o Banco de Dados `energy`

Após conectar, o NeoDash exibirá a barra lateral esquerda. Clique no seletor de banco de dados (dropdown) e escolha o banco **`energy`** na lista:

```
 ┌─────────────────────┐
 │  aerolatam           │
 │  datalineage         │
 │ ▸ energy ◂           │  ← Selecione este
 │  fsi360              │
 │  fundosdemo          │
 │  neo4j               │
 │  ...                 │
 └─────────────────────┘
```

### Passo 4 — Abrir o Dashboard

Após selecionar o banco `energy`, o dashboard **"Energy Demo - Cockpit"** será carregado automaticamente na barra lateral. Clique nele para abrir.

Você verá as 4 abas de navegação no topo:

1. **Energy Network & Assets**
2. **Network - Impact Analysis**
3. **Customer 360**
4. **Shortest Path between equipments**

> 💡 **Alternativa — Importar manualmente:** caso o dashboard não apareça automaticamente, clique no ícone **📂 Load** na barra superior do NeoDash e selecione o arquivo `dashboard.json` disponível neste repositório.

### Parâmetros Interativos

O dashboard utiliza parâmetros que podem ser ajustados diretamente nos widgets de seleção:

| Parâmetro | Tipo | Exemplo | Usado em |
|---|---|---|---|
| `neodash_country` | string | `PT` | Filtro de país (Network & Assets) |
| `neodash_bus1` | number | `1517` | Busbar de origem (Shortest Path) |
| `neodash_bus2` | number | `1807` | Busbar de destino (Shortest Path) |
| `neodash_customer` | string | `Curtis Diaz` | Cliente selecionado (Customer 360) |
| `neodash_addr` | string | `Rua de Campolide, Lisboa` | Endereço para análise de impacto |

---

## 📁 Estrutura do Repositório

```
.
├── README.md              # Este arquivo
├── dashboard.json         # Dashboard NeoDash (pronto para importar)
└── docs/
    └── screenshots/       # Capturas de tela do dashboard
```

---

## 📄 Licença

Este projeto é disponibilizado para fins de demonstração e aprendizado.
