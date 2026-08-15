# RFC — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| **Status** | Rascunho — aguardando revisão dos participantes da reunião |
| **Autor(es)** | Larissa (Tech Lead), com proposta técnica de Diego (Eng. Sênior — Plataforma) |
| **Data** | 2026-08-14 |
| **Revisores** | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Eng. Pleno — Time de Pedidos), Diego (Eng. Sênior — Plataforma), Sofia (Eng. de Segurança) |
| **Documentos relacionados** | [PRD](./PRD.md) · [FDD](./FDD.md) · [ADR-001 a ADR-007](./adrs/) · [TRACKER](./TRACKER.md) |

> Convenção de origem usada neste documento: `[TRANSCRICAO hh:mm Nome]` para falas da reunião,
> `[CODIGO caminho/do/arquivo]` para o código existente, `[ADR-NNN]` para um ADR do pacote.
> Itens marcados com ⚠️ não têm origem direta na fonte e precisam de confirmação.

---

## 1. Resumo executivo (TL;DR)

Propomos publicar as mudanças de status de pedido para endpoints HTTP dos clientes usando o **padrão Outbox sobre o MySQL já existente**: a transação que muda o status do pedido também grava o evento, e um **processo separado** (`npm run worker`) consome essa tabela por polling de 2 segundos e executa as chamadas HTTP. Disparo síncrono dentro do service de pedidos foi descartado por acoplar a disponibilidade dos nossos pedidos à disponibilidade dos clientes; um broker externo foi descartado por exigir infraestrutura nova para um time pequeno.

Falhas entram em **retry com backoff exponencial** (5 tentativas, 1m/5m/30m/2h/12h) e, esgotadas, vão para uma **DLQ persistida** com replay manual restrito a `ADMIN`. Cada envio é assinado em **HMAC-SHA256** com secret única por endpoint, rotacionável com 24h de grace period. A garantia publicada é **at-least-once**, com deduplicação do lado do cliente pelo header `X-Event-Id`.

O sistema atual não é alterado em comportamento: a única mudança em código existente é a inclusão de uma escrita a mais dentro da transação de `OrderService.changeStatus`.

## 2. Contexto e problema

Três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) fazem hoje polling em `GET /orders` para descobrir mudanças de status, o que torna a integração deles lenta e cara `[TRANSCRICAO 09:00 Marcos]`. O motivo e o valor de negócio dessa mudança estão no [PRD](./PRD.md); o que interessa aqui é a restrição técnica que ela impõe.

O sistema **não tem nenhum mecanismo de notificação externa, evento, fila ou webhook** — as dependências são Express, Prisma, JWT, Zod e Pino, sem broker, cliente de fila ou scheduler `[CODIGO package.json, src/app.ts]`. Tudo precisa ser construído.

A restrição decisiva está em `OrderService.changeStatus`. O método já executa, dentro de um único `prisma.$transaction`, a validação da transição, o débito ou reposição de estoque em `product.stockQuantity`, o `UPDATE` em `orders` e o `INSERT` em `order_status_history` `[CODIGO src/modules/orders/order.service.ts::changeStatus]`. Acrescentar uma chamada HTTP nesse bloco significaria manter a transação aberta durante a resposta de um terceiro `[TRANSCRICAO 09:04 Bruno]`.

O orçamento de latência é generoso: os clientes consideram "tempo real" qualquer coisa **abaixo de 10 segundos** `[TRANSCRICAO 09:02 Marcos]`. Isso abre espaço para uma solução assíncrona simples, sem necessidade de entrega instantânea.

Escopo direcional: os webhooks são **exclusivamente outbound**. Os clientes recebem, não enviam `[TRANSCRICAO 09:02 Marcos]`, `[TRANSCRICAO 09:03 Sofia]`.

## 3. Proposta técnica

**Publicação — padrão Outbox.** A mudança de status insere uma linha em `webhook_outbox` dentro da mesma transação que já atualiza pedido, histórico e estoque. Se a transação commitou, o evento existe; se deu rollback, o evento some junto `[TRANSCRICAO 09:06 Diego]`. A integração no service se dá por uma função que recebe o *transaction client* atual, em vez de injetar um repositório inteiro no `OrderService` `[TRANSCRICAO 09:41 Bruno]`, `[TRANSCRICAO 09:41 Diego]`. O payload é **renderizado e persistido no momento da inserção** — um snapshot do estado no instante do fato, não uma referência a ser resolvida depois `[ADR-006]`.

**Filtragem na origem.** Cada endpoint de webhook declara quais status quer receber. O filtro é aplicado **na inserção**: se nenhum webhook ativo do cliente escuta aquele status, a linha nem chega a ser criada `[TRANSCRICAO 09:34 Bruno]`.

**Consumo — worker separado em polling.** Um processo Node distinto da API, com entrypoint próprio no molde de `src/server.ts` `[CODIGO src/server.ts]`, lê em batch os eventos pendentes mais antigos a cada 2 segundos, envia e marca o resultado. Roda contra o mesmo banco, com instância própria de `PrismaClient`, porque o client é por processo `[TRANSCRICAO 09:30 Bruno]`. A operação é **single-worker** nesta fase, o que dá ordenação por `order_id` — não ordenação global `[TRANSCRICAO 09:12 Diego]`.

**Resiliência.** Timeout de 10 segundos por chamada `[TRANSCRICAO 09:42 Diego]`. Falha entra em backoff exponencial de 5 tentativas (1m/5m/30m/2h/12h, ~15 horas de janela) e, esgotada, é movida para uma tabela `webhook_dead_letter` separada, com payload e motivo. O reprocessamento é manual, por endpoint administrativo que exige role `ADMIN` e registra quem executou `[ADR-003]`.

**Segurança do envio.** O corpo é assinado em HMAC-SHA256 com uma secret **única por endpoint** — não há secret global de plataforma — enviada em `X-Signature`. A secret é rotacionável pela API, com a anterior válida em paralelo por 24 horas `[ADR-004]`. A URL cadastrada deve ser HTTPS, validada no schema Zod `[TRANSCRICAO 09:23 Sofia]`, e o payload tem teto de 64KB, com erro em vez de truncamento `[TRANSCRICAO 09:24 Larissa]`.

**Contrato de entrega.** At-least-once. Cada evento carrega um UUID estável em `X-Event-Id`, gerado na inserção na outbox e preservado em todos os retries; a deduplicação é responsabilidade do cliente `[ADR-005]`.

**Superfície da API.** CRUD de configuração de webhook (criar, listar, editar, remover), consulta ao histórico de entregas por endpoint, rotação de secret e replay administrativo de DLQ. O CRUD exige apenas autenticação; o replay exige `ADMIN` `[TRANSCRICAO 09:36 Sofia]`. Os payloads, status codes e a matriz de erros estão no [FDD](./FDD.md#5-contratos-públicos).

**Forma no código.** Webhooks entra como módulo convencional em `src/modules/`, seguindo a mesma divisão de `src/modules/orders/`, reaproveitando `AppError`, o middleware de erro centralizado, o Pino e a validação Zod já existentes, com códigos de erro sob o prefixo `WEBHOOK_` `[ADR-007]`.

## 4. Alternativas consideradas

### ALT-01 — Disparo HTTP síncrono dentro de `changeStatus`

- **Descrição:** chamar o endpoint do cliente diretamente no service de pedidos, no momento da mudança de status, sem nenhuma camada intermediária.
- **Por que foi descartada:** a transação de mudança de status já é pesada, e prender uma conexão de banco durante a resposta de um terceiro faz um cliente lento travar mudanças de status **de outros pedidos**. Pior: se o cliente estiver fora do ar, não há resposta aceitável — dar rollback em uma mudança de status legítima porque um terceiro caiu é inviável.
- **Origem:** `[TRANSCRICAO 09:04 Bruno]`, `[TRANSCRICAO 09:06 Diego]` · detalhado em [ADR-001](./adrs/ADR-001-outbox-no-mysql.md)

### ALT-02 — Redis Streams ou broker de mensageria externo

- **Descrição:** publicar o evento em um stream/fila fora do banco relacional, com o worker consumindo dali.
- **Por que foi descartada:** obrigaria subir e operar um componente de infraestrutura novo (Redis Cluster) para um time pequeno, com volume que não justifica o custo operacional — classificado na reunião como overengineering. O MySQL já em produção resolve, e o outbox no mesmo banco ainda dá atomicidade entre o fato e o evento, que a fila externa não daria de graça.
- **Origem:** `[TRANSCRICAO 09:07 Larissa]`, `[TRANSCRICAO 09:07 Diego]` · detalhado em [ADR-001](./adrs/ADR-001-outbox-no-mysql.md)

### ALT-03 — Trigger no MySQL para notificar o worker reativamente

- **Descrição:** substituir o polling por um gatilho no banco que avisasse o consumidor assim que uma linha entrasse na outbox, eliminando a latência de fila.
- **Por que foi descartada:** o MySQL não tem listener nativo equivalente ao `LISTEN/NOTIFY` do PostgreSQL. Trigger no MySQL executa SQL, não notifica processo externo — avisar o worker exigiria improvisar escrita em arquivo ou chamada a um endpoint interno. Como o orçamento de latência é de 10 segundos, o polling de 2 segundos atende com folga e não paga esse custo de complexidade.
- **Origem:** `[TRANSCRICAO 09:09 Bruno]`, `[TRANSCRICAO 09:09 Diego]` · detalhado em [ADR-002](./adrs/ADR-002-worker-separado-em-polling.md)

### ALT-04 — Garantia de entrega exactly-once

- **Descrição:** assegurar que cada evento chegue exatamente uma vez, poupando o cliente de implementar deduplicação.
- **Por que foi descartada:** exigiria coordenação com estado compartilhado entre os dois lados, complexidade desproporcional para uma feature de três sprints. At-least-once com identificador de evento é o padrão que Stripe e GitHub adotam e resolve a quase totalidade dos casos, ao custo — assumido — de transferir a deduplicação para o cliente.
- **Origem:** `[TRANSCRICAO 09:25 Diego]` · detalhado em [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-event-id.md)

> Outras alternativas reais discutidas na reunião ficaram registradas nos ADRs da decisão correspondente, para não estourar o orçamento de página deste documento: 3 tentativas de retry e retry indefinido `[ADR-003]`, DLQ como estado na própria outbox `[ADR-003]`, secret global de plataforma `[ADR-004]`, renderização do payload no envio `[ADR-006]` e `PrismaClient` compartilhado entre API e worker `[ADR-007]`.

## 5. Questões em aberto

| ID | Questão | Quem decide / quando | Origem |
| --- | --- | --- | --- |
| Q-01 | **Rate limiting de saída.** Um cliente com 50 pedidos mudando de status em um minuto receberia 50 chamadas em sequência. Ficou decidido observar e implementar apenas se virar problema real — sem critério numérico definido para "virar problema". | Diego / após medição em produção | `[TRANSCRICAO 09:38 Diego]`, `[TRANSCRICAO 09:39 Larissa]` |
| Q-02 | **Escala para múltiplos workers.** O desenho atual só garante ordenação enquanto houver um worker único. Particionamento por `order_id` e lock pessimista foram citados como caminhos possíveis, sem escolha nem gatilho definido. | Diego / "problema do futuro" | `[TRANSCRICAO 09:13 Bruno]`, `[TRANSCRICAO 09:13 Diego]` |
| Q-03 | **Arquivamento das linhas entregues na outbox.** Reconhecido como necessário (~30 dias), explicitamente colocado fora do escopo desta feature e sem dono ou prazo atribuído. | Sem dono definido | `[TRANSCRICAO 09:08 Diego]` |
| Q-04 | **Nível de autorização do CRUD de webhook.** Fica em "qualquer role autenticada por enquanto", com endurecimento admitido como possível no futuro, sem critério nem data. | Sofia / "mais pra frente" | `[TRANSCRICAO 09:36 Marcos]`, `[TRANSCRICAO 09:37 Sofia]` |

**Premissas assumidas na redação deste documento:**

| ID | Premissa assumida | O que confirmaria | Impacto se errada |
| --- | --- | --- | --- |
| P-01 | ⚠️ **Armazenamento das secrets em repouso.** A reunião definiu secret por endpoint e rotação, mas não discutiu se a secret é cifrada no banco ou guardada em texto. Assumimos que essa definição cabe à revisão de segurança. | Revisão de segurança da Sofia, reservada para antes do deploy `[TRANSCRICAO 09:46 Sofia]` | Pode exigir coluna e fluxo de cifragem/decifragem no módulo, afetando o modelo de dados |
| P-02 | ⚠️ **Supervisão do processo worker.** A reunião decidiu que o worker é um processo separado, mas não definiu quem o mantém no ar (supervisor, orquestrador, política de restart). Assumimos que a esteira atual será estendida. | Time de Plataforma / infra | Sem supervisão, a queda do worker vira interrupção silenciosa de entrega (ver RSC-01) |
| P-03 | ⚠️ **Data da reunião.** A transcrição registra "quinta-feira, 09:00" sem data completa; as datas nos documentos deste pacote referem-se à redação, não à reunião. | Convite da call | Nenhum impacto técnico |

## 6. Impacto e riscos

**Impacto.**

- **Código existente:** a única alteração de comportamento em código de produção é dentro de `OrderService.changeStatus`, que ganha uma escrita a mais na mesma transação `[CODIGO src/modules/orders/order.service.ts]`. `src/app.ts` e `src/routes/index.ts` mudam apenas para registrar o módulo novo, seguindo o padrão de qualquer módulo `[CODIGO src/app.ts, src/routes/index.ts]`.
- **Banco de dados:** migration nova com quatro tabelas (configuração de webhook, outbox, histórico de entregas e dead letter). Nenhuma tabela existente muda de forma `[CODIGO prisma/schema.prisma]`. Modelagem detalhada no [FDD](./FDD.md#4-fluxos-detalhados).
- **Deploy e operação:** o projeto passa de uma para **duas unidades de execução** (API e worker), com um script `npm run worker` novo. A esteira atual contempla um único processo `[CODIGO package.json, src/server.ts]`.
- **Clientes atuais:** nenhum breaking change. A feature é puramente aditiva; nenhum endpoint existente muda de contrato.
- **Produto:** a garantia at-least-once precisa ser documentada com destaque no portal do desenvolvedor, senão vira volume de suporte `[TRANSCRICAO 09:26 Marcos]`.
- **Segurança:** a revisão de HMAC e geração de secret é **bloqueante para o deploy** e reserva pelo menos dois dias úteis no fim do cronograma de três sprints `[TRANSCRICAO 09:46 Sofia]`.

**Riscos.**

| ID | Risco | Probabilidade | Impacto | Mitigação | Origem |
| --- | --- | --- | --- | --- | --- |
| RSC-01 | **Queda silenciosa do worker.** Sendo processo único e separado, se ele cair nenhuma notificação sai — e nada no desenho atual avisa que isso aconteceu. | Média | Alto | ⚠️ Heartbeat do worker e alarme sobre a idade do evento pendente mais antigo na outbox (não discutido na reunião; detalhado em [FDD](./FDD.md#8-observabilidade)) | `[TRANSCRICAO 09:11 Diego]`, `[TRANSCRICAO 09:13 Diego]` |
| RSC-02 | **Falha na escrita da outbox derruba a mudança de status.** É a contrapartida deliberada da atomicidade: um problema na `webhook_outbox` passa a impedir uma operação de negócio que hoje não depende dela. | Baixa | Alto | Trade-off aceito conscientemente na reunião; ⚠️ acompanhar taxa de erro de `changeStatus` nos primeiros dias após o deploy | `[TRANSCRICAO 09:40 Bruno]`, `[TRANSCRICAO 09:41 Diego]` |
| RSC-03 | **Eventos morrem na DLQ sem ninguém perceber.** O replay é manual e a notificação ao cliente por e-mail foi adiada para a próxima fase, então não há gatilho automático para alguém olhar a fila. | Alta | Médio | ⚠️ Métrica de contagem da DLQ com alarme; a notificação ao cliente segue fora de escopo desta fase | `[TRANSCRICAO 09:18 Diego]`, `[TRANSCRICAO 09:37 Larissa]` |
| RSC-04 | **Crescimento indefinido da outbox.** O arquivamento de linhas entregues foi reconhecido como necessário e deixado fora do escopo (Q-03); sem ele, a tabela cresce sem limite e degrada o polling. | Alta | Médio | Índices em status e `created_at` com leitura em batch pequeno seguram o curto prazo; o arquivamento precisa de dono antes de o volume crescer | `[TRANSCRICAO 09:08 Diego]` |
| RSC-05 | **Secrets de cliente vazando em log.** As secrets viram ativo sensível no banco, mas a configuração de `redact` do Pino hoje cobre `password`, `passwordHash`, `token` e `accessToken` — **não cobre `secret`**. | Média | Alto | Estender `redactPaths` antes do deploy e incluir o ponto na revisão de segurança da Sofia | `[CODIGO src/shared/logger/index.ts]`, `[TRANSCRICAO 09:22 Diego]` |

## 7. Decisões relacionadas

| ADR | Decisão | Situação |
| --- | --- | --- |
| [ADR-001](./adrs/ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL para publicação de eventos de pedido | Existente |
| [ADR-002](./adrs/ADR-002-worker-separado-em-polling.md) | Worker em processo separado consumindo a outbox por polling de 2s | Existente |
| [ADR-003](./adrs/ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff exponencial (5 tentativas) e DLQ em tabela separada | Existente |
| [ADR-004](./adrs/ADR-004-hmac-sha256-secret-por-endpoint.md) | HMAC-SHA256 com secret única por endpoint e rotação com grace period de 24h | Existente |
| [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-event-id.md) | Entrega at-least-once com deduplicação pelo cliente via `X-Event-Id` | Existente |
| [ADR-006](./adrs/ADR-006-snapshot-do-payload-na-outbox.md) | Persistir o payload renderizado (snapshot) na inserção da outbox | Existente |
| [ADR-007](./adrs/ADR-007-reuso-dos-padroes-existentes.md) | Webhooks como módulo convencional, reaproveitando os padrões do projeto | Existente |

Decisões táticas que **não** viraram ADR por não terem alternativa real em disputa na reunião — teto de 64KB de payload, timeout de 10s, conjunto de headers do envio e formato do payload — estão registradas no [FDD](./FDD.md) e rastreadas no [TRACKER](./TRACKER.md).
