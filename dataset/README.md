# Base de Dados — Logística de Frota Própria (Carga Pesada)

**Empresa fictícia:** TransCarga Brasil Transportes Ltda.
**Segmento:** Transportadora rodoviária de carga pesada com frota própria
**Período:** 01/01/2024 a 18/08/2025
**Formato:** SQLite (`logistica_frota.db`) + CSVs individuais (UTF-8)

Modelo dimensional (estrela), pensado para alimentar um dashboard nos moldes do
exemplo "Controlo Financeiro": KPIs no topo (receita, custo, margem, fluxo de
caixa), tendência mensal, distribuição de custos, orçado vs. real e análise por
centro de custo — com a camada operacional de logística (SLA, ocorrências, frota).

## Tabelas dimensão

### dim_calendario
| Campo | Descrição |
|---|---|
| id_data | Chave (AAAAMMDD) |
| data | Data completa |
| ano, mes, mes_nome, trimestre | Atributos de tempo |
| dia_semana | Nome do dia da semana |
| fim_de_semana | Booleano |

### dim_filial (Centro de Custo)
id_filial, nome_filial, cidade, uf, regiao — 5 filiais (SP, PR, MG, BA, GO)

### dim_veiculo (Frota)
id_veiculo, placa, tipo_veiculo (Truck/Carreta/Bitrem/Rodotrem/Toco), marca,
ano_fabricacao, capacidade_ton, consumo_medio_km_l, id_filial, status_frota

### dim_motorista
id_motorista, nome, cnh_categoria (C/D/E), id_filial, data_admissao,
score_seguranca (indicador interno 0–10)

### dim_cliente
id_cliente, nome_cliente, segmento (Agronegócio, Indústria, Varejo/Atacado,
Construção Civil, Mineração, Bebidas), uf

### dim_rota
id_rota, origem, destino, distancia_km — 20 rotas entre praças relevantes do Brasil

## Tabelas fato

### fato_viagens (núcleo financeiro + operacional)
Uma linha por viagem/frete realizado.
| Campo | Descrição |
|---|---|
| id_viagem | Chave |
| id_data, id_veiculo, id_motorista, id_cliente, id_rota, id_filial | Chaves estrangeiras |
| peso_ton | Carga transportada |
| valor_frete | Receita da viagem (R$) |
| km_rodado | Distância da rota |
| tempo_previsto_h / tempo_realizado_h | Base para cálculo de SLA |
| status_entrega | "No Prazo" / "Atrasado" |
| ocorrencia | Nenhuma, Pane Mecânica, Trânsito/Rota, Atraso em Carga/Descarga, Problema Documental |

### fato_custos
Custos operacionais por dia/filial/categoria — equivalente ao gráfico
"Distribuição de Custos" do exemplo.
Categorias: Combustível (45%), Manutenção (20%), Motorista (17%), Pedágio (10%),
Seguro (5%), Outros (3%) — proporções aproximadas, com variação diária.

### fato_orcamento
Granularidade mensal por filial: receita_orcada, receita_real, custo_orcado,
custo_real — equivalente ao gráfico "Desempenho Orçamental".

## Sugestões de KPIs (equivalentes ao dashboard de referência)
- **Receita Total / Custo Total / Lucro Bruto / Margem de Lucro** → SUM(valor_frete), SUM(valor), diferença e %
- **Fluxo de Caixa** → Receita real – Custo real por período
- **Receita vs. Custo (mensal)** → agregação de fato_viagens e fato_custos por mês
- **Distribuição de Custos** → fato_custos agrupado por categoria_custo
- **Desempenho Orçamental** → fato_orcamento (orçado vs. real)
- **Análise por Centro de Custo** → agregações por id_filial
- **KPIs operacionais extras** → % No Prazo vs. Atrasado (SLA), custo por km rodado,
  km rodado por veículo, ocorrências por tipo, receita por tonelada

## Relacionamentos (chaves)
fato_viagens.id_data → dim_calendario.id_data
fato_viagens.id_veiculo → dim_veiculo.id_veiculo
fato_viagens.id_motorista → dim_motorista.id_motorista
fato_viagens.id_cliente → dim_cliente.id_cliente
fato_viagens.id_rota → dim_rota.id_rota
fato_viagens.id_filial / fato_custos.id_filial / fato_orcamento.id_filial → dim_filial.id_filial
fato_custos.id_data → dim_calendario.id_data
