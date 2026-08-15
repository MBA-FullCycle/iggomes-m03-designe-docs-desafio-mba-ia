# ADR-006 — Persistir o payload renderizado (snapshot) na inserção da outbox

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00, no bloco final após a saída de Marcos e Sofia (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Larissa (Tech Lead), Bruno (Eng. Pleno — Time de Pedidos, proponente da questão), Diego (Eng. Sênior — Plataforma)

## 2. Contexto

Definido o padrão outbox ([ADR-001](./ADR-001-outbox-no-mysql.md)), restava uma questão de modelagem levantada no fechamento da reunião: **a linha da outbox guarda o payload já renderizado, ou guarda apenas o `order_id` e o payload é montado na hora do envio?** `[TRANSCRICAO 09:51 Bruno]`

A pergunta tem peso porque o intervalo entre inserção e entrega não é curto: com o backoff de até ~15 horas ([ADR-003](./ADR-003-retry-com-backoff-e-dlq.md)) e com replay manual de DLQ sem prazo definido, o pedido pode mudar várias vezes de estado entre o momento em que o evento nasceu e o momento em que ele é efetivamente entregue.

## 3. Decisão

O payload é **renderizado e persistido na linha da outbox no momento da inserção**, dentro da transação de mudança de status — um snapshot do estado do pedido no instante em que o status mudou `[TRANSCRICAO 09:52 Larissa]`, `[TRANSCRICAO 09:52 Diego]`.

O conteúdo do snapshot é deliberadamente enxuto: `event_id`, `event_type` (ex.: `order.status_changed`), `timestamp` em ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos do pedido como `total_cents`. **Os itens do pedido não vão no payload** — o cliente que precisar de detalhe consulta `GET /orders/:id` `[TRANSCRICAO 09:43 Diego]`.

O identificador da linha da outbox é **UUID**, seguindo o padrão do resto do projeto `[TRANSCRICAO 09:51 Larissa]` — o que é confirmado pelo schema, onde todos os modelos usam `@id @default(uuid()) @db.Char(36)` `[CODIGO prisma/schema.prisma]`.

## 4. Alternativas Consideradas

### Guardar apenas `order_id` e renderizar o payload no momento do envio

- **Descrição:** manter a outbox mínima (referência ao pedido e a transição), montando o JSON com uma consulta ao pedido na hora de disparar a chamada HTTP.
- **Por que foi descartada:** se o pedido mudar entre a inserção e o envio, o evento entregue descreveria um estado que não corresponde ao fato que o originou. Um evento de `PAID` entregue depois de o pedido já ter ido para `SHIPPED` chegaria com dados inconsistentes com a própria transição que ele anuncia — descrito na reunião como "caso esquisito".
- **Origem:** `[TRANSCRICAO 09:51 Bruno]` (proposta), `[TRANSCRICAO 09:52 Larissa]` (descarte)

## 5. Consequências

**Positivas:**

- **O evento é imutável e fiel ao instante do fato.** Um retry disparado 12 horas depois entrega exatamente o mesmo conteúdo da primeira tentativa — propriedade sem a qual o `X-Event-Id` de [ADR-005](./ADR-005-entrega-at-least-once-com-event-id.md) não faria sentido, já que o cliente dedupica assumindo que o mesmo id carrega o mesmo conteúdo.
- **O worker não consulta `orders`, `customers` ou `order_items` no envio**, o que reduz carga de leitura no MySQL e mantém o processamento independente do estado atual do domínio de pedidos.
- **O replay de DLQ reentrega exatamente o que foi gerado**, sem risco de reconstruir um payload diferente do original.

**Negativas:**

- **O cliente pode receber dados desatualizados.** Um evento entregue com 15 horas de atraso descreve o pedido como ele era, não como está. A mitigação é o próprio payload enxuto, que empurra o cliente a buscar o detalhe atual em `GET /orders/:id` quando precisar `[TRANSCRICAO 09:43 Diego]` — mas o cliente que confiar apenas no payload verá informação velha.
- **Cada linha da outbox fica materialmente maior** por carregar o JSON completo, agravando o crescimento da tabela e a dívida de arquivamento já assumida em [ADR-001](./ADR-001-outbox-no-mysql.md). É também o que torna necessário o teto de 64KB por payload, com erro em vez de truncamento `[TRANSCRICAO 09:23 Sofia]`, `[TRANSCRICAO 09:24 Diego]`, `[TRANSCRICAO 09:24 Larissa]`.
- **Mudança de formato de payload não se aplica retroativamente.** Eventos já enfileirados serão entregues no formato antigo, então qualquer evolução de contrato precisa conviver com as duas versões durante o tempo de drenagem da fila.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Fluxos detalhados](../FDD.md#4-fluxos-detalhados) e [FDD — Contratos públicos](../FDD.md#5-contratos-públicos)
- Relacionados: [ADR-001](./ADR-001-outbox-no-mysql.md) (a tabela onde o snapshot vive), [ADR-005](./ADR-005-entrega-at-least-once-com-event-id.md) (imutabilidade como pré-requisito da dedup)
- Substitui: nenhum
