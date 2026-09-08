# ADR 0001: Adotar Data Lake (Parquet) em vez de Data Warehouse tradicional ou Lakehouse gerenciado

## Status

Aceita

## Contexto

O Projeto 03 do portfolio precisava de uma camada de armazenamento
para as tres etapas do pipeline de qualidade do ar (Bronze, Silver,
Gold). Tres familias de solucao eram candidatas: um Data Warehouse
tradicional (tabelas gerenciadas por um motor unico, ex.: Postgres,
Snowflake), um Data Lake puro (arquivos em um formato aberto, sem
motor de banco dono dos dados) ou um Lakehouse gerenciado (Delta
Lake, Apache Iceberg) que promete o melhor dos dois mundos.

Restricoes do contexto: portfolio pessoal, sem orcamento de
infraestrutura paga, precisando rodar 100% local para ser testado
de verdade (nao apenas simulado), e com o objetivo explicito de
demonstrar entendimento de arquitetura em camadas (medallion), nao
so de uma ferramenta especifica.

## Opcoes consideradas

### Opcao A - Data Warehouse tradicional (Postgres como motor unico)
- Pros: modelo relacional familiar, transacoes ACID nativas, uma
  unica ferramenta pra aprender.
- Contras: dado fica preso ao motor - trocar de ferramenta de
  consulta no futuro significa migrar dados, nao so trocar o
  cliente. Sem separacao clara entre camadas de maturidade do dado
  (tudo vira "tabela" independente de ser bruto ou curado). Exige
  um servidor de banco rodando o tempo todo, mesmo para cargas
  pequenas e esporadicas.

### Opcao B - Data Lake puro (Parquet em arquivos, sem motor dono)
- Pros: dado portavel entre DuckDB, Spark, pandas ou Polars sem
  reescrever nada. Custo de armazenamento e computacao desacoplados
  - nao precisa de um servidor ligado pra so guardar arquivo.
  Particionamento por diretorio (Hive-style) e nativo do formato,
  sem exigir configuracao especial de tabela.
- Contras: sem transacoes ACID nativas entre arquivos - escritas
  concorrentes na mesma particao podem corromper dados se nao
  houver disciplina de codigo (mitigado no ADR 0002). Sem controle
  de acesso em nivel de linha/coluna nativo do formato - fica a
  cargo de quem consome.

### Opcao C - Lakehouse gerenciado (Delta Lake ou Apache Iceberg)
- Pros: ACID sobre arquivos, time travel, schema enforcement nativo
  - resolve exatamente as contras da Opcao B.
- Contras: adiciona uma camada de metadados e uma biblioteca extra
  (delta-rs/deltalake ou pyiceberg) que precisa ser mantida e
  entendida. Para o volume de dados deste portfolio (dezenas de MB,
  nao TB), o overhead de metadados do Lakehouse nao se paga - e
  ferramenta pensada pra escala que o projeto nao tem.

## Decisao

Data Lake puro em Parquet (Opcao B). O criterio de desempate foi
proporcionalidade: Lakehouse resolve um problema (escritas
concorrentes, time travel) que o projeto nao tem na escala em que
opera, e adicionar essa complexidade so pra "usar a ferramenta certa
no papel" seria over-engineering. A ausencia de ACID foi endereçada
diretamente com uma regra de design mais simples e barata: cada
camada tem uma politica de escrita propria (ADR 0002), que elimina o
cenario de corrupcao por escrita concorrente sem precisar de um
framework transacional.

## Consequencias

Se o projeto crescer para múltiplos consumidores escrevendo na
mesma particao simultaneamente, ou precisar de time travel para
auditoria, a ausencia de Delta/Iceberg vira uma limitacao real, nao
teorica - esse e o gatilho concreto para revisitar esta decisao.
Ate la, o ganho de portabilidade (o mesmo Parquet lido por DuckDB,
dbt e pandas sem conversao) supera o custo de nao ter transacoes
nativas. Um efeito colateral aceito: qualquer verificacao de
integridade entre camadas (ex.: "todo Silver tem Bronze
correspondente") precisa ser feita por codigo (ver Projeto 06), nao
delegada ao motor de armazenamento.
