# ADR-002 — Worker em processo separado consumindo a outbox por polling de 2 segundos

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00 (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior — Plataforma), Bruno (Eng. Pleno — Time de Pedidos)

## 2. Contexto

Com a outbox decidida ([ADR-001](./ADR-001-outbox-no-mysql.md)), restava definir **quem lê a tabela e onde esse processo roda** `[TRANSCRICAO 09:08 Larissa]`.

Duas restrições delimitam o espaço de solução:

1. O orçamento de latência é de 10 segundos ponta a ponta — é o que os clientes chamam de "tempo real" `[TRANSCRICAO 09:02 Marcos]`.
2. O projeto hoje tem um único entrypoint, `src/server.ts`, que sobe o Express e registra os handlers de shutdown `[CODIGO src/server.ts]`. Não existe nenhum processo de background.

## 3. Decisão

O consumo da outbox roda em um **processo Node separado da API**, com entrypoint próprio `src/worker.ts` e script `npm run worker`, espelhando a estrutura de `src/server.ts` `[TRANSCRICAO 09:11 Larissa]`. Esse processo faz **polling a cada 2 segundos**, buscando em batch pequeno os eventos pendentes mais antigos, processando e marcando o resultado.

O worker usa o mesmo banco e a mesma `DATABASE_URL`, mas **instancia o próprio `PrismaClient`**, porque o client é por processo `[TRANSCRICAO 09:30 Bruno]` — a factory `createPrismaClient()` já existe e é reaproveitada `[CODIGO src/config/database.ts]`.

A operação é **single-worker** nesta fase. A ordenação de entrega é garantida por `order_id` (implicitamente, pela ordem de `created_at` da outbox), não globalmente `[TRANSCRICAO 09:12 Diego]`.

## 4. Alternativas Consideradas

### Trigger no MySQL para notificar o worker de forma reativa

- **Descrição:** em vez de polling, usar um gatilho no banco que avisasse o processo consumidor assim que uma linha entrasse na outbox, eliminando a latência de fila.
- **Por que foi descartada:** o MySQL não tem listener nativo equivalente ao `LISTEN/NOTIFY` do PostgreSQL. Trigger no MySQL executa SQL, não notifica processo externo — avisar o worker exigiria improvisar algo como escrever em arquivo ou bater em um endpoint HTTP. O polling de 2 segundos atende o requisito de 10 segundos com folga, sem essa gambiarra.
- **Origem:** `[TRANSCRICAO 09:09 Bruno]` (proposta), `[TRANSCRICAO 09:09 Diego]` (descarte)

### Worker rodando dentro do mesmo processo da API

- **Descrição:** iniciar o loop de consumo junto com o Express, dentro de `src/server.ts`, evitando uma unidade de deploy nova.
- **Por que foi descartada:** o ciclo de vida do worker ficaria amarrado ao da API — um restart ou deploy da API derruba a entrega de webhooks junto. Além disso, em ambiente com múltiplas instâncias da API, cada instância viraria um worker concorrente, quebrando a premissa de single-worker sem ninguém decidir isso.
- **Origem:** `[TRANSCRICAO 09:11 Diego]`

## 5. Consequências

**Positivas:**

- **Latência de fila limitada a ~2 segundos no pior caso**, aceita explicitamente pelo time e confortavelmente dentro do orçamento de 10s `[TRANSCRICAO 09:10 Larissa]`, `[TRANSCRICAO 09:10 Marcos]`.
- **Isolamento de falha e de deploy:** reiniciar a API não interrompe a entrega; um worker travado não afeta o atendimento HTTP dos pedidos.
- **Ordem preservada por pedido** enquanto houver um único worker, que é exatamente a garantia que os clientes pediram — eles querem saber o que aconteceu com cada pedido, não uma ordem global `[TRANSCRICAO 09:14 Marcos]`.

**Negativas:**

- **Polling gera carga constante no MySQL mesmo sem eventos.** São ~30 consultas por minuto contra a outbox, 24h por dia, independentemente de haver movimento.
- **O single-worker é ponto único de falha e teto de throughput.** Se o processo cair, nada é entregue até ele voltar — e não há alerta automático para isso nesta fase. Escalar para múltiplos workers **quebra a garantia de ordenação**, exigindo particionamento por `order_id` ou lock pessimista; isso foi conscientemente adiado como "problema do futuro" `[TRANSCRICAO 09:13 Diego]` e registrado como limitação conhecida `[TRANSCRICAO 09:13 Larissa]`.
- **Uma unidade de deploy nova para operar:** processo, supervisão, restart e observabilidade próprios, em um projeto que até aqui tinha um único processo.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Fluxos detalhados](../FDD.md#4-fluxos-detalhados)
- Relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md) (a tabela consumida), [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md) (o que o worker faz quando o envio falha), [ADR-007](./ADR-007-reuso-dos-padroes-existentes.md) (por que o worker instancia o próprio `PrismaClient`)
- Substitui: nenhum
