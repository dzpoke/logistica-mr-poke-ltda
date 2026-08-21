# Mr Poke Transpostes LTDA 

![alt text](image.png)

Dashboard de Business Intelligence para uma transportadora fictícia de **frota
própria / carga pesada** (TransCarga Brasil Transportes Ltda.), unindo a
camada **financeira** (receita, custo, margem, orçado vs. real) à camada
**operacional** de logística (SLA de entregas, ocorrências, frota, motoristas
e rotas).

O projeto foi inspirado no layout de um dashboard financeiro de construtora
(KPIs no topo, tendência mensal, distribuição por categoria, orçado vs. real,
tabela por centro de custo), adaptado para o segmento de transporte rodoviário
de carga pesada.

## Estrutura do repositório

| Caminho | Conteúdo |
|---|---|
| [`logistisca_v1.pbix`](logistisca_v1.pbix) | Arquivo Power BI com o modelo e o relatório |
| [`dataset/`](dataset/) | Base de dados (SQLite + CSVs) e documentação do modelo dimensional |
| [`layout/`](layout/) | Wireframes das páginas do relatório |
| [`CLAUDE.md`](CLAUDE.md) | Contexto do projeto para uso com Claude Code |

## Base de dados

- **Formato:** SQLite (`dataset/logistica_frota.db`) + CSVs individuais equivalentes (UTF-8)
- **Modelo:** esquema estrela — 3 tabelas fato + 6 tabelas dimensão
- **Período coberto:** 01/01/2024 a 18/08/2025
- **Volume:** ~8.900 viagens, ~15.400 lançamentos de custo, 100 linhas de orçamento mensal

### Modelo dimensional

```
                dim_calendario
                      │
dim_veiculo ── fato_viagens ── dim_cliente
                      │
dim_motorista ────────┼──────── dim_rota
                      │
                dim_filial
                      │
          fato_custos ┴ fato_orcamento
```

**Dimensões:** calendário, filial (centro de custo), veículo (frota),
motorista, cliente e rota.

**Fatos:**
- `fato_viagens` — 1 linha por viagem/frete (receita, peso, km, prazos, ocorrências)
- `fato_custos` — 1 linha por dia/filial/categoria de custo
- `fato_orcamento` — 1 linha por mês/filial (orçado vs. real)

Detalhamento completo dos campos e relacionamentos em [`dataset/README.md`](dataset/README.md).

## Páginas do relatório

O relatório foi planejado em 5 páginas, na seguinte ordem de prioridade:

1. **Visão Geral (Executiva)** — a saúde da empresa em 10 segundos: receita,
   custo, lucro, margem, % de entregas no prazo, receita vs. custo mensal,
   status da frota e comparativo rápido por filial.
2. **Financeiro** — análise financeira detalhada: KPIs de receita/custo/margem/
   fluxo de caixa, distribuição de custos por categoria, orçado vs. real e
   desvio por filial.
3. **Operacional / SLA** — desempenho de entregas: % no prazo/atrasado, tempo
   médio de atraso, ocorrências por tipo e desempenho por rota.
4. **Frota e Motoristas** — utilização da frota, consumo médio, ranking de
   motoristas por segurança e produtividade.
5. **Clientes e Rotas** — top clientes por receita, receita por segmento e
   rentabilidade por rota.

Todas as páginas seguem o mesmo padrão de layout: filtros no topo → cartões de
KPI → dois gráficos lado a lado → tabela detalhada. Os wireframes de cada
página estão em [`layout/`](layout/).

## Convenções do modelo (DAX / Power Query)

- Medidas em português, PascalCase com espaços (ex.: `Receita Total`,
  `Margem de Lucro %`, `% Entregas No Prazo`)
- `dim_calendario` marcada como tabela oficial de datas (*Mark as Date Table*)
- Relacionamentos 1-para-muitos, direção única (dimensão → fato)

## Status

Modelo de dados e wireframes definidos e aprovados. Próximas etapas:
construção das medidas DAX, modelagem dos relacionamentos no Power BI e
avaliação de um mapa geográfico na página de Rotas.