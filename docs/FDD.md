# FDD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| **Status** | Rascunho — aguardando revisão técnica antes do início da implementação |
| **Autor(es)** | Larissa (Tech Lead) |
| **Data** | 2026-08-14 |
| **Revisores/Aprovadores** | Bruno (Eng. Pleno — Time de Pedidos), Diego (Eng. Sênior — Plataforma), Sofia (Eng. de Segurança — revisão de segurança bloqueante antes do deploy) |
| **Documentos relacionados** | [RFC](./RFC.md) · [PRD](./PRD.md) · [ADR-001 a ADR-007](./adrs/) · [TRACKER](./TRACKER.md) |

> Convenção de origem usada neste documento: `[RFC seção N]` para a proposta técnica aprovada,
> `[ADR-NNN]` para uma decisão já registrada, `[CODIGO caminho/do/arquivo::símbolo]` para o
> código-base existente (confirmado antes de citar), `[TRANSCRICAO hh:mm Nome]` para falas da
> reunião. Itens marcados com ⚠️ PREMISSA não têm origem direta na fonte e precisam de confirmação.

---

## 1. Contexto e motivação técnica

O [RFC](./RFC.md#3-proposta-técnica) fechou a arquitetura: outbox no MySQL escrita dentro da transação de mudança de status, consumida por um worker em processo separado, com retry/DLQ, assinatura HMAC-SHA256 e entrega at-least-once. Os sete ADRs em [`docs/adrs/`](./adrs/) defendem cada uma dessas escolhas.

Este documento parte dessas decisões como fechadas e desce ao nível que falta para começar a codar: o modelo de dados exato, os passos de cada mecanismo, os payloads dos endpoints, os códigos de erro, os números de resiliência e o ponto exato do código atual onde cada peça se encaixa.

O sistema não tem hoje nenhum mecanismo de evento, fila, agendador ou notificação externa — as dependências de produção são Express, Prisma, `jsonwebtoken`, `bcrypt`, Zod, Pino e `uuid` `[CODIGO package.json]`. Tudo que este FDD descreve é código novo, exceto um único ponto de alteração em código existente: a transação de `OrderService.changeStatus` `[CODIGO src/modules/orders/order.service.ts::changeStatus]`.

## 2. Objetivos técnicos

Quando a implementação estiver pronta, as seguintes afirmações precisam ser verificáveis no código e em teste:

- **OT-01.** Toda transição de status aceita por `canTransition` que tenha ao menos um endpoint de webhook ativo escutando o status de destino gera exatamente uma linha em `webhook_outbox`, na mesma transação `[ADR-001]`, `[CODIGO src/modules/orders/order.status.ts::canTransition]`.
- **OT-02.** Nenhuma chamada HTTP a terceiro acontece dentro de `prisma.$transaction` `[TRANSCRICAO 09:04 Bruno]`.
- **OT-03.** O worker roda como processo independente, iniciável por `npm run worker`, sem depender do processo HTTP estar de pé `[ADR-002]`.
- **OT-04.** A latência entre o commit da mudança de status e a primeira tentativa de entrega é ≤ 2 segundos no caminho feliz `[TRANSCRICAO 09:09 Diego]`.
- **OT-05.** Todo request de saída carrega `X-Event-Id`, `X-Webhook-Id`, `X-Signature` e `X-Timestamp` `[TRANSCRICAO 09:44 Diego]`, `[TRANSCRICAO 09:44 Sofia]`.
- **OT-06.** Todo erro do módulo herda de `AppError` e usa código com prefixo `WEBHOOK_`, sem alteração no middleware de erro existente `[ADR-007]`, `[CODIGO src/middlewares/error.middleware.ts]`.
- **OT-07.** Nenhuma dependência npm nova é adicionada ao `package.json` — HMAC via `node:crypto`, HTTP via `fetch` global do Node 20 `[CODIGO package.json]`.

## 3. Escopo e exclusões

**Nesta fase (implementado por este FDD):**

- Modelo de dados: `webhook_endpoints`, `webhook_outbox`, `webhook_deliveries` e `webhook_dead_letter`.
- Módulo `src/modules/webhooks/` no padrão controller/service/repository/routes/schemas.
- Entrypoint `src/worker.ts` com o loop de polling e a lógica de entrega.
- CRUD de configuração, rotação de secret, histórico de entregas e replay administrativo de DLQ.
- Integração na transação de `changeStatus`.

**Fora desta fase:**

| Exclusão | Motivo | Origem |
| --- | --- | --- |
| Arquivamento/expurgo das linhas entregues na outbox (~30 dias) | Reconhecido como necessário e explicitamente deixado fora do escopo da feature | `[TRANSCRICAO 09:08 Diego]`, `[RFC Q-03]` |
| Notificação por e-mail ao cliente com webhook falhando | Adiado para fase seguinte, após medição de impacto | `[TRANSCRICAO 09:37 Larissa]` |
| Rate limiting de saída por cliente | Observar e implementar se virar problema | `[TRANSCRICAO 09:39 Diego]`, `[RFC Q-01]` |
| Dashboard/painel visual para o cliente | Projeto separado do time de frontend; nesta fase, só endpoints | `[TRANSCRICAO 09:40 Larissa]` |
| Múltiplos workers em paralelo e particionamento por `order_id` | Quebra a garantia de ordenação atual; adiado | `[TRANSCRICAO 09:13 Diego]`, `[RFC Q-02]` |
| Webhooks inbound (cliente enviando para nós) | Escopo é exclusivamente outbound | `[TRANSCRICAO 09:02 Marcos]`, `[TRANSCRICAO 09:03 Sofia]` |

## 4. Fluxos detalhados

### Modelo de dados

Quatro tabelas novas, todas com `id` UUID em `CHAR(36)`, seguindo o padrão de todos os modelos existentes `[TRANSCRICAO 09:51 Larissa]`, `[CODIGO prisma/schema.prisma]`.

| Tabela | Campos principais | Índices | Origem |
| --- | --- | --- | --- |
| `webhook_endpoints` | `id`, `customerId`, `url`, `secret`, `previousSecret`, `previousSecretExpiresAt`, `events` (Json — lista de `OrderStatus`), `active`, `createdAt`, `updatedAt` | `customerId`, `active` | `[TRANSCRICAO 09:21 Bruno]`, `[TRANSCRICAO 09:21 Sofia]`, `[TRANSCRICAO 09:33 Marcos]` |
| `webhook_outbox` | `id`, `webhookEndpointId`, `orderId`, `eventType`, `payload` (Json — snapshot), `status` (`PENDING`/`PROCESSING`/`DELIVERED`/`FAILED`), `attempts`, `nextAttemptAt`, `lastError`, `requestId`, `createdAt`, `updatedAt` | `(status, nextAttemptAt)`, `createdAt`, `orderId` | `[ADR-001]`, `[ADR-006]`, `[TRANSCRICAO 09:08 Diego]` |
| `webhook_deliveries` | `id`, `outboxEventId`, `webhookEndpointId`, `attempt`, `responseStatus`, `responseBody`, `durationMs`, `createdAt` | `webhookEndpointId`, `createdAt` | `[TRANSCRICAO 09:34 Marcos]` |
| `webhook_dead_letter` | `id`, `outboxEventId`, `webhookEndpointId`, `payload` (Json), `failureReason`, `attempts`, `createdAt`, `replayedAt`, `replayedById` | `webhookEndpointId`, `createdAt` | `[ADR-003]`, `[TRANSCRICAO 09:18 Diego]`, `[TRANSCRICAO 09:36 Sofia]` |

O índice composto `(status, nextAttemptAt)` é o que sustenta a consulta de polling — busca por status pendente e vencimento — conforme a exigência de índice em status e `created_at` levantada na reunião `[TRANSCRICAO 09:08 Diego]`.

### FLUXO-01 — Criação do evento na outbox (dentro de `changeStatus`)

1. `OrderService.changeStatus` abre `prisma.$transaction` e executa o que já faz hoje: carrega o pedido, valida `from === to`, valida `canTransition(from, to)`, debita ou repõe estoque, atualiza `orders` e insere em `order_status_history` `[CODIGO src/modules/orders/order.service.ts::changeStatus]`.
2. **Ainda dentro do `tx`**, o service chama `publishWebhookEvent(tx, order, fromStatus, toStatus)` — uma função que recebe o *transaction client* atual, em vez de um repositório injetado `[TRANSCRICAO 09:41 Bruno]`, `[TRANSCRICAO 09:41 Diego]`.
3. A função busca os endpoints do cliente do pedido com `active = true` e cujo array `events` contenha `toStatus`.
4. Se **nenhum** endpoint escuta aquele status, a função retorna sem escrever nada — o filtro é aplicado na inserção, não no envio, para economizar linha na tabela `[TRANSCRICAO 09:34 Bruno]`, `[TRANSCRICAO 09:34 Diego]`.
5. Para cada endpoint elegível, monta o payload renderizado (snapshot) e insere uma linha em `webhook_outbox` com `status = PENDING`, `attempts = 0`, `nextAttemptAt = now()` e o `requestId` da requisição HTTP de origem, para correlação `[ADR-006]`, `[CODIGO src/middlewares/request-logger.middleware.ts]`.
6. A chamada é posicionada **por último** dentro da transação, depois das atualizações de estoque, para não estender o tempo de lock em `products` (ver RSC-04).
7. **Caminho de falha:** qualquer erro na inserção propaga e a transação inteira sofre rollback — o status não muda se o evento não puder ser registrado `[TRANSCRICAO 09:40 Bruno]`, `[TRANSCRICAO 09:41 Diego]`.

**Origem:** `[ADR-001]`, `[ADR-006]`, `[CODIGO src/modules/orders/order.service.ts::changeStatus]`

### FLUXO-02 — Ciclo de polling e entrega pelo worker

1. `src/worker.ts` sobe, instancia o próprio `PrismaClient` via `createPrismaClient()` e registra handlers de `SIGINT`/`SIGTERM` no molde de `src/server.ts` `[CODIGO src/config/database.ts::createPrismaClient]`, `[CODIGO src/server.ts]`.
2. A cada **2 segundos**, o loop consulta `webhook_outbox` por `status = PENDING AND nextAttemptAt <= now()`, ordenado por `createdAt` ascendente, com `take` de batch pequeno `[TRANSCRICAO 09:09 Diego]`, `[TRANSCRICAO 09:08 Diego]`.
3. Para cada evento, o worker faz um **claim condicional** antes de enviar: `UPDATE webhook_outbox SET status = 'PROCESSING', updatedAt = now() WHERE id = ? AND status = 'PENDING'`. Se a atualização afetar 0 linhas, outro processo já pegou o evento e este o ignora (proteção contra worker duplicado — ver RSC-01).
4. Carrega o endpoint, calcula a assinatura (FLUXO-07) e monta os headers.
5. Executa `POST` na `url` do endpoint com `fetch`, sob **timeout de 10 segundos** `[TRANSCRICAO 09:42 Diego]`.
6. Registra uma linha em `webhook_deliveries` com `attempt`, `responseStatus`, `responseBody` e `durationMs`, independentemente do resultado `[TRANSCRICAO 09:34 Marcos]`.
7. **Sucesso** (resposta `2xx`): marca o evento como `DELIVERED`.
8. **Falha** (status não-`2xx`, erro de rede, ou timeout estourado): segue para FLUXO-03.
9. Processado o batch, o loop aguarda o próximo tick.

**Origem:** `[ADR-002]`, `[ADR-003]`, `[RFC seção 3]`

### FLUXO-03 — Retry com backoff exponencial

1. Incrementa `attempts` e grava `lastError` com o código da matriz de erros correspondente.
2. Se `attempts < 5`, calcula `nextAttemptAt = now() + backoff[attempts]`, onde `backoff = [1min, 5min, 30min, 2h, 12h]`, e devolve o evento para `status = PENDING` `[TRANSCRICAO 09:17 Diego]`, `[TRANSCRICAO 09:17 Larissa]`.
3. O evento volta a ser elegível para o polling apenas quando `nextAttemptAt` vencer — é o que impede o worker de reprocessá-lo no tick seguinte.
4. Se `attempts >= 5`, segue para FLUXO-04.

**Origem:** `[ADR-003]`, `[TRANSCRICAO 09:15 Diego]`

### FLUXO-04 — Movimentação para a DLQ

1. Insere uma linha em `webhook_dead_letter` com o `payload` original, `failureReason` (motivo da última falha), `attempts` e o timestamp `[TRANSCRICAO 09:18 Diego]`.
2. Marca o evento na outbox como `FAILED` — a linha permanece na outbox como rastro, mas deixa de ser elegível ao polling.
3. Emite o log `webhook_moved_to_dead_letter` e incrementa a métrica correspondente (seção 8).
4. **Nenhuma notificação automática é disparada** — e-mail ao cliente está fora de escopo nesta fase `[TRANSCRICAO 09:37 Larissa]`. A recuperação depende de alguém consultar a DLQ (ver RSC-05).

**Origem:** `[ADR-003]`

### FLUXO-05 — Replay manual da DLQ

1. Requisição autenticada chega em `POST /api/v1/admin/webhooks/dead-letter/:id/replay`, passando por `authenticate` e por `requireRole('ADMIN')` `[CODIGO src/middlewares/auth.middleware.ts::requireRole]`, `[TRANSCRICAO 09:36 Sofia]`.
2. Carrega a linha da DLQ; se não existir, lança `WEBHOOK_DEAD_LETTER_NOT_FOUND`.
3. Insere uma **nova linha** em `webhook_outbox` com o **mesmo `payload` e o mesmo `id` de evento** do original, `status = PENDING`, `attempts = 0`, `nextAttemptAt = now()`. Preservar o identificador de evento é o que mantém a deduplicação do cliente funcionando no replay `[ADR-005]`.
4. Grava `replayedAt` e `replayedById` (id do usuário do JWT) na linha da DLQ, atendendo à exigência de auditoria de quem executou o replay `[TRANSCRICAO 09:36 Sofia]`.
5. Emite o log `webhook_dead_letter_replayed` com `userId`, `eventId` e `webhookId`.
6. Responde `202 Accepted` — o replay enfileira, não entrega de forma síncrona.

**Origem:** `[ADR-003]`, `[TRANSCRICAO 09:18 Diego]`, `[TRANSCRICAO 09:35 Diego]`

### FLUXO-06 — Rotação de secret com grace period

1. Requisição autenticada chega em `POST /api/v1/webhooks/:id/secret/rotate`.
2. O service gera uma secret nova criptograficamente aleatória com `node:crypto`.
3. Move a secret atual para `previousSecret` e define `previousSecretExpiresAt = now() + 24h` `[TRANSCRICAO 09:21 Sofia]`.
4. Grava a secret nova em `secret` e a devolve **em texto na resposta** — é a única oportunidade do cliente obtê-la.
5. Durante as 24 horas seguintes, os envios continuam sendo assinados apenas com a secret **nova**; a `previousSecret` existe para que o cliente possa aceitar as duas do lado dele enquanto migra os sistemas `[TRANSCRICAO 09:21 Sofia]`.
6. Vencido o prazo, a `previousSecret` é limpa e deixa de ter qualquer efeito.

**Origem:** `[ADR-004]`

### FLUXO-07 — Cálculo da assinatura HMAC

1. Serializa o payload em JSON — a mesma string que vai no corpo do request, byte a byte.
2. Calcula `HMAC-SHA256(corpo, secret_do_endpoint)` com `node:crypto`, em hexadecimal `[TRANSCRICAO 09:20 Sofia]`.
3. Envia o resultado em `X-Signature` e o timestamp do envio em `X-Timestamp`, este último para que o cliente possa detectar replay attack se quiser `[TRANSCRICAO 09:44 Diego]`.
4. Do lado do cliente, a verificação é recalcular o HMAC sobre o corpo cru recebido e comparar com o header. A comparação deve ser feita em tempo constante — orientação a documentar no portal do desenvolvedor.

**Origem:** `[ADR-004]`

## 5. Contratos públicos

Todos os endpoints ficam sob o prefixo `/api/v1` já usado pela API, montado em `src/app.ts` `[CODIGO src/app.ts]` — na reunião os caminhos foram citados sem o prefixo `[TRANSCRICAO 09:34 Marcos]`, `[TRANSCRICAO 09:35 Diego]`. Todas as rotas do módulo exigem `authenticate`; apenas o replay exige `ADMIN` `[TRANSCRICAO 09:37 Sofia]`.

### CONTRATO-01 — `POST /api/v1/webhooks`

Cadastra um endpoint de webhook. A secret é **gerada por nós** e devolvida apenas nesta resposta `[TRANSCRICAO 09:31 Marcos]`. O `customerId` vai no corpo, não é inferido do JWT — o token atual representa o usuário operador do nosso sistema, não o cliente `[TRANSCRICAO 09:32 Bruno]`, `[TRANSCRICAO 09:32 Larissa]`.

**Headers:** `Authorization: Bearer <jwt>`, `Content-Type: application/json`

**Request:**
```json
{
  "customerId": "8f14e45f-ceea-4c3b-9a1d-2b7c5f0e1a33",
  "url": "https://atlas-comercial.example.com/hooks/oms",
  "events": ["SHIPPED", "DELIVERED"]
}
```

**Response `201`:**
```json
{
  "id": "3c9a1b77-51d2-4e88-b0a4-9f2e7c1d4e60",
  "customerId": "8f14e45f-ceea-4c3b-9a1d-2b7c5f0e1a33",
  "url": "https://atlas-comercial.example.com/hooks/oms",
  "events": ["SHIPPED", "DELIVERED"],
  "active": true,
  "secret": "whsec_9f3c1e7d5a2b48c0913e6f4a7d2c8b15",
  "createdAt": "2026-08-14T13:20:41.512Z"
}
```

| Status | Semântica |
| --- | --- |
| `201` | Endpoint criado; `secret` presente no corpo e não recuperável depois |
| `400` | `url` malformada, não-HTTPS, ou `events` com status inválido |
| `401` | Token ausente ou inválido |
| `404` | `customerId` não existe |

**Origem:** `[TRANSCRICAO 09:31 Marcos]`, `[TRANSCRICAO 09:33 Marcos]`, `[ADR-004]`

### CONTRATO-02 — `GET /api/v1/webhooks?customerId=<uuid>`

Lista os endpoints de um cliente. A resposta segue o formato paginado já usado pelo resto da API `[CODIGO src/shared/http/response.ts::paginated]`, com os mesmos limites de `listOrdersQuerySchema` (`pageSize` padrão 20, máximo 100) `[CODIGO src/modules/orders/order.schemas.ts]`. **A secret nunca é devolvida em listagem ou consulta.**

**Response `200`:**
```json
{
  "data": [
    {
      "id": "3c9a1b77-51d2-4e88-b0a4-9f2e7c1d4e60",
      "customerId": "8f14e45f-ceea-4c3b-9a1d-2b7c5f0e1a33",
      "url": "https://atlas-comercial.example.com/hooks/oms",
      "events": ["SHIPPED", "DELIVERED"],
      "active": true,
      "createdAt": "2026-08-14T13:20:41.512Z"
    }
  ],
  "pagination": { "page": 1, "pageSize": 20, "total": 1, "totalPages": 1 }
}
```

| Status | Semântica |
| --- | --- |
| `200` | Lista retornada (possivelmente vazia) |
| `400` | `customerId` ausente ou não-UUID |
| `401` | Token ausente ou inválido |

**Origem:** `[TRANSCRICAO 09:33 Bruno]`, `[CODIGO src/shared/http/response.ts]`

### CONTRATO-03 — `PATCH /api/v1/webhooks/:id`

Edita `url`, `events` e `active`. **Não** rotaciona secret — isso tem endpoint próprio (CONTRATO-05).

**Request:**
```json
{ "events": ["PAID", "SHIPPED", "DELIVERED"], "active": false }
```

**Response `200`:**
```json
{
  "id": "3c9a1b77-51d2-4e88-b0a4-9f2e7c1d4e60",
  "url": "https://atlas-comercial.example.com/hooks/oms",
  "events": ["PAID", "SHIPPED", "DELIVERED"],
  "active": false,
  "updatedAt": "2026-08-14T15:02:10.004Z"
}
```

| Status | Semântica |
| --- | --- |
| `200` | Endpoint atualizado |
| `400` | Campo inválido (URL não-HTTPS, status desconhecido em `events`) |
| `404` | Endpoint não existe |

**Origem:** `[TRANSCRICAO 09:33 Bruno]`

### CONTRATO-04 — `DELETE /api/v1/webhooks/:id`

Remove o cadastro. Eventos já enfileirados para esse endpoint deixam de ser entregues.

**Response `204`:** sem corpo, seguindo o padrão de `OrderController.delete` `[CODIGO src/modules/orders/order.controller.ts]`.

| Status | Semântica |
| --- | --- |
| `204` | Removido |
| `404` | Endpoint não existe |

**Origem:** `[TRANSCRICAO 09:33 Bruno]`

### CONTRATO-05 — `POST /api/v1/webhooks/:id/secret/rotate`

Gera uma secret nova, mantendo a anterior válida por 24 horas (FLUXO-06).

**Response `200`:**
```json
{
  "id": "3c9a1b77-51d2-4e88-b0a4-9f2e7c1d4e60",
  "secret": "whsec_41d8ca9e07b3f562a8c19e40d7b3f218",
  "previousSecretExpiresAt": "2026-08-15T15:10:00.000Z"
}
```

| Status | Semântica |
| --- | --- |
| `200` | Secret rotacionada; nova secret no corpo, não recuperável depois |
| `404` | Endpoint não existe |

**Origem:** `[TRANSCRICAO 09:21 Sofia]`, `[ADR-004]`

### CONTRATO-06 — `GET /api/v1/webhooks/:id/deliveries`

Histórico de entregas do endpoint: sucesso/falha, resposta e tempo de resposta `[TRANSCRICAO 09:34 Marcos]`.

**Response `200`:**
```json
{
  "data": [
    {
      "id": "b21f0c94-77ae-4a3e-8c2f-0d5e91a7b3c4",
      "eventId": "e7c3d9a1-4b62-4f08-9d51-6a2c8e0f7b93",
      "orderId": "1d4f8a02-6c93-4e77-b5a1-8f2c0e6d3b49",
      "attempt": 2,
      "responseStatus": 500,
      "responseBody": "{\"error\":\"internal\"}",
      "durationMs": 412,
      "createdAt": "2026-08-14T13:26:44.180Z"
    }
  ],
  "pagination": { "page": 1, "pageSize": 100, "total": 1, "totalPages": 1 }
}
```

| Status | Semântica |
| --- | --- |
| `200` | Histórico retornado, mais recentes primeiro |
| `404` | Endpoint não existe |

**Origem:** `[TRANSCRICAO 09:34 Marcos]`

### CONTRATO-07 — `POST /api/v1/admin/webhooks/dead-letter/:id/replay`

Recoloca um evento morto na outbox. **Exige role `ADMIN`** e registra quem executou `[TRANSCRICAO 09:35 Diego]`, `[TRANSCRICAO 09:36 Sofia]`.

**Headers:** `Authorization: Bearer <jwt com role ADMIN>`

**Response `202`:**
```json
{
  "deadLetterId": "9a0b7c31-2e58-41d7-b6f9-3c8e5a2d0f14",
  "eventId": "e7c3d9a1-4b62-4f08-9d51-6a2c8e0f7b93",
  "status": "PENDING",
  "replayedAt": "2026-08-14T16:41:09.220Z",
  "replayedById": "5b2e0c88-9f14-4a63-8d07-2e6c1b4a9f35"
}
```

| Status | Semântica |
| --- | --- |
| `202` | Evento reenfileirado; entrega ocorrerá no próximo ciclo do worker |
| `403` | Autenticado, mas sem role `ADMIN` |
| `404` | Item de DLQ não existe |

**Origem:** `[TRANSCRICAO 09:35 Diego]`, `[TRANSCRICAO 09:36 Sofia]`, `[ADR-003]`

### CONTRATO-08 — Requisição de saída para o endpoint do cliente

Este é o contrato que **nós cumprimos** com o cliente. O payload é o snapshot persistido na outbox `[ADR-006]`.

**Headers enviados:**

| Header | Conteúdo | Origem |
| --- | --- | --- |
| `Content-Type` | `application/json` | `[TRANSCRICAO 09:44 Diego]` |
| `X-Event-Id` | UUID do evento, estável entre retries e replay | `[TRANSCRICAO 09:25 Diego]` |
| `X-Webhook-Id` | Id do cadastro de webhook, para clientes com vários endpoints | `[TRANSCRICAO 09:44 Sofia]` |
| `X-Signature` | HMAC-SHA256 do corpo, em hexadecimal | `[TRANSCRICAO 09:20 Sofia]` |
| `X-Timestamp` | Timestamp do envio, para detecção de replay attack pelo cliente | `[TRANSCRICAO 09:44 Diego]` |

**Corpo enviado:**
```json
{
  "event_id": "e7c3d9a1-4b62-4f08-9d51-6a2c8e0f7b93",
  "event_type": "order.status_changed",
  "timestamp": "2026-08-14T13:26:41.900Z",
  "order_id": "1d4f8a02-6c93-4e77-b5a1-8f2c0e6d3b49",
  "order_number": "ORD-000412",
  "from_status": "PROCESSING",
  "to_status": "SHIPPED",
  "customer_id": "8f14e45f-ceea-4c3b-9a1d-2b7c5f0e1a33",
  "total_cents": 148900
}
```

Os **itens do pedido não vão no payload**, para mantê-lo enxuto; o cliente que precisar de detalhe consulta `GET /api/v1/orders/:id` `[TRANSCRICAO 09:43 Diego]`, `[CODIGO src/modules/orders/order.routes.ts]`. O formato de `order_number` (`ORD-000412`) segue o gerado por `reserveOrderNumber` `[CODIGO src/modules/orders/order.service.ts::reserveOrderNumber]`, e os valores de `from_status`/`to_status` são os do enum `OrderStatus` `[CODIGO prisma/schema.prisma]`.

**Resposta esperada do cliente:**

| Status | Semântica |
| --- | --- |
| `2xx` | Entrega bem-sucedida; evento marcado como `DELIVERED` |
| Qualquer outro | Tratado como falha; entra em retry (FLUXO-03) |
| Sem resposta em 10s | Tratado como falha por timeout | 

**Origem:** `[TRANSCRICAO 09:43 Diego]`, `[TRANSCRICAO 09:44 Diego]`, `[ADR-005]`, `[ADR-006]`

## 6. Matriz de erros previstos

Todos os erros do módulo herdam de `AppError` e são formatados pelo middleware existente, sem alteração nele `[CODIGO src/shared/errors/app-error.ts]`, `[CODIGO src/middlewares/error.middleware.ts]`. O prefixo `WEBHOOK_` foi fechado na reunião `[TRANSCRICAO 09:28 Bruno]`, `[TRANSCRICAO 09:29 Larissa]`.

| Código | Causa | Status | Ação esperada do cliente | Origem |
| --- | --- | --- | --- | --- |
| `WEBHOOK_NOT_FOUND` | `:id` de webhook inexistente em consulta, edição, remoção, rotação ou histórico | `404` | Verificar o id do cadastro | `[TRANSCRICAO 09:28 Bruno]` |
| `WEBHOOK_INVALID_URL` | `url` malformada ou com esquema diferente de `https` | `400` | Cadastrar URL HTTPS válida | `[TRANSCRICAO 09:23 Sofia]`, `[TRANSCRICAO 09:28 Bruno]` |
| `WEBHOOK_SECRET_REQUIRED` | Operação que depende de secret ativa em endpoint sem secret válida | `400` | Rotacionar a secret antes de prosseguir | `[TRANSCRICAO 09:28 Bruno]` |
| `WEBHOOK_INVALID_EVENT_TYPE` | `events` contém valor fora do enum `OrderStatus` | `400` | Usar apenas status válidos do ciclo de vida do pedido | `[TRANSCRICAO 09:33 Marcos]`, `[CODIGO prisma/schema.prisma]` |
| `WEBHOOK_DEAD_LETTER_NOT_FOUND` | `:id` de item de DLQ inexistente no replay | `404` | Verificar o id na listagem de DLQ | `[TRANSCRICAO 09:35 Diego]` |
| `WEBHOOK_PAYLOAD_TOO_LARGE` | Payload do evento excede 64KB no momento do envio | — (worker) | Nenhuma; o evento erra e vai para retry/DLQ, não é truncado | `[TRANSCRICAO 09:23 Sofia]`, `[TRANSCRICAO 09:24 Diego]` |
| `WEBHOOK_DELIVERY_TIMEOUT` | Endpoint do cliente não respondeu em 10 segundos | — (worker) | Reduzir o tempo de resposta do próprio endpoint | `[TRANSCRICAO 09:42 Diego]` |
| `WEBHOOK_DELIVERY_FAILED` | Resposta não-`2xx` ou erro de rede na entrega | — (worker) | Consultar `GET /webhooks/:id/deliveries` para o corpo da resposta | `[TRANSCRICAO 09:34 Marcos]` |
| `WEBHOOK_INACTIVE` | Replay ou envio direcionado a endpoint com `active = false` | `409` | Reativar o cadastro antes do replay | ⚠️ PREMISSA — derivado do campo `active` definido em `[TRANSCRICAO 09:21 Bruno]`; a reunião não discutiu esse caso |
| `WEBHOOK_ROTATION_IN_PROGRESS` | Nova rotação solicitada enquanto o grace period de 24h da anterior ainda corre | `409` | Aguardar o fim do grace period | ⚠️ PREMISSA — a reunião definiu o grace period `[TRANSCRICAO 09:21 Sofia]` mas não tratou rotações encadeadas |

Os códigos marcados `— (worker)` não têm status HTTP porque não são resposta a uma requisição: são gravados em `webhook_outbox.lastError` e em `webhook_dead_letter.failureReason`, e ficam visíveis ao cliente pelo histórico de entregas.

**Erros genéricos reaproveitados sem duplicação:** `UNAUTHORIZED` (401), `FORBIDDEN` (403), `VALIDATION_ERROR` (400) e `NOT_FOUND` continuam vindo das classes existentes `[CODIGO src/shared/errors/http-errors.ts]` — o módulo não cria variantes `WEBHOOK_` para o que já é coberto.

## 7. Estratégias de resiliência

| ID | Estratégia | Valor concreto | Origem |
| --- | --- | --- | --- |
| RESIL-01 | Timeout por tentativa de entrega | 10 segundos; estouro conta como falha | `[TRANSCRICAO 09:42 Diego]` |
| RESIL-02 | Número máximo de tentativas | 5 | `[TRANSCRICAO 09:15 Diego]`, `[TRANSCRICAO 09:17 Larissa]` |
| RESIL-03 | Progressão de backoff | 1min → 5min → 30min → 2h → 12h (janela total ≈ 15h) | `[TRANSCRICAO 09:17 Diego]` |
| RESIL-04 | Destino após esgotar tentativas | Tabela `webhook_dead_letter`, com replay manual por ADMIN | `[TRANSCRICAO 09:18 Diego]`, `[ADR-003]` |
| RESIL-05 | Intervalo de polling | 2 segundos | `[TRANSCRICAO 09:09 Diego]` |
| RESIL-06 | Garantia de entrega | At-least-once; dedup pelo cliente via `X-Event-Id` estável entre retries e replay | `[ADR-005]` |
| RESIL-07 | Atomicidade da publicação | `INSERT` na outbox dentro da transação de `changeStatus`; falha causa rollback | `[ADR-001]` |
| RESIL-08 | Teto de payload | 64KB; excedente **erra**, não trunca | `[TRANSCRICAO 09:24 Diego]`, `[TRANSCRICAO 09:24 Larissa]` |
| RESIL-09 | Proteção contra processamento concorrente | Claim condicional `PENDING → PROCESSING` via `UPDATE ... WHERE status = 'PENDING'` antes do envio | ⚠️ PREMISSA — mecanismo não discutido na reunião; necessário para sustentar a premissa de single-worker sob restart ou deploy sobreposto |
| RESIL-10 | Recuperação de eventos presos em `PROCESSING` | Evento em `PROCESSING` há mais de 60 segundos volta a `PENDING` no início do ciclo | ⚠️ PREMISSA — decorre de RESIL-09; sem isso um worker morto no meio do envio deixa o evento preso para sempre |

**Não há circuit breaker nem fallback alternativo de entrega.** Um endpoint consistentemente indisponível simplesmente consome as 5 tentativas e vai para a DLQ — a reunião não discutiu desligamento automático de endpoints problemáticos, e a notificação ao cliente ficou fora de escopo `[TRANSCRICAO 09:37 Larissa]`.

## 8. Observabilidade

**Métricas** (nomes propostos; a reunião não discutiu instrumentação de métricas — ⚠️ PREMISSA quanto aos nomes e ao coletor):

| ID | Métrica | Tipo | Para que serve |
| --- | --- | --- | --- |
| OBS-01 | `webhook_outbox_pending_total` | Gauge | Detectar acúmulo de fila |
| OBS-02 | `webhook_outbox_oldest_pending_age_seconds` | Gauge | Sinal primário de worker parado (RSC-06) |
| OBS-03 | `webhook_delivery_attempts_total{result}` | Counter | Taxa de sucesso/falha por resultado |
| OBS-04 | `webhook_delivery_duration_ms` | Histogram | Distribuição de tempo de resposta dos clientes; base para revisitar o timeout de 10s |
| OBS-05 | `webhook_dead_letter_total` | Counter | Gatilho de alarme para eventos mortos (RSC-05) |
| OBS-06 | `webhook_worker_heartbeat_timestamp` | Gauge | Prova de vida do processo separado |

**Logs.** Reaproveitam o Pino já configurado, sem dependência nova `[CODIGO src/shared/logger/index.ts]`. Os nomes de evento seguem a convenção `snake_case` já usada no projeto (`server_started`, `http_request`, `shutdown_initiated`) `[CODIGO src/server.ts]`, `[CODIGO src/middlewares/request-logger.middleware.ts]`:

| Evento de log | Campos | Momento |
| --- | --- | --- |
| `webhook_event_enqueued` | `eventId`, `webhookId`, `orderId`, `toStatus`, `requestId` | FLUXO-01, após a inserção |
| `webhook_delivery_attempt` | `eventId`, `webhookId`, `attempt`, `statusCode`, `durationMs` | FLUXO-02, a cada envio |
| `webhook_delivery_failed` | `eventId`, `webhookId`, `attempt`, `errorCode`, `nextAttemptAt` | FLUXO-03 |
| `webhook_moved_to_dead_letter` | `eventId`, `webhookId`, `attempts`, `failureReason` | FLUXO-04 |
| `webhook_dead_letter_replayed` | `deadLetterId`, `eventId`, `userId` | FLUXO-05, para auditoria `[TRANSCRICAO 09:36 Sofia]` |

O worker instancia o logger com `base: { service: 'order-management-worker' }`, para separar seus logs dos da API, que hoje usa `service: 'order-management-api'` `[CODIGO src/shared/logger/index.ts]`.

**Redaction — mudança obrigatória.** A configuração atual de `redactPaths` cobre `req.headers.authorization`, `req.headers.cookie`, `*.password`, `*.passwordHash`, `*.token` e `*.accessToken`, **mas não cobre `secret`** `[CODIGO src/shared/logger/index.ts]`. Sem estender a lista com `*.secret` e `*.previousSecret`, qualquer log de objeto de endpoint grava a secret do cliente em texto. Ponto obrigatório da revisão de segurança `[TRANSCRICAO 09:46 Sofia]`.

**Tracing.** O projeto **não tem biblioteca de tracing distribuído** hoje — não há OpenTelemetry nem equivalente nas dependências `[CODIGO package.json]`, e a reunião não discutiu o tema. A correlação viável sem dependência nova é propagar o `requestId` que o `requestLogger` já gera e devolve em `X-Request-Id` `[CODIGO src/middlewares/request-logger.middleware.ts]`: ele é gravado na coluna `webhook_outbox.requestId` na inserção (FLUXO-01) e reemitido em todos os logs do worker para aquele evento. Isso liga a requisição HTTP que mudou o status à entrega do webhook, cruzando a fronteira entre os dois processos. Tracing distribuído propriamente dito exigiria dependência e decisão novas — registrado como Q-03.

## 9. Dependências e compatibilidade

| ID | Dependência | Situação | Origem |
| --- | --- | --- | --- |
| DEP-01 | MySQL via Prisma `5.22.0` | Existente; ganha uma migration com 4 tabelas novas, sem alterar tabelas atuais | `[CODIGO package.json, prisma/schema.prisma]` |
| DEP-02 | HMAC-SHA256 | `node:crypto`, módulo nativo — **nenhum pacote novo** | `[TRANSCRICAO 09:20 Sofia]` |
| DEP-03 | Cliente HTTP de saída | `fetch` global, disponível nativamente sob `"node": ">=20"` declarado em `engines` | `[CODIGO package.json]` |
| DEP-04 | Geração de UUID | `uuid@11.0.3` já em `dependencies`, ou `@default(uuid())` do Prisma | `[CODIGO package.json, prisma/schema.prisma]` |
| DEP-05 | Logging | `pino@9.5.0` já em uso, sem mudança de versão | `[CODIGO package.json]` |
| DEP-06 | Validação | `zod@3.23.8` já em uso | `[CODIGO package.json]` |
| DEP-07 | Novo script `npm run worker` e segunda unidade de deploy | A esteira atual contempla um único processo (`npm start`) | `[CODIGO package.json]`, `[ADR-002]` |

**Compatibilidade.** Nenhum endpoint existente muda de contrato: a feature é aditiva. O único comportamento alterado em código existente é a transação de `changeStatus`, que passa a incluir uma escrita a mais — o corpo da resposta de `PATCH /api/v1/orders/:id/status` permanece idêntico `[CODIGO src/modules/orders/order.controller.ts::changeStatus]`.

## 10. Critérios de aceite técnicos

- **CA-01.** Dado um pedido de um cliente com um endpoint ativo escutando `SHIPPED`, quando `changeStatus` move o pedido de `PROCESSING` para `SHIPPED`, então exatamente uma linha `PENDING` é criada em `webhook_outbox` na mesma transação.
- **CA-02.** Dado um cliente sem nenhum endpoint escutando `PAID`, quando um pedido vai para `PAID`, então nenhuma linha é criada em `webhook_outbox` `[TRANSCRICAO 09:34 Bruno]`.
- **CA-03.** Dado que a inserção na outbox falhe, quando `changeStatus` é executado, então o status do pedido **não** é alterado e nenhuma linha é gravada em `order_status_history` — a transação inteira sofre rollback `[TRANSCRICAO 09:40 Bruno]`.
- **CA-04.** Dado um evento pendente, quando o worker executa um ciclo, então a entrega ocorre em no máximo 2 segundos após o commit, no caminho feliz `[TRANSCRICAO 09:09 Diego]`.
- **CA-05.** Dado um endpoint que responde `500`, quando o worker tenta entregar, então `attempts` vai a 1 e `nextAttemptAt` fica 1 minuto à frente; após 5 falhas, o evento está em `webhook_dead_letter` e marcado `FAILED` na outbox.
- **CA-06.** Dado um endpoint que não responde, quando 10 segundos se passam, então a tentativa é abortada e registrada com `WEBHOOK_DELIVERY_TIMEOUT` `[TRANSCRICAO 09:42 Diego]`.
- **CA-07.** Dado um envio qualquer, quando o request chega ao cliente, então `HMAC-SHA256(corpo, secret)` calculado pelo cliente é idêntico ao valor de `X-Signature`.
- **CA-08.** Dado um evento reentregue por retry ou por replay de DLQ, quando o cliente inspeciona `X-Event-Id`, então o valor é idêntico ao da primeira tentativa `[ADR-005]`.
- **CA-09.** Dada uma rotação de secret, quando 24 horas se passam, então `previousSecret` deixa de existir; antes disso, ela permanece registrada no endpoint `[TRANSCRICAO 09:21 Sofia]`.
- **CA-10.** Dado um usuário com role `OPERATOR`, quando ele chama o endpoint de replay, então a resposta é `403 FORBIDDEN` `[TRANSCRICAO 09:36 Sofia]`, `[CODIGO src/middlewares/auth.middleware.ts::requireRole]`.
- **CA-11.** Dado um replay executado por um ADMIN, quando a operação conclui, então `replayedById` na DLQ contém o id do usuário do JWT e o log `webhook_dead_letter_replayed` foi emitido `[TRANSCRICAO 09:36 Sofia]`.
- **CA-12.** Dado um cadastro com `url` começando em `http://`, quando o `POST /api/v1/webhooks` é chamado, então a resposta é `400 WEBHOOK_INVALID_URL` `[TRANSCRICAO 09:23 Sofia]`.
- **CA-13.** Dado o módulo implementado, quando `npm run lint` e `npm run test` são executados, então passam sem erro e sem nenhuma dependência nova em `package.json` `[CODIGO package.json]`.

## 11. Riscos e mitigação

| ID | Risco | Cenário concreto | Mitigação | Origem |
| --- | --- | --- | --- | --- |
| RSC-01 | **Duas instâncias do worker processando o mesmo evento** | Deploy sobreposto ou restart mal coordenado sobe dois processos; ambos leem o mesmo batch e entregam o evento em duplicidade, quebrando também a ordenação por `order_id` | Claim condicional `PENDING → PROCESSING` antes do envio (RESIL-09); a duplicata residual é absorvida pelo contrato at-least-once | `[ADR-002]`, `[ADR-005]` |
| RSC-02 | **Evento preso em `PROCESSING`** | O worker morre entre o claim e a marcação do resultado; o evento fica em `PROCESSING` e nunca mais é elegível ao polling — some silenciosamente | Recuperação de eventos em `PROCESSING` há mais de 60s (RESIL-10) e métrica OBS-02 sobre a idade do pendente mais antigo | ⚠️ Derivado de RESIL-09 |
| RSC-03 | **Rajada ao worker voltar de uma parada longa** | Worker fora por 3 horas acumula milhares de eventos com `nextAttemptAt` vencido; ao subir, dispara todos e sobrecarrega os endpoints dos clientes | Batch pequeno por ciclo limita a vazão natural `[TRANSCRICAO 09:08 Diego]`; é também o cenário que mais aproxima o rate limiting de saída de virar necessário (Q-01 do RFC) | `[TRANSCRICAO 09:38 Diego]` |
| RSC-04 | **Aumento do tempo de lock na transação de pedido** | A escrita na outbox estende a transação que já debita estoque em `products`; sob concorrência alta no mesmo produto, aumenta contenção de lock | Posicionar a inserção da outbox como última operação da transação (FLUXO-01, passo 6) e manter a consulta de endpoints indexada por `customerId` | `[CODIGO src/modules/orders/order.service.ts::debitStock]` |
| RSC-05 | **Evento morto na DLQ sem ninguém perceber** | Endpoint de um cliente fica fora por 20h; os eventos esgotam as 5 tentativas e vão para a DLQ, mas o replay é manual e não há e-mail nesta fase | Alarme sobre OBS-05; a notificação ao cliente permanece fora de escopo e é candidata à próxima fase | `[TRANSCRICAO 09:18 Diego]`, `[TRANSCRICAO 09:37 Larissa]` |
| RSC-06 | **Secret de cliente gravada em log** | Qualquer `logger.info({ webhook })` com o objeto de endpoint completo grava a secret em texto, porque `redactPaths` não cobre `secret` hoje | Estender `redactPaths` com `*.secret` e `*.previousSecret` antes do deploy; item obrigatório da revisão de segurança | `[CODIGO src/shared/logger/index.ts]`, `[TRANSCRICAO 09:46 Sofia]` |

## 12. Integração com o sistema existente

Todos os caminhos abaixo foram confirmados no código-base.

| Arquivo | Ponto de integração | Mudança |
| --- | --- | --- |
| `src/modules/orders/order.service.ts::changeStatus` | Transação que hoje valida a transição, ajusta estoque, atualiza `orders` e insere em `order_status_history` | Passa a chamar `publishWebhookEvent(tx, order, fromStatus, toStatus)` como última operação dentro do mesmo `prisma.$transaction`, recebendo o `TxClient` já existente no escopo — sem injetar um repositório novo no `OrderService` `[TRANSCRICAO 09:41 Bruno]` |
| `src/shared/errors/app-error.ts::AppError` | Classe base com `statusCode`, `errorCode` e `details` | As classes do módulo (`WebhookNotFoundError`, `WebhookInvalidUrlError`, …) herdam dela ou de suas subclasses HTTP, no mesmo molde de `InvalidStatusTransitionError` e `InsufficientStockError` `[CODIGO src/shared/errors/http-errors.ts]` |
| `src/middlewares/error.middleware.ts::errorMiddleware` | Já formata `AppError`, `ZodError` e `PrismaClientKnownRequestError` em `{ error: { code, message, details } }` | **Nenhuma alteração.** A integração se dá por herança de `AppError` — é exatamente o que torna o reuso viável `[TRANSCRICAO 09:29 Bruno]` |
| `src/middlewares/auth.middleware.ts::requireRole` | Middleware que já valida `ADMIN`/`OPERATOR` e lança `ForbiddenError` | Aplicado como `requireRole('ADMIN')` na rota de replay de DLQ; as demais rotas usam apenas `authenticate` `[TRANSCRICAO 09:36 Larissa]`, `[TRANSCRICAO 09:37 Sofia]` |
| `src/middlewares/validate.middleware.ts::validate` | Valida `body`, `query` e `params` com Zod e converte `ZodError` em `ValidationError` | Reutilizado nas rotas do módulo; `webhook.schemas.ts` define os schemas, incluindo a checagem de URL HTTPS `[TRANSCRICAO 09:23 Sofia]` |
| `src/shared/logger/index.ts::logger` | Pino configurado com `redact`, `base` e `timestamp` ISO | Reutilizado pelo módulo e pelo worker (este com `service: 'order-management-worker'`); **`redactPaths` precisa ganhar `*.secret` e `*.previousSecret`** (RSC-06) |
| `src/config/database.ts::createPrismaClient` | Factory que cria o `PrismaClient` e o singleton `prisma` usado pela API | O worker chama a factory para instanciar o **próprio** client, já que `PrismaClient` é por processo `[TRANSCRICAO 09:30 Bruno]` |
| `src/server.ts` | Entrypoint que sobe o Express e registra `SIGINT`/`SIGTERM` com `prisma.$disconnect()` | Serve de molde para `src/worker.ts`, que replica o bootstrap e o shutdown gracioso, trocando o `listen` pelo loop de polling `[TRANSCRICAO 09:11 Larissa]` |
| `src/app.ts::buildControllers` e `src/routes/index.ts::buildApiRouter` | Montagem de dependências e registro dos routers sob `/api/v1` | Ganham o `WebhookController` no tipo `Controllers` e `router.use('/webhooks', buildWebhookRouter(...))`, seguindo o padrão dos módulos atuais `[TRANSCRICAO 09:27 Bruno]` |
| `prisma/schema.prisma` | Schema MySQL com `OrderStatus`, `Order`, `OrderStatusHistory` e ids `@default(uuid()) @db.Char(36)` | Ganha os 4 modelos novos, referenciando `OrderStatus` e `Customer` existentes; nenhum modelo atual é alterado `[TRANSCRICAO 09:51 Larissa]` |
| `src/shared/http/response.ts::paginated` | Helper de resposta paginada usado por todas as listagens | Reutilizado em `GET /webhooks` e `GET /webhooks/:id/deliveries`, mantendo o formato `{ data, pagination }` |
| `src/modules/orders/order.status.ts::canTransition` | Máquina de estados que define as transições válidas | Fonte de verdade dos valores de `from_status`/`to_status` do payload e dos valores aceitos em `events`; nenhum evento é gerado para transição que a máquina rejeita |

## 13. Questões em aberto e premissas

| ID | Questão em aberto | Quem decide / quando | Origem |
| --- | --- | --- | --- |
| Q-01 | **Onde o teto de 64KB é aplicado.** Sofia disse "a gente não envia" `[TRANSCRICAO 09:23 Sofia]`, o que aponta para validação no momento do envio (evento erra e vai para retry/DLQ). A alternativa — validar na inserção — seria incompatível com `[ADR-001]`, porque faria a mudança de status sofrer rollback por causa do tamanho de um payload. Este FDD adota a validação no envio; a leitura precisa ser confirmada. | Diego / Sofia, na revisão deste FDD | `[TRANSCRICAO 09:23 Sofia]`, `[TRANSCRICAO 09:24 Diego]` |
| Q-02 | **Armazenamento das secrets em repouso.** A reunião definiu secret por endpoint e rotação, mas não se a coluna guarda a secret cifrada ou em texto. | Sofia, na revisão de segurança bloqueante | `[TRANSCRICAO 09:46 Sofia]`, `[RFC P-01]` |
| Q-03 | **Tracing distribuído.** O projeto não tem instrumentação de tracing e a reunião não tratou do tema. A correlação por `requestId` cobre o caso, mas não é tracing propriamente dito. | Time de Plataforma | `[CODIGO package.json]` |
| Q-04 | **Retenção do histórico de entregas.** `webhook_deliveries` cresce a cada tentativa, incluindo retries. A reunião definiu o histórico como requisito `[TRANSCRICAO 09:34 Marcos]` mas não sua janela de retenção — mesma lacuna do arquivamento da outbox. | Sem dono definido | `[TRANSCRICAO 09:34 Marcos]`, `[RFC Q-03]` |
| Q-05 | **Supervisão do processo worker.** Quem mantém o processo no ar (supervisor, orquestrador, política de restart) não foi definido. | Time de Plataforma | `[RFC P-02]` |

| ID | Premissa assumida | O que confirmaria | Impacto se errada |
| --- | --- | --- | --- |
| P-01 | ⚠️ **Claim condicional e recuperação de `PROCESSING`** (RESIL-09/RESIL-10). A reunião assumiu single-worker sem discutir como isso é garantido durante restart ou deploy. | Revisão técnica de Diego | Sem o claim, um deploy sobreposto gera entregas duplicadas em massa; sem a recuperação, eventos somem em `PROCESSING` |
| P-02 | ⚠️ **Nomes de métricas e coletor** (OBS-01 a OBS-06). Métricas não foram discutidas na reunião; os nomes seguem convenção Prometheus. | Time de Plataforma | Renomeação de métricas e ajuste de dashboards/alarmes |
| P-03 | ⚠️ **Erros `WEBHOOK_INACTIVE` e `WEBHOOK_ROTATION_IN_PROGRESS`.** Derivados de campos e fluxos definidos na reunião, mas os cenários específicos não foram tratados. | Revisão deste FDD | Códigos podem ser removidos ou substituídos por comportamento silencioso |
| P-04 | ⚠️ **Prefixo `whsec_` no formato da secret.** Convenção de mercado (Stripe) adotada para legibilidade; a reunião definiu apenas que a secret é gerada por nós. | Sofia, na revisão de segurança | Apenas o formato textual da secret muda; nenhum impacto de fluxo |
