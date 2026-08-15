# ADR-005 — Garantia de entrega at-least-once com deduplicação pelo cliente via `X-Event-Id`

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00 (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Diego (Eng. Sênior — Plataforma, proponente), Larissa (Tech Lead), Sofia (Eng. de Segurança), Marcos (Product Manager)

## 2. Contexto

A combinação de outbox ([ADR-001](./ADR-001-outbox-no-mysql.md)) com retry ([ADR-003](./ADR-003-retry-com-backoff-e-dlq.md)) torna a entrega duplicada **inevitável em certos cenários**: se o cliente processa a requisição mas a resposta se perde, ou se o worker cai depois de enviar e antes de marcar o evento como entregue, o mesmo evento é reenviado.

A decisão desta ADR é qual **garantia de entrega** publicamos no contrato, e de quem é a responsabilidade de lidar com a duplicata `[TRANSCRICAO 09:24 Diego]`.

## 3. Decisão

O contrato de entrega é **at-least-once**: garantimos que o evento chega pelo menos uma vez, e assumimos publicamente que ele **pode chegar mais de uma vez**.

Cada evento recebe um **UUID gerado no momento da inserção na outbox**, enviado no header `X-Event-Id`. Esse identificador é estável entre a primeira tentativa e todos os retries, incluindo o replay de DLQ — é o que torna a deduplicação possível. A **deduplicação é responsabilidade do cliente**, feita pelo `X-Event-Id` do lado dele `[TRANSCRICAO 09:25 Diego]`.

Essa característica será documentada de forma destacada no portal do desenvolvedor `[TRANSCRICAO 09:26 Marcos]`.

## 4. Alternativas Consideradas

### Garantia exactly-once

- **Descrição:** assegurar que cada evento chegue exatamente uma vez ao cliente, eliminando a necessidade de dedup do lado dele.
- **Por que foi descartada:** exigiria coordenação entre os dois lados (algum protocolo de confirmação com estado compartilhado), aumentando muito a complexidade de uma feature que precisa caber em três sprints. O argumento decisivo foi que at-least-once com `event_id` é o padrão de mercado — Stripe e GitHub operam assim — e resolve 99% dos casos.
- **Origem:** `[TRANSCRICAO 09:25 Diego]`

### Deduplicação do nosso lado, antes do envio

- **Descrição:** manter no nosso banco o registro de quais eventos já foram confirmados como entregues e suprimir reenvios, poupando o cliente de implementar dedup.
- **Por que foi descartada:** ⚠️ **Não discutida na fonte** — incluída porque é a reação intuitiva à objeção levantada por Sofia de que a decisão "joga responsabilidade pro cliente" `[TRANSCRICAO 09:25 Sofia]`. Ela não resolve o problema: a duplicata que importa nasce justamente quando o cliente processou o evento e a resposta se perdeu no caminho — do nosso lado, isso é indistinguível de uma falha real, e o reenvio acontece de qualquer forma. Ela reduziria alguns casos, sem permitir mudar a garantia publicada de at-least-once para exactly-once.
- **Origem:** ⚠️ Não discutida na fonte — incluída por ser tecnicamente plausível

## 5. Consequências

**Positivas:**

- **Implementação simples e sem coordenação distribuída**, compatível com o prazo de três sprints e com o modelo single-worker `[TRANSCRICAO 09:46 Larissa]`.
- **Alinhamento com o padrão de mercado:** clientes que já integram Stripe ou GitHub reconhecem o modelo e frequentemente já têm dedup implementada.
- **O `X-Event-Id` serve também como chave de correlação** para suporte e debug: o mesmo identificador aparece na outbox, no histórico de entregas, na DLQ e nos logs do worker.

**Negativas:**

- **Transfere trabalho e risco para o cliente**, como foi apontado na própria reunião `[TRANSCRICAO 09:25 Sofia]`. Um cliente que não implementar dedup vai processar o mesmo evento duas vezes — em uma integração que dispare ação com efeito colateral (emitir documento, notificar transportadora), isso vira problema real dele, atribuído à nossa plataforma.
- **Cria uma dependência de documentação:** se o portal do desenvolvedor não explicar a garantia com destaque, a consequência prática é volume de ticket de suporte `[TRANSCRICAO 09:26 Marcos]`.
- **O contrato de entrega é deliberadamente fraco nas duas pontas:** somado à ordenação garantida apenas por `order_id` e apenas enquanto houver um único worker ([ADR-002](./ADR-002-worker-separado-em-polling.md)), o cliente não pode assumir nem unicidade nem ordem global. Isso precisa estar explícito no contrato público, não apenas nos nossos documentos internos.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Contratos públicos](../FDD.md#5-contratos-públicos)
- Relacionados: [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md) (origem das duplicatas), [ADR-006](./ADR-006-snapshot-do-payload-na-outbox.md) (o `event_id` nasce junto com o snapshot)
- Substitui: nenhum
