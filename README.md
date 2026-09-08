# Registro de Decisões de Arquitetura (ADRs)

Este repositório documenta, no formato ADR (Architecture Decision
Record), as decisões estruturais tomadas ao longo dos Projetos 01-06
do meu portfólio de dados — formalizadas com o mesmo rigor que um
time com um arquiteto dedicado aplicaria. Cada ADR segue um template
fixo: contexto, opções consideradas (com prós e contras reais, não
só a opção vencedora), decisão e consequências — inclusive as
negativas.

Projeto 07 de uma série que documenta minha transição de Analista de
Dados para Arquitetura de Dados — veja o [perfil
completo](https://github.com/amanda-martins-data).

## Por que um repositório de ADRs

Um Engenheiro de Dados sênior toma boas decisões técnicas. Um
Arquiteto de Dados documenta o porquê de cada decisão de um jeito
que sobrevive à pessoa que a tomou — para que, meses depois, alguém
novo no time entenda por que o Silver é particionado por
`measured_date` sem precisar perguntar. Este repositório é esse
exercício aplicado retroativamente aos Projetos 01-06, mais algumas
decisões prospectivas — coisas ainda não implementadas, mas que sei
justificar tecnicamente.

## Estrutura

```
.
├── template/
│   └── adr-template.md         # o molde vazio, reutilizável
└── decisions/
    ├── 0001-data-lake-vs-warehouse-vs-lakehouse.md
    ├── 0002-particionamento-bronze-ingestion-vs-silver-measured.md
    ├── 0003-orquestracao-self-hosted-vs-gerenciada.md
    ├── 0004-nat-gateway-vs-vpc-endpoints.md
    ├── 0005-terraform-state-local-vs-remoto.md
    ├── 0006-limite-entre-logica-deterministica-e-llm.md
    ├── 0007-great-expectations-vs-framework-proprio.md
    ├── 0008-modelagem-star-schema-vs-data-vault-vs-obt.md
    ├── 0009-estrategia-evolucao-de-schema-em-parquet.md
    └── 0010-batch-vs-streaming-proxima-evolucao.md
```

## Índice dos ADRs

### Retroativos — decisões já implementadas nos Projetos 01-06

| ADR | Decisão | Projeto de origem |
|---|---|---|
| [0001](decisions/0001-data-lake-vs-warehouse-vs-lakehouse.md) | Data Lake (Parquet) em vez de Data Warehouse ou Lakehouse gerenciado | Projeto 03 |
| [0002](decisions/0002-particionamento-bronze-ingestion-vs-silver-measured.md) | Bronze particionado por `ingestion_date`, Silver por `measured_date` | Projeto 03 |
| [0003](decisions/0003-orquestracao-self-hosted-vs-gerenciada.md) | Airflow self-hosted em vez de serviço gerenciado (MWAA/Composer) | Projeto 02 |
| [0004](decisions/0004-nat-gateway-vs-vpc-endpoints.md) | VPC Endpoints em vez de NAT Gateway | Projeto 04 |
| [0005](decisions/0005-terraform-state-local-vs-remoto.md) | Terraform state local, com limitação documentada | Projeto 04 |
| [0006](decisions/0006-limite-entre-logica-deterministica-e-llm.md) | LLM nunca calcula nem decide — só explica em linguagem natural | Projeto 05 |
| [0007](decisions/0007-great-expectations-vs-framework-proprio.md) | Great Expectations para validação de linha, Python puro para validação de carga | Projeto 06 |

### Prospectivos — decisões justificadas, ainda não implementadas

| ADR | Decisão |
|---|---|
| [0008](decisions/0008-modelagem-star-schema-vs-data-vault-vs-obt.md) | Star Schema seria a escolha se o domínio crescesse para múltiplas fontes |
| [0009](decisions/0009-estrategia-evolucao-de-schema-em-parquet.md) | Estratégia de evolução de schema aditiva + versionamento explícito para mudanças destrutivas |
| [0010](decisions/0010-batch-vs-streaming-proxima-evolucao.md) | Gatilhos concretos que justificariam migrar de batch para streaming |

## Como usar o template

Cada novo ADR começa como uma cópia de
[`template/adr-template.md`](template/adr-template.md), numerado
sequencialmente. As seções são propositalmente rígidas — a
disciplina de sempre listar as opções descartadas (não só a
vencedora) é o que torna um ADR útil daqui a um ano, quando ninguém
mais lembra por que a decisão foi tomada daquele jeito.
