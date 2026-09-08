# ADR 0005: Usar Terraform state local no portfolio, documentando o backend remoto como o caminho de producao

## Status

Aceita (com limitacao explicita documentada)

## Contexto

O Projeto 04 provisiona toda a infraestrutura da AWS via Terraform.
O Terraform precisa guardar o "state" (o retrato de quais recursos
existem e com quais configuracoes) em algum lugar. As duas opcoes
padrao da industria sao manter o state em um arquivo local (na
maquina de quem roda o `terraform apply`) ou em um backend remoto
compartilhado, tipicamente um bucket S3 com trava de escrita via
DynamoDB.

## Opcoes consideradas

### Opcao A - State local (arquivo `.tfstate` na maquina)
- Pros: zero configuracao adicional, funciona imediatamente ao
  clonar o repositorio, nao exige nenhum recurso AWS pre-existente
  so para guardar o state.
- Contras: nao serve para times - duas pessoas rodando `apply` a
  partir de maquinas diferentes vao ter states divergentes e podem
  sobrescrever a infraestrutura uma da outra. Sem trava (lock),
  duas execucoes simultaneas podem corromper o state.

### Opcao B - Backend remoto (S3 + DynamoDB para lock)
- Pros: state compartilhado entre qualquer pessoa ou pipeline de
  CI/CD que precise rodar Terraform, com trava contra execucoes
  concorrentes - e o padrao esperado em qualquer ambiente real de
  equipe.
- Contras: exige provisionar o bucket S3 e a tabela DynamoDB *antes*
  de poder rodar o Terraform principal (um problema classico de
  "ovo e galinha" em Terraform, geralmente resolvido com um segundo
  projeto Terraform minimo so para o backend, ou provisionamento
  manual unico).

## Decisao

Opcao A para este portfolio, com a limitacao registrada
explicitamente no README do Projeto 04 desde a primeira versao - nao
como uma omissao descoberta depois, mas como uma escolha consciente
de escopo. O motivo: o projeto e avaliado por uma pessoa por vez,
sem cenario real de multiplas execucoes concorrentes, e adicionar o
backend remoto exigiria documentar e provisionar um segundo conjunto
de recursos (bucket + tabela) so para sustentar a demonstracao
principal.

## Consequencias

Este projeto, como esta, nao e seguro para uso em equipe - isso e
dito explicitamente, nao escondido. O gatilho para migrar para
backend remoto e trivial de identificar: qualquer segunda pessoa
precisando rodar `terraform apply` neste codigo. A migracao em si e
direta (adicionar um bloco `backend "s3"` e rodar `terraform init
-migrate-state`), mas depende de um bucket e tabela existirem
primeiro - o que reforca por que a Opcao B tem um custo de
configuracao inicial que a Opcao A nao tem.
