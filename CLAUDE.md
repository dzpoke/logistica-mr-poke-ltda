# TransCarga Brasil — Projeto Power BI Logística

Contexto do projeto para o Claude Code. Ler este arquivo antes de qualquer tarefa
relacionada a este projeto.

## Sobre o projeto

Dashboard Power BI para uma transportadora fictícia de **frota própria / carga
pesada** (TransCarga Brasil Transportes Ltda.), inspirado no layout de um
dashboard financeiro de construtora (KPIs no topo, tendência mensal, distribuição
por categoria, orçado vs. real, tabela por centro de custo), adaptado para o
segmento de logística com camada financeira **e** operacional.

## Base de dados

- **Arquivo:** `logistica_frota.db` (SQLite) + CSVs individuais equivalentes
- **Modelo:** esquema estrela, 3 fatos + 6 dimensões
- **Período coberto:** 01/01/2024 a 18/08/2025
- **Volume:** ~8.900 viagens, ~15.400 lançamentos de custo, 100 linhas de orçamento mensal

### Tabelas dimensão
| Tabela | Chave | Principais campos |
|---|---|---|
| `dim_calendario` | id_data (AAAAMMDD) | data, ano, mes, mes_nome, trimestre, dia_semana, fim_de_semana |
| `dim_filial` | id_filial | nome_filial, cidade, uf, regiao — 5 filiais (SP, PR, MG, BA, GO) |
| `dim_veiculo` | id_veiculo | placa, tipo_veiculo, marca, ano_fabricacao, capacidade_ton, consumo_medio_km_l, id_filial, status_frota |
| `dim_motorista` | id_motorista | nome, cnh_categoria, id_filial, data_admissao, score_seguranca |
| `dim_cliente` | id_cliente | nome_cliente, segmento, uf |
| `dim_rota` | id_rota | origem, destino, distancia_km |

### Tabelas fato
| Tabela | Grão | Principais campos |
|---|---|---|
| `fato_viagens` | 1 linha por viagem/frete | id_data, id_veiculo, id_motorista, id_cliente, id_rota, id_filial, peso_ton, valor_frete, km_rodado, tempo_previsto_h, tempo_realizado_h, status_entrega (No Prazo/Atrasado), ocorrencia |
| `fato_custos` | 1 linha por dia/filial/categoria | id_data, id_filial, categoria_custo (Combustível 45%, Manutenção 20%, Motorista 17%, Pedágio 10%, Seguro 5%, Outros 3%), valor |
| `fato_orcamento` | 1 linha por mês/filial | ano, mes, id_filial, receita_orcada, receita_real, custo_orcado, custo_real |

### Relacionamentos
```
fato_viagens.id_data     → dim_calendario.id_data
fato_viagens.id_veiculo  → dim_veiculo.id_veiculo
fato_viagens.id_motorista→ dim_motorista.id_motorista
fato_viagens.id_cliente  → dim_cliente.id_cliente
fato_viagens.id_rota     → dim_rota.id_rota
fato_viagens.id_filial   → dim_filial.id_filial
fato_custos.id_data      → dim_calendario.id_data
fato_custos.id_filial    → dim_filial.id_filial
fato_orcamento.id_filial → dim_filial.id_filial
```

## Páginas do relatório (solicitação do cliente)

Cliente pediu 5 páginas, nesta ordem de prioridade:

### 1. Visão Geral (Executiva)
Página que a diretoria abre todo dia — "a saúde da empresa em 10 segundos".
- KPIs: Receita total, Custo total, Lucro bruto, Margem, % entregas no prazo
- Gráfico de linha: Receita vs. Custo mensal
- Rosca: status da frota (ativo/manutenção/inativo)
- Tabela comparativa rápida por filial (receita, custo, margem, SLA)

### 2. Financeiro
Foco em análise financeira detalhada e comparação entre filiais.
- KPIs: Receita, Custo, Lucro, Margem, Custo/km, Fluxo de caixa
- Gráfico de linha: Receita vs. Custo mensal
- Rosca: distribuição de custos por categoria (`fato_custos.categoria_custo`)
- Barras: Orçado vs. Real (receita e custo) — `fato_orcamento`
- Tabela: análise por filial com desvio orçado x real
- Filtros: filial, período

### 3. Operacional / SLA
Página do time de operações.
- KPIs: % no prazo, % atrasado, tempo médio de atraso, total de viagens
- Barras empilhadas: SLA no prazo x atrasado por mês
- Barras: ocorrências por tipo (`fato_viagens.ocorrencia`)
- Tabela: desempenho por rota (viagens, % no prazo, tempo médio, ocorrências)
- Filtros: filial, rota

### 4. Frota e Motoristas
- KPIs: veículos ativos, em manutenção, km rodado total, consumo médio, receita por veículo
- Barras: utilização (km rodado) por tipo de veículo
- Rosca: status da frota
- Tabela: ranking de motoristas por segurança e produtividade (score_seguranca, viagens, km rodado)
- Filtros: filial, tipo de veículo

### 5. Clientes e Rotas
- KPIs: clientes ativos, receita média por cliente, receita por km média, rotas monitoradas
- Barras: top 10 clientes por receita
- Rosca: receita por segmento de cliente (Agronegócio, Indústria, Varejo/Atacado, Construção Civil, Mineração, Bebidas)
- Tabela: rentabilidade por rota (distância, receita/km, % no prazo, viagens)
- Filtros: segmento, período

## Wireframes

Wireframes SVG de todas as 5 páginas já foram desenhados e aprovados
conceitualmente na conversa com o cliente (ver histórico do chat). Layout
segue o padrão: barra de filtros no topo → linha de cartões de KPI → 2
gráficos lado a lado → tabela detalhada na parte inferior.

## Convenções de nomenclatura (para DAX/Power Query)

- Medidas em português, PascalCase com espaços: `Receita Total`, `Custo Total`,
  `Margem de Lucro %`, `% Entregas No Prazo`
- Usar `dim_calendario` marcada como tabela de datas oficial (Mark as Date Table)
- Relacionamentos: sempre 1-para-muitos, direção única (dimensão → fato),
  exceto se surgir necessidade de filtro cruzado explícito

## Próximos passos possíveis
- Construção das medidas DAX (Receita Total, Custo Total, Margem, SLA %, etc.)
- Modelagem do relacionamento no Power BI (Model view)
- Ajuste de wireframe: considerar mapa geográfico na página de Rotas
- Exportar dados atualizados caso o volume/período mude