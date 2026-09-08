# ADR 0003: Orquestrar com Airflow self-hosted (Docker Compose) em vez de servico gerenciado

## Status

Aceita

## Contexto

O Projeto 02 precisava de um orquestrador para colocar o pipeline
do Projeto 01 sob agendamento, retries e monitoramento de producao.
As opcoes praticas eram rodar Airflow via Docker Compose localmente
(self-hosted) ou usar um servico gerenciado como Amazon MWAA ou
Google Cloud Composer. A decisao precisava considerar tanto o
objetivo de portfolio (o projeto precisa ser testavel por qualquer
pessoa, sem custo) quanto o que seria a escolha correta em um
contexto real de empresa.

## Opcoes consideradas

### Opcao A - Airflow self-hosted via Docker Compose
- Pros: zero custo de infraestrutura, roda em qualquer maquina com
  Docker, sem dependencia de conta em nuvem para ser avaliado por
  terceiros. Controle total sobre versao e configuracao.
- Contras: em producao real, exigiria alguem responsavel por
  disponibilidade, backup do metastore e atualizacoes de seguranca
  do proprio Airflow - trabalho operacional que um servico gerenciado
  absorve.

### Opcao B - Servico gerenciado (Amazon MWAA ou Cloud Composer)
- Pros: disponibilidade, escalonamento de workers e patches de
  seguranca sao responsabilidade do provedor. E a escolha mais comum
  em empresas que ja operam na nuvem e nao querem manter
  infraestrutura de orquestracao.
- Contras: custo minimo mensal (MWAA cobra mesmo com o ambiente
  ocioso, historicamente na faixa de centenas de dolares/mes para o
  ambiente menor), e exige uma conta de nuvem paga so para o
  avaliador rodar o projeto - inviavel para um portfolio publico.

## Decisao

Opcao A para este projeto, com a ressalva explicita de que essa e
uma decisao de contexto (portfolio, custo zero, avaliacao por
terceiros), nao uma afirmacao de que self-hosted e sempre a escolha
certa. Em uma empresa que ja opera na AWS ou GCP com Data
Engineers dedicados a poucas centenas de DAGs, o calculo muda: o
custo do MWAA/Composer tende a ser menor que o custo de uma pessoa
mantendo o Airflow no ar.

## Consequencias

Este projeto nao demonstra (nem tenta demonstrar) configuracao de um
orquestrador gerenciado - isso e uma lacuna consciente do portfolio,
nao um gap de conhecimento nao percebido. O gatilho para adotar um
servico gerenciado em um contexto real seria: mais de uma pessoa
precisando manter o ambiente, necessidade de alta disponibilidade
formal (SLA), ou o time de dados nao tendo capacidade de operar
infraestrutura como uma responsabilidade a mais.
