# ADR 0009: Estrategia para evolucao de schema em arquivos Parquet sem quebrar consumidores existentes

## Status

Proposta (nao implementada - ADR prospectivo)

## Contexto

O Data Lake em Parquet (ADR 0001) traz um problema que um Data
Warehouse tradicional resolve de forma mais transparente: quando o
schema de uma fonte muda (uma coluna nova aparece, uma e removida,
um tipo muda de inteiro para decimal), arquivos Parquet ja gravados
com o schema antigo continuam existindo ao lado de arquivos novos
com o schema novo. Sem uma estrategia explicita, um leitor que
espera o schema antigo pode quebrar ao encontrar um arquivo novo, ou
vice-versa. Este ADR registra, de forma prospectiva, como o projeto
trataria essa evolucao caso a fonte de dados (hoje OpenAQ) mudasse
seu formato de resposta.

## Opcoes consideradas

### Opcao A - Reprocessar todo o historico a cada mudanca de schema
- Pros: garante que todos os arquivos de uma camada tenham sempre o
  mesmo schema - simplifica quem le.
- Contras: caro e lento a medida que o volume historico cresce;
  exige acesso aos dados brutos originais (Bronze) para
  reprocessamento completo, o que só é possível porque a Bronze é
  imutável e retém tudo (ADR 0001/0002) - uma fonte que não
  preservasse o histórico bruto não teria essa opção disponível.

### Opcao B - Aditiva com leitura tolerante (schema evolution nativo do formato)
- Pros: colunas novas sao adicionadas sem reescrever arquivos
  antigos; leitores configurados para uniao de schemas (schema
  merging, disponivel em engines como Spark e, com limitacoes, no
  proprio DuckDB) preenchem com nulo o que nao existia no arquivo
  antigo. Nao exige reprocessamento retroativo para adicoes de
  coluna.
- Contras: nao resolve remocao ou mudanca de tipo de coluna de forma
  transparente - essas mudancas continuam exigindo tratamento
  explicito (ver Opcao C).

### Opcao C - Versionar o schema explicitamente por particao (ex.: coluna `_schema_version`)
- Pros: torna a mudanca de schema visivel e rastreavel nos proprios
  dados, em vez de implicita na estrutura de arquivos. Permite que o
  codigo de leitura trate versoes diferentes de forma explicita
  (branching por versao) em vez de depender de comportamento
  implicito do engine de leitura.
- Contras: adiciona uma coluna de controle que nao tem valor de
  negocio, e exige que todo pipeline de leitura saiba interpretar
  versoes.

## Decisao

Combinacao de B e C: mudancas aditivas (nova coluna) sao tratadas de
forma tolerante (Opcao B) por serem o caso mais comum e de menor
risco; mudancas destrutivas (remocao, troca de tipo) exigem
versionamento explicito (Opcao C) porque sao exatamente o tipo de
mudanca que schema merging automatico esconde em vez de sinalizar. O
criterio de desempate foi tratar os dois tipos de mudanca de forma
proporcional ao risco que cada um representa: mudanca aditiva e
de baixo risco, mudanca destrutiva precisa ser visivel.

## Consequencias

Esta estrategia nunca foi implementada no pipeline atual porque a
fonte de dados (OpenAQ) nao mudou seu schema desde a criacao do
projeto - e por isso este ADR e prospectivo. O gatilho concreto para
implementar isso de fato e a primeira mudanca real de schema na
fonte, momento em que a ausencia dessa estrategia se tornaria um
incidente de producao em vez de uma decisao antecipada.
