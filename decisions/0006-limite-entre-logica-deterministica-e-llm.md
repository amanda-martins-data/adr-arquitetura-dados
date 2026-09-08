# ADR 0006: Nunca delegar calculo ou decisao de anomalia ao LLM - so a explicacao em linguagem natural

## Status

Aceita

## Contexto

O Projeto 05 integra a API da Claude a dois pontos do pipeline: um
agente que explica anomalias estatisticas e um agente que sugere
documentacao de modelos dbt. Era tecnicamente possivel pedir ao
proprio modelo de linguagem para, dado um conjunto de numeros brutos,
identificar quais sao anomalos - LLMs conseguem fazer aritmetica
simples e ate reconhecer padroes em dados pequenos. A decisao de
arquitetura era onde exatamente traçar a fronteira entre o que o
LLM faz e o que permanece em codigo determinístico.

## Opcoes consideradas

### Opcao A - Deixar o LLM analisar os dados brutos e apontar anomalias
- Pros: menos codigo para escrever - um prompt bem construido pode
  fazer o trabalho de deteccao e explicacao em uma unica chamada.
- Contras: LLMs nao sao deterministicos por padrao (mesma entrada
  pode gerar respostas diferentes) e nao vem com garantia matematica
  de que um calculo de z-score ou media foi feito corretamente -
  "alucinacao numerica" e um risco documentado em varios estudos
  sobre uso de LLMs para tarefas quantitativas. Um pipeline de dados
  que decide o que e anomalia baseado em algo nao-auditavel e
  nao-testavel de forma tradicional e um risco desproporcional ao
  ganho de simplicidade.

### Opcao B - Deteccao 100% em codigo Python; LLM so recebe o resultado e explica em texto
- Pros: a deteccao (calculo de z-score, comparacao com limite) e
  testavel com testes unitarios convencionais, sem qualquer
  dependencia de rede ou custo de API. O LLM recebe apenas numeros ja
  corretos e sua unica tarefa e traduzir isso para linguagem natural
  - uma tarefa em que LLMs sao genuinamente bons e onde erros sao
  facilmente perceptiveis por um leitor humano (ao contrario de um
  erro de calculo escondido em uma explicacao fluente).
- Contras: exige escrever e manter a logica estatistica
  separadamente (mais codigo do que uma unica chamada de API faria).

## Decisao

Opcao B. O criterio de desempate foi risco assimetrico: o custo de
escrever mais algumas dezenas de linhas de Python e pequeno e
conhecido; o custo de um pipeline de qualidade de dados relatar
"nenhuma anomalia" quando havia uma (ou o inverso) por um erro de
calculo do LLM e alto e dificil de detectar, porque a resposta em
linguagem natural tende a soar confiante independente de estar
certa.

## Consequencias

O agente de qualidade do Projeto 05 nunca pode ser a unica camada de
deteccao de um sistema em producao - ele e uma camada de
comunicacao sobre uma deteccao que ja aconteceu. Essa mesma regra foi
aplicada ao agente de documentacao (Projeto 05): a resposta do LLM
em YAML e sempre validada estruturalmente antes de ser usada, nunca
aplicada as cegas. O gatilho para revisitar esta decisao seria a
disponibilidade de garantias formais de correção matemática em
LLMs (ex.: ferramentas de verificação simbólica acopladas ao
modelo) que hoje não fazem parte do fluxo padrão de uso da API.
