# ADR 0007: Usar Great Expectations em vez de um framework de validacao proprio

## Status

Aceita

## Contexto

O Projeto 06 precisava validar a qualidade da camada Gold do
pipeline: valores dentro de faixa, categorias validas, colunas nao
nulas, alem de checagens de frescor e completude que nenhuma
biblioteca padrao resolve pronta. Havia duas rotas possiveis:
escrever funcoes de validacao proprias em Python (como ja acontecia
nos testes do dbt do Projeto 01) ou adotar uma ferramenta dedicada de
qualidade de dados.

## Opcoes consideradas

### Opcao A - Framework de validacao proprio (funcoes Python)
- Pros: controle total, zero dependencia externa, mais facil de
  entender linha por linha para quem le o codigo pela primeira vez.
- Contras: reinventa um problema ja resolvido pela industria -
  regras como "valor entre X e Y" ou "coluna pertence a um conjunto
  finito de categorias" sao extremamente comuns e ja tem
  implementacoes maduras, testadas e com relatorio padronizado.
  Um framework proprio tende a crescer organicamente sem a
  consistencia de uma ferramenta pensada desde o inicio para isso.

### Opcao B - Great Expectations (biblioteca dedicada, real, nao simulada)
- Pros: vocabulario padrao de mercado (o termo "expectation" e
  reconhecido por qualquer pessoa que ja trabalhou com qualidade de
  dados), suporte a modo efemero (contexto em memoria, sem exigir
  projeto completo em disco), relatorio estruturado de quais
  expectativas passaram ou falharam por linha, incluindo contagem de
  valores fora do esperado.
- Contras: biblioteca com dependencias proprias (pandas incluso)
  que precisam ser instaladas e versionadas; curva de aprendizado da
  API da ferramenta antes de escrever a primeira expectation.

## Decisao

Opcao B, mas com uma divisao clara de responsabilidade em vez de
usar Great Expectations para tudo: a ferramenta valida **linhas**
(schema e valores individuais), enquanto checagens de frescor e
completude - que sao sobre a **carga inteira**, nao sobre uma linha -
ficam em Python puro, sem tentar forcar o GX a resolver um tipo de
problema para o qual ele nao foi desenhado (ver tambem a separacao
equivalente entre Bronze e Silver no ADR 0002, mesmo principio
aplicado a validacao). O criterio de desempate para adotar GX na
parte de linha foi nao reinventar uma ferramenta madura para um
problema comum.

## Consequencias

O projeto ganha uma dependencia externa relativamente grande (Great
Expectations traz pandas e outras bibliotecas). Em troca, testes que
rodam a ferramenta de verdade (nao simulada) sao possiveis em modo
efemero, sem exigir infraestrutura adicional - o que foi validado no
Projeto 06 com 6 dos 17 testes automatizados rodando o GX real. O
gatilho para revisitar esta decisao seria a ferramenta se mostrar
pesada demais para o volume de dados do pipeline (ex.: tempo de
inicializacao do contexto se tornando um gargalo em execucoes muito
frequentes), caso em que a Opcao A voltaria a ser considerada para as
checagens mais simples.
