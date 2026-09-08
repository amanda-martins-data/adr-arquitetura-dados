# ADR 0010: Permanecer em batch (Airflow diario) ate que gatilhos especificos justifiquem streaming

## Status

Proposta (nao implementada - ADR prospectivo)

## Contexto

Todo o portfolio (Projetos 01 a 06) opera em lotes agendados
(batch): o Airflow do Projeto 02 roda o pipeline em intervalos
fixos, nao continuamente a cada novo dado disponivel. Para dados de
qualidade do ar, que mudam de minuto a minuto na fonte real, batch
diario ou horario introduz uma latencia inerente entre o evento
acontecer e ele aparecer no dashboard. Este ADR registra
prospectivamente quando essa troca deixaria de ser aceitavel e o
projeto justificaria migrar para uma arquitetura orientada a eventos
(ex.: Kafka ou Kinesis).

## Opcoes consideradas

### Opcao A - Permanecer em batch
- Pros: mais simples de operar, testar e depurar - o estado do
  sistema em um dado momento e sempre reproduzivel rodando o mesmo
  intervalo novamente. Menor custo de infraestrutura (sem cluster de
  streaming rodando 24/7).
- Contras: latencia entre evento e disponibilidade do dado igual ao
  intervalo do agendamento, no minimo.

### Opcao B - Migrar para arquitetura orientada a eventos (streaming)
- Pros: latencia de segundos a poucos minutos entre o evento e o
  dado disponivel para consumo - essencial se o caso de uso for
  alerta de emergencia (ex.: pico de poluicao exigindo aviso
  imediato a populacao).
- Contras: introduz complexidade operacional significativa: schema
  registry para compatibilidade de mensagens, estrategia de
  reprocessamento de eventos com problema (dead-letter queue),
  garantias de entrega (at-least-once vs exactly-once), e um cluster
  ou servico gerenciado (Kafka, Kinesis) rodando continuamente.

## Decisao

Permanecer em batch (Opcao A) e nao migrar de forma preventiva. A
decisao registra explicitamente os gatilhos que justificariam a
Opcao B, para que a migracao seja motivada por um requisito real e
nao por preferencia tecnologica: (1) um caso de uso de alerta com
SLA de latencia menor que o menor intervalo de batch pratico (ex.:
"avisar em ate 5 minutos" quando o batch mais frequente viavel e
horario); (2) o volume de eventos justificando o custo fixo de um
cluster de streaming rodando o tempo todo, hoje inexistente para o
volume de dados do OpenAQ; (3) multiplos consumidores downstream
precisando reagir ao mesmo evento em tempo real, em vez de um unico
dashboard atualizado periodicamente.

## Consequencias

O portfolio atual nao demonstra streaming, e este ADR documenta que
essa e uma lacuna reconhecida e nao um ponto cego - a ausencia e
proposital dado que nenhum dos tres gatilhos acima se aplica ao
escopo atual do projeto. Se um projeto futuro do portfolio
implementar o caso de uso de alerta em tempo real, a arquitetura
de referencia seria: ingestao via
Kinesis/Kafka, schema registry para compatibilidade, e um consumidor
que grava tanto em uma camada de streaming (para o alerta imediato)
quanto na Bronze existente (para preservar a trilha de auditoria ja
estabelecida no ADR 0002).
