# Architectural Decision Records

Este diretório armazena os ADRs (Architectural Decision Records) do projeto.
Cada decisão arquitetural relevante deve ser registrada aqui em arquivos individuais,
nomeados sequencialmente no formato `ADR-NNN-titulo-em-kebab-case.md`
(por exemplo `ADR-001-outbox-no-mysql.md`), com `NNN` de 3 dígitos, sem reaproveitar
números de ADRs rejeitados, descontinuados ou substituídos.

## Índice — Sistema de Webhooks de Notificação de Pedidos

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](./ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL para publicação de eventos de pedido | Aceita |
| [ADR-002](./ADR-002-worker-separado-em-polling.md) | Worker em processo separado consumindo a outbox por polling de 2s | Aceita |
| [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff exponencial (5 tentativas) e DLQ em tabela separada | Aceita |
| [ADR-004](./ADR-004-hmac-sha256-secret-por-endpoint.md) | HMAC-SHA256 com secret única por endpoint e rotação com grace period | Aceita |
| [ADR-005](./ADR-005-entrega-at-least-once-com-event-id.md) | Entrega at-least-once com deduplicação pelo cliente via `X-Event-Id` | Aceita |
| [ADR-006](./ADR-006-snapshot-do-payload-na-outbox.md) | Persistir o payload renderizado (snapshot) na inserção da outbox | Aceita |
| [ADR-007](./ADR-007-reuso-dos-padroes-existentes.md) | Webhooks como módulo convencional, reaproveitando os padrões do projeto | Aceita |

Todos os sete ADRs derivam da reunião técnica registrada em [`TRANSCRICAO.md`](../../TRANSCRICAO.md)
e do código-base atual. A proposta consolidada está no [RFC](../RFC.md); o detalhamento de
implementação, no [FDD](../FDD.md); a rastreabilidade item a item, no [TRACKER](../TRACKER.md).
