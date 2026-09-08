# ADR 0002: Particionar Bronze por data de ingestao e Silver por data do evento

## Status

Aceita

## Contexto

No Projeto 03, cada camada do medallion (Bronze, Silver, Gold)
precisava de uma estrategia de particionamento em disco. A escolha
obvia seria usar o mesmo criterio em todas as camadas para
simplicidade. Mas Bronze e Silver respondem a perguntas diferentes:
Bronze existe para auditoria ("o que a fonte nos enviou e quando"),
Silver existe para consumo de negocio ("o que aconteceu e quando").
Essas duas perguntas nem sempre tem a mesma resposta - um dado
medido no dia 5 pode chegar no pipeline (ser ingerido) no dia 7, por
atraso da fonte ou reprocessamento manual.

## Opcoes consideradas

### Opcao A - Particionar as duas camadas por data de ingestao
- Pros: uma unica logica de particionamento em todo o pipeline,
  mais simples de implementar e explicar.
- Contras: quem consome a Silver para responder "qual foi a media de
  poluicao no dia 5" precisa saber que o dado do dia 5 pode estar
  espalhado em varias particoes de ingestao diferentes, se houve
  reprocessamento. A pergunta de negocio mais comum ("o que aconteceu
  em tal dia") fica cara de responder.

### Opcao B - Particionar as duas camadas por data do evento (measured_date)
- Pros: consistencia entre as camadas.
- Contras: Bronze deixa de ser uma trilha de auditoria fiel - se o
  objetivo da Bronze e reconstruir "o que recebemos e quando", perder
  a data de ingestao como particao joga fora exatamente a informacao
  que a Bronze deveria preservar.

### Opcao C - Bronze por data de ingestao, Silver por data do evento (particionamentos diferentes por camada)
- Pros: cada camada particiona pelo que ela realmente representa.
  Bronze vira auditoria correta (imutavel, cronologia de chegada).
  Silver vira uma tabela de negocio diretamente consultavel por
  periodo de interesse, sem o consumidor precisar saber nada sobre
  como o dado foi ingerido.
- Contras: exige que a transformacao Bronze -> Silver faca uma
  reparticao explicita (nao e so uma copia de arquivo) - mais uma
  etapa de logica para implementar e testar.

## Decisao

Opcao C. O criterio de desempate foi que Bronze e Silver tem
publicos e proposito diferentes, e forcar o mesmo particionamento
nas duas otimiza a implementacao (mais simples de escrever) as
custas de quem consome o dado (mais dificil de usar) - uma troca que
nao vale a pena quando o custo extra de implementar a reparticao e
pequeno e pago uma unica vez.

## Consequencias

A transformacao Silver precisa necessariamente reler o historico
completo da Bronze para conseguir agrupar por measured_date
corretamente (ja que um mesmo dia de evento pode ter chegado em
ingestoes diferentes) - isso e o que motivou a Silver ser idempotente
por particao (reescreve o dia inteiro a cada execucao, ver logica de
`silver_transform.py` no Projeto 03) em vez de fazer append
incremental. Se o volume de dados crescer ao ponto de reler toda a
Bronze a cada execucao da Silver ficar caro, o gatilho para revisitar
esta decisao e implementar uma leitura incremental da Bronze (ex.:
so reprocessar particoes de ingestao novas desde a ultima execucao)
em vez de reler tudo.
