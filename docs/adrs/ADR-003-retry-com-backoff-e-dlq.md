# ADR-003 — Retry com backoff exponencial (5 tentativas) e DLQ em tabela separada

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00 (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior — Plataforma), Bruno (Eng. Pleno — Time de Pedidos), Marcos (Product Manager)

## 2. Contexto

O worker ([ADR-002](./ADR-002-worker-separado-em-polling.md)) entrega eventos para endpoints HTTP que estão fora da nossa infraestrutura e podem estar indisponíveis por motivos que não controlamos. A pergunta desta decisão é: **o que fazer com um envio que falha, e por quanto tempo insistir** `[TRANSCRICAO 09:14 Larissa]`.

O time tem histórico concreto: já houve cliente com indisponibilidade de duas horas por manutenção planejada `[TRANSCRICAO 09:16 Diego]`. Qualquer política agressiva demais descartaria eventos legítimos nesse cenário.

## 3. Decisão

Falha de envio dispara **retry com backoff exponencial, em até 5 tentativas**, com a progressão **1 minuto → 5 minutos → 30 minutos → 2 horas → 12 horas** — quase 15 horas entre a primeira falha e a última tentativa `[TRANSCRICAO 09:17 Diego]`, `[TRANSCRICAO 09:17 Larissa]`.

Um HTTP call que ultrapasse o **timeout de 10 segundos** é tratado como falha e entra no mesmo fluxo de retry `[TRANSCRICAO 09:42 Diego]`.

Esgotadas as 5 tentativas, o evento é movido para uma **tabela separada `webhook_dead_letter`**, guardando payload, motivo da falha e timestamp `[TRANSCRICAO 09:18 Diego]`. O reprocessamento é **manual**, via `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento como pendente na outbox, **exige role `ADMIN`** e registra em log quem executou o replay, para auditoria `[TRANSCRICAO 09:36 Sofia]`, `[TRANSCRICAO 09:36 Larissa]`.

## 4. Alternativas Consideradas

### 3 tentativas, com política mais agressiva

- **Descrição:** limitar a 3 tentativas em uma janela curta (~30 minutos), matando o evento mais cedo.
- **Por que foi descartada:** três tentativas em meia hora não cobrem uma indisponibilidade de cliente de duas horas — cenário que já aconteceu de fato com um cliente em manutenção planejada. A política mataria eventos que teriam sido entregues com sucesso pouco depois.
- **Origem:** `[TRANSCRICAO 09:16 Bruno]` (proposta), `[TRANSCRICAO 09:16 Diego]` (descarte)

### Retry indefinido com backoff

- **Descrição:** nunca desistir, apenas espaçar cada vez mais as tentativas.
- **Por que foi descartada:** um cliente que simplesmente sumiu (endpoint desligado, empresa migrou de sistema) deixaria eventos pendurados para sempre, sem nunca chegar a um estado terminal — a outbox nunca esvazia e ninguém é obrigado a olhar o problema.
- **Origem:** `[TRANSCRICAO 09:15 Diego]`

### Marcar como `failed` na própria outbox, sem tabela de DLQ

- **Descrição:** manter o evento morto na `webhook_outbox` com um status terminal, evitando uma segunda tabela.
- **Por que foi descartada:** polui a leitura da outbox principal, que passaria a acumular linhas que o worker nunca mais processa. A tabela separada mantém a outbox enxuta e funciona como evidência dedicada para debug e reprocessamento.
- **Origem:** `[TRANSCRICAO 09:17 Larissa]` (pergunta), `[TRANSCRICAO 09:18 Diego]` (descarte)

## 5. Consequências

**Positivas:**

- **A janela de ~15 horas cobre indisponibilidades reais de cliente**, incluindo o caso concreto de duas horas que motivou o descarte das 3 tentativas. Um cliente fora por 15 horas já tem um problema sério do lado dele, e isso foi aceito explicitamente pelo PM `[TRANSCRICAO 09:17 Marcos]`.
- **A outbox permanece enxuta:** o worker nunca varre eventos permanentemente mortos, o que preserva a eficiência do polling.
- **Falha permanente vira um registro consultável**, com payload e motivo, em vez de um evento perdido — e o replay é auditável e restrito a `ADMIN`, reaproveitando `requireRole` `[CODIGO src/middlewares/auth.middleware.ts::requireRole]`.

**Negativas:**

- **Latência de cauda muito alta.** Um evento que falha na primeira tentativa pode chegar até ~15 horas depois. O cliente que voltou do ar recebe um lote atrasado, com conteúdo que reflete o estado de quando o status mudou, não o estado atual ([ADR-006](./ADR-006-snapshot-do-payload-na-outbox.md)).
- **A recuperação depende de alguém perceber.** O replay é manual e **não existe alerta automático** nesta fase — a notificação por e-mail ao cliente com webhook falhando foi explicitamente adiada `[TRANSCRICAO 09:37 Larissa]`. Sem alguém consultando a DLQ, um evento morto fica morto.
- **Mais superfície para construir e manter:** duas tabelas, uma máquina de estados de tentativa, um endpoint administrativo e um requisito de auditoria — em uma feature estimada em três sprints `[TRANSCRICAO 09:46 Larissa]`.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Estratégias de resiliência](../FDD.md#7-estratégias-de-resiliência) e [FDD — Contratos públicos](../FDD.md#5-contratos-públicos)
- Relacionados: [ADR-002](./ADR-002-worker-separado-em-polling.md) (quem executa o retry), [ADR-005](./ADR-005-entrega-at-least-once-com-event-id.md) (retry é a principal origem de entrega duplicada)
- Substitui: nenhum
