# ADR 0008: Adotar Star Schema para a camada Gold, com Data Vault e One Big Table descartados para o contexto atual

## Status

Proposta (nao implementada - ADR prospectivo)

## Contexto

A camada Gold do Projeto 03 hoje e uma unica tabela larga
(`air_quality_daily_gold`) com uma linha por cidade, poluente e dia.
Isso funciona bem para o volume e a variedade de consultas atuais
(essencialmente um dashboard de series temporais), mas nao e a unica
forma de modelar uma camada de consumo analitico. Este ADR registra,
de forma prospectiva, qual seria a escolha de modelagem se o projeto
precisasse crescer para suportar multiplas fontes de dados
ambientais (nao so qualidade do ar) e multiplos times de consumo
(dashboards, relatorios ESG, alertas).

## Opcoes consideradas

### Opcao A - Star Schema (fatos e dimensoes)
- Pros: modelo consolidado ha decadas, com suporte nativo na maioria
  das ferramentas de BI (incluindo Power BI, ja usado no portfolio).
  Separa claramente "o que aconteceu" (fatos: medicoes) de "sobre o
  que" (dimensoes: cidade, poluente, tempo) - facilita adicionar
  novas dimensoes (ex.: fonte de emissao, bacia hidrografica) sem
  reescrever os fatos existentes.
- Contras: exige disciplina de modelagem dimensional (chaves
  substitutas, dimensoes lentamente variaveis) que adiciona
  complexidade quando o dominio ainda esta mudando rapido.

### Opcao B - Data Vault (hubs, links, satellites)
- Pros: desenhado para ambientes com muitas fontes de dados
  divergentes e mudanca frequente de schema na origem - separa
  identidade de negocio (hub), relacionamento (link) e atributos
  (satellite) de forma que uma fonte nova raramente exige alterar
  estrutura existente.
- Contras: complexidade de modelagem e de consulta
  desproporcionalmente alta para o numero de fontes que este projeto
  tem hoje (uma). Data Vault se paga em organizacoes com dezenas de
  sistemas de origem heterogeneos, nao em um pipeline com uma unica
  API de dados ambientais.

### Opcao C - One Big Table (o formato atual da Gold)
- Pros: simplicidade maxima de consulta - uma unica tabela, sem
  joins. E o que o projeto usa hoje.
- Contras: nao escala bem para multiplas fontes ou dominios - cada
  nova fonte de dado ambiental (ex.: qualidade da agua, ruido)
  tenderia a virar uma nova tabela larga desconectada das demais, em
  vez de compartilhar dimensoes comuns (cidade, tempo).

## Decisao

Para o escopo atual (uma fonte, um dominio, consumo majoritariamente
via BI), a Opcao C permanece adequada e nao deveria ser trocada
preventivamente. Mas se o portfolio evoluir para multiplas fontes
ambientais compartilhando as mesmas dimensoes de cidade e tempo, a
escolha prospectiva registrada aqui e Star Schema (Opcao A), nao Data
Vault - o criterio de desempate e que Data Vault resolve um problema
de escala de fontes (dezenas) que este dominio nao tem nem
projetando crescimento realista.

## Consequencias

Esta decisao e deliberadamente prospectiva: nenhuma migracao foi
feita. Documentar isso agora serve para que, se e quando o pipeline
receber uma segunda fonte de dados ambientais, a decisao de
modelagem ja tenha sido pensada com calma, em vez de tomada sob
pressao de prazo. O gatilho concreto para migrar da Opcao C para a
Opcao A e a chegada de uma segunda fonte de dados que compartilhe
dimensoes (cidade, tempo) com a atual.
