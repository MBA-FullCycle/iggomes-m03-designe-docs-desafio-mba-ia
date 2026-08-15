# ADR-007 — Webhooks como módulo convencional, reaproveitando os padrões existentes do projeto

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00 (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Bruno (Eng. Pleno — Time de Pedidos, proponente), Larissa (Tech Lead), Diego (Eng. Sênior — Plataforma), Sofia (Eng. de Segurança)

## 2. Contexto

A feature introduz o primeiro componente do sistema que **não é um handler HTTP** — o worker de entrega ([ADR-002](./ADR-002-worker-separado-em-polling.md)). Isso abre a porta para tratar webhooks como um subsistema à parte, com convenções próprias.

A codebase, porém, tem um padrão uniforme e consolidado: cada domínio é um módulo em `src/modules/` com `controller`, `service`, `repository`, `routes` e `schemas`, montado em `buildControllers` e registrado em `buildApiRouter` `[CODIGO src/modules/orders/, src/app.ts, src/routes/index.ts]`. A decisão é se webhooks segue essa convenção ou diverge dela `[TRANSCRICAO 09:27 Bruno]`.

## 3. Decisão

**Reuso máximo do que já existe.** Webhooks entra como mais um módulo convencional, sem infraestrutura de código nova `[TRANSCRICAO 09:30 Larissa]`:

| O que é reaproveitado | Onde já existe | Como é usado |
| --- | --- | --- |
| Estrutura de módulo (controller/service/repository/routes/schemas) | `src/modules/orders/` | `src/modules/webhooks/` segue a mesma divisão `[TRANSCRICAO 09:27 Bruno]` |
| `AppError` com `statusCode`, `errorCode` e `details` | `src/shared/errors/app-error.ts` | Erros de webhook herdam de `AppError`, no molde de `InvalidStatusTransitionError` e `InsufficientStockError` `[CODIGO src/shared/errors/http-errors.ts]` |
| Convenção de código de erro em `SCREAMING_SNAKE_CASE` | `INSUFFICIENT_STOCK`, `INVALID_STATUS_TRANSITION` | Todo código do módulo usa o prefixo `WEBHOOK_` `[TRANSCRICAO 09:28 Bruno]`, `[TRANSCRICAO 09:29 Larissa]` |
| Middleware de erro centralizado | `src/middlewares/error.middleware.ts` | Já formata `AppError`, `ZodError` e `PrismaClientKnownRequestError` — **não precisa de alteração** `[TRANSCRICAO 09:29 Bruno]` |
| Logger Pino | `src/shared/logger/index.ts` | Nenhuma dependência de log nova `[TRANSCRICAO 09:29 Bruno]` |
| Validação com Zod | `src/middlewares/validate.middleware.ts` + `*.schemas.ts` | Schemas do módulo no mesmo formato; validação de URL HTTPS entra aqui `[TRANSCRICAO 09:23 Sofia]` |
| Autorização por papel | `src/middlewares/auth.middleware.ts::requireRole` | `requireRole('ADMIN')` no replay de DLQ `[TRANSCRICAO 09:36 Larissa]` |
| Entrypoint de processo | `src/server.ts` | `src/worker.ts` é criado no mesmo molde `[TRANSCRICAO 09:11 Larissa]` |
| Factory de conexão | `src/config/database.ts::createPrismaClient` | O worker chama a factory e **instancia o próprio client** `[TRANSCRICAO 09:30 Bruno]` |
| Identificadores UUID | `prisma/schema.prisma` (todos os modelos usam `@default(uuid()) @db.Char(36)`) | Tabelas de webhook seguem o mesmo padrão `[TRANSCRICAO 09:51 Larissa]` |

A lógica de processamento do worker mora dentro do módulo (`src/modules/webhooks/webhook.worker.ts`), e apenas o entrypoint fica na raiz de `src/` `[TRANSCRICAO 09:28 Bruno]`.

## 4. Alternativas Consideradas

### Compartilhar a mesma instância de `PrismaClient` entre API e worker

- **Descrição:** reaproveitar o singleton `prisma` exportado por `src/config/database.ts`, evitando um segundo pool de conexões contra o mesmo MySQL.
- **Por que foi descartada:** `PrismaClient` é por processo — como o worker roda em um processo Node distinto ([ADR-002](./ADR-002-worker-separado-em-polling.md)), o singleton exportado não é compartilhável entre eles. O reuso possível é o da factory `createPrismaClient()`, não o da instância.
- **Origem:** `[TRANSCRICAO 09:29 Diego]` (pergunta), `[TRANSCRICAO 09:30 Bruno]` (descarte)

### Hierarquia de erros própria do módulo de webhooks

- **Descrição:** definir classes de erro independentes de `AppError`, mais adequadas a um contexto que também roda fora do ciclo de request HTTP.
- **Por que foi descartada:** ⚠️ **Não discutida na fonte** — incluída porque o worker realmente não é um handler HTTP, o que torna a pergunta legítima. Ela cai ao ser confrontada com o código: `errorMiddleware` só formata `AppError`, `ZodError` e `PrismaClientKnownRequestError`; qualquer outra classe cai no branch final e vira um `500 INTERNAL_SERVER_ERROR` genérico, perdendo `errorCode` e `details` na resposta ao cliente `[CODIGO src/middlewares/error.middleware.ts]`. Divergir custaria alterar o middleware central, contra o objetivo declarado de não mexer no que já funciona.
- **Origem:** ⚠️ Não discutida na fonte — incluída por ser tecnicamente plausível

## 5. Consequências

**Positivas:**

- **Nenhuma alteração necessária no tratamento de erros.** Como os erros de webhook herdam de `AppError`, o `errorMiddleware` já devolve `{ error: { code, message, details } }` com o `statusCode` correto, sem uma linha de mudança `[CODIGO src/middlewares/error.middleware.ts]`.
- **Curva de entrada baixa:** qualquer pessoa do time que já mexeu em `src/modules/orders/` sabe onde encontrar cada coisa no módulo novo.
- **Consistência de contrato externo:** códigos `WEBHOOK_*` ficam no mesmo formato de `INSUFFICIENT_STOCK` e `INVALID_STATUS_TRANSITION`, então clientes que já tratam erros da API não precisam de um parser especial.

**Negativas:**

- **O padrão reaproveitado é HTTP-first, e metade da feature não é HTTP.** O worker não passa por rotas nem por middlewares: `validate.middleware.ts` e `error.middleware.ts` **não cobrem** o caminho de execução dele. O tratamento de erro, o logging de contexto e a captura de exceções não tratadas no loop de polling precisam ser escritos à mão, sem apoio da infraestrutura existente — é o principal ponto onde o reuso não entrega o que promete.
- **Um segundo pool de conexões contra o mesmo MySQL.** API e worker abrem clients independentes, o que exige rever o limite de conexões do banco antes do deploy.
- **Arquivos centrais estáveis precisam ser tocados:** o tipo `Controllers` e `buildControllers` em `src/app.ts`, e `buildApiRouter` em `src/routes/index.ts`, mudam para registrar o módulo — pequeno, mas é acoplamento que cada módulo novo paga.
- **`package.json` ganha um script novo (`npm run worker`)** e o processo de deploy passa a ter duas unidades, o que a esteira atual não contempla `[CODIGO package.json]`.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Integração com o sistema existente](../FDD.md#12-integração-com-o-sistema-existente)
- Relacionados: [ADR-002](./ADR-002-worker-separado-em-polling.md) (o processo separado que motiva a alternativa do `PrismaClient`), [ADR-004](./ADR-004-hmac-sha256-secret-por-endpoint.md) (validação de URL HTTPS via Zod)
- Substitui: nenhum
