# ADR-001 — Padrão Outbox no MySQL para publicação de eventos de pedido

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00 (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior — Plataforma), Bruno (Eng. Pleno — Time de Pedidos)

## 2. Contexto

Três clientes B2B pediram notificação em tempo real de mudança de status dos pedidos deles, com "tempo real" definido pelo próprio cliente como qualquer latência abaixo de 10 segundos `[TRANSCRICAO 09:02 Marcos]`. A necessidade de negócio e o SLA estão no [PRD](../PRD.md); a proposta de arquitetura completa está no [RFC](../RFC.md#3-proposta-técnica).

A pergunta específica que este ADR responde é: **onde o evento nasce** — dentro da transação de mudança de status, ou fora dela `[TRANSCRICAO 09:03 Larissa]`.

A restrição vem do código atual. `OrderService.changeStatus` já executa, dentro de um único `prisma.$transaction`, quatro escritas acopladas: validação da transição via `canTransition`, débito ou reposição de estoque em `product.stockQuantity`, `UPDATE` em `orders` e `INSERT` em `order_status_history` `[CODIGO src/modules/orders/order.service.ts::changeStatus]`. É uma transação já pesada, e o sistema não tem hoje nenhum mecanismo de evento, fila ou notificação externa `[CODIGO src/app.ts, package.json]`.

## 3. Decisão

Adotar o **padrão Outbox sobre o MySQL já existente**: a mudança de status insere uma linha na tabela `webhook_outbox` **dentro da mesma transação** que atualiza `orders`, `order_status_history` e o estoque. Um processo separado lê essa tabela e executa as chamadas HTTP (ver [ADR-002](./ADR-002-worker-separado-em-polling.md)).

Se o `INSERT` na outbox falhar, a transação inteira sofre rollback — não pode existir caso de status mudado sem evento registrado `[TRANSCRICAO 09:40 Bruno]`.

## 4. Alternativas Consideradas

### Disparo HTTP síncrono dentro de `changeStatus`

- **Descrição:** chamar o endpoint do cliente diretamente no service de pedidos, no momento da mudança de status.
- **Por que foi descartada:** dois motivos concretos levantados na reunião. Primeiro, a transação já é pesada e um cliente lento passaria a travar a mudança de status **de outros pedidos**, porque a conexão fica presa durante a chamada HTTP. Segundo, se o cliente estiver fora do ar não existe resposta razoável: dar rollback em uma mudança de status legítima porque um terceiro caiu é inaceitável.
- **Origem:** `[TRANSCRICAO 09:04 Bruno]`, `[TRANSCRICAO 09:06 Diego]`

### Redis Streams (ou broker de mensageria externo)

- **Descrição:** publicar o evento em um stream do Redis, com o worker consumindo dali em vez do banco relacional.
- **Por que foi descartada:** exigiria subir e operar infraestrutura nova (Redis Cluster) para um time pequeno, sem ganho proporcional no volume atual — classificado na reunião como overengineering. O MySQL já em produção resolve o problema sem componente novo.
- **Origem:** `[TRANSCRICAO 09:07 Larissa]`, `[TRANSCRICAO 09:07 Diego]`

## 5. Consequências

**Positivas:**

- **Atomicidade real entre fato e evento.** Se a transação commitou, o evento existe; se deu rollback, o evento some junto. Não há janela em que o status mudou e a notificação se perdeu, nem o inverso `[TRANSCRICAO 09:06 Diego]`.
- **Nenhum componente de infraestrutura novo para operar.** Mesmo MySQL, mesmo Prisma, mesma `DATABASE_URL` `[CODIGO src/config/database.ts]`.
- **Desacoplamento de latência:** a duração da chamada HTTP ao cliente deixa de influenciar o tempo da transação de pedido.

**Negativas:**

- **A disponibilidade de `changeStatus` passa a depender da escrita na outbox.** É a contrapartida direta da garantia de atomicidade: um problema de escrita na `webhook_outbox` (lock, disco cheio, constraint) derruba uma operação de negócio que antes não dependia dela `[TRANSCRICAO 09:40 Bruno]`.
- **Crescimento da tabela e dívida operacional assumida.** A outbox acumula linhas entregues; o arquivamento (~30 dias) foi reconhecido como necessário e **explicitamente deixado fora do escopo desta feature** `[TRANSCRICAO 09:08 Diego]`. Sem isso, a tabela cresce indefinidamente.
- **Carga adicional de escrita e leitura no banco principal.** Toda mudança de status vira uma escrita a mais, e o worker faz leituras periódicas contra o mesmo MySQL que serve a API. Mitigado por índices em status e `created_at` e por leitura em batch pequeno `[TRANSCRICAO 09:08 Diego]`, mas não eliminado.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Fluxos detalhados](../FDD.md#4-fluxos-detalhados) e [FDD — Integração com o sistema existente](../FDD.md#12-integração-com-o-sistema-existente)
- Relacionados: [ADR-002](./ADR-002-worker-separado-em-polling.md) (quem consome a outbox), [ADR-006](./ADR-006-snapshot-do-payload-na-outbox.md) (o que a linha da outbox guarda)
- Substitui: nenhum
