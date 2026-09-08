# ADR 0004: Usar VPC Endpoints em vez de NAT Gateway para a Lambda alcancar S3 e Secrets Manager

## Status

Aceita

## Contexto

No Projeto 04, a funcao Lambda precisa estar dentro de uma VPC para
se conectar de forma privada ao RDS Postgres. Isso tem um efeito
colateral que so foi descoberto durante a implementacao: uma Lambda
dentro de uma VPC perde, por padrao, a rota de saida para a
internet publica - e com ela, o acesso a servicos AWS que ficam fora
da VPC, como S3 e Secrets Manager, que o pipeline tambem precisa
usar. Sem corrigir isso, a Lambda simplesmente trava tentando
alcancar esses servicos.

## Opcoes consideradas

### Opcao A - NAT Gateway
- Pros: solucao generica que resolve acesso a qualquer endereco na
  internet, nao so servicos AWS - e o padrao mais conhecido e
  ensinado para "Lambda em VPC precisa de internet".
- Contras: custo fixo por hora mesmo sem trafego (na epoca da
  implementacao, em torno de US$32/mes so de taxa, mais custo por GB
  processado) - desproporcional para um pipeline batch diario que
  processa poucos megabytes.

### Opcao B - VPC Endpoints (Gateway para S3, Interface para Secrets Manager)
- Pros: o Gateway Endpoint para S3 nao tem custo adicional. O
  Interface Endpoint para Secrets Manager tem custo por hora, mas
  bem menor que um NAT Gateway. Trafego fica inteiramente dentro da
  rede da AWS, sem passar pela internet publica - superficie de
  ataque menor.
- Contras: resolve apenas os servicos AWS especificos para os quais
  um endpoint foi criado - se o pipeline precisasse futuramente
  chamar uma API externa (fora da AWS), VPC Endpoints nao ajudariam
  e o NAT Gateway (ou uma alternativa) voltaria a ser necessario.

## Decisao

Opcao B. O criterio de desempate foi que todas as dependencias
externas da Lambda neste pipeline sao servicos AWS (S3, Secrets
Manager) - nao ha nenhuma chamada a uma API de terceiros que exija
internet publica generica. Nesse cenario especifico, pagar o custo
fixo de um NAT Gateway para resolver um problema que os Endpoints
resolvem de graca (S3) ou mais barato (Secrets Manager) e desperdicio
de orcamento.

## Consequencias

A API publica do OpenAQ (fonte de dados original do pipeline) fica
inalcancavel pela Lambda dentro da VPC, ja que ela nao e um servico
AWS e nao tem endpoint disponivel - por isso o pipeline usa dado
sintetico por padrao (`openaq_use_synthetic=true`) quando rodando
dentro da VPC. O gatilho para revisitar esta decisao e o pipeline
precisar, de fato, alcançar uma API externa em producao - nesse
momento, as opcoes seriam adicionar um NAT Gateway so para essa rota
especifica, ou mover a chamada externa para fora da VPC (ex.: uma
Lambda separada, sem VPC, que so busca o dado publico e entrega para
a Lambda de dentro da VPC via S3 ou fila).
