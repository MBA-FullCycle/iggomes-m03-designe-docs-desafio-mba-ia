# O que pertence a cada seção do FDD

Referência de redação. Cada seção traz o propósito, o que precisa estar lá e um par de exemplos
de conteúdo forte e fraco.

> Os exemplos usam a mesma feature fictícia dos outros geradores da família —
> **exportação/agendamento de relatórios de vendas** — continuando exatamente de onde o RFC parou.
> O ❌ do `rfc-generator` para "Proposta técnica" (tabela `report_jobs`, `BullMQ`, endpoint
> `POST /reports/:id/generate`) era detalhe demais para o RFC — mas é exatamente o nível certo
> aqui: aquele detalhe agora vira o ✅ do FDD. Os exemplos servem para mostrar a *forma* de um bom
> item, nunca para serem copiados como conteúdo. Todo conteúdo do seu FDD vem da sua fonte.

## Índice

1. [Contexto e motivação técnica](#1-contexto-e-motivação-técnica)
2. [Objetivos técnicos](#2-objetivos-técnicos)
3. [Escopo e exclusões](#3-escopo-e-exclusões)
4. [Fluxos detalhados](#4-fluxos-detalhados)
5. [Contratos públicos](#5-contratos-públicos)
6. [Matriz de erros previstos](#6-matriz-de-erros-previstos)
7. [Estratégias de resiliência](#7-estratégias-de-resiliência)
8. [Observabilidade](#8-observabilidade)
9. [Dependências e compatibilidade](#9-dependências-e-compatibilidade)
10. [Critérios de aceite técnicos](#10-critérios-de-aceite-técnicos)
11. [Riscos e mitigação](#11-riscos-e-mitigação)
12. [Integração com o sistema existente](#12-integração-com-o-sistema-existente)

---

## 1. Contexto e motivação técnica

**Propósito:** situar o desenvolvedor no ponto exato onde o RFC parou, sem repetir o problema de
negócio (isso é PRD) nem o debate de arquitetura (isso é RFC/ADR).

Uma frase de ponte para o RFC/ADR relevante basta; o resto da seção é sobre o que este documento
especificamente vai detalhar e por quê essa camada de detalhe é necessária agora (por exemplo, o
time de implementação está bloqueado sem saber o formato exato de um payload).

- ✅ "O RFC-0004 definiu que a geração de relatórios passa a ser assíncrona via worker dedicado
  [RFC seção 3], e o ADR-003 fechou o uso de uma tabela de jobs em vez de fila gerenciada externa
  [ADR-003]. Este FDD detalha a implementação dessas decisões: o schema da tabela, o contrato do
  endpoint que cria o job, e o comportamento exato do worker em cada tentativa — pontos que o RFC
  deixou em nível de componente."
- ❌ "Precisamos melhorar a geração de relatórios porque os usuários reclamam da demora." (isso é
  motivação de produto, já resolvida no PRD; repetir aqui não ajuda quem vai codar)

## 2. Objetivos técnicos

**Propósito:** nomear o que precisa ser verdadeiro no sistema, em termos verificáveis no código,
quando a implementação estiver pronta — diferente do objetivo de negócio do PRD (métrica, NPS,
receita).

- ✅ "Garantir que nenhum job de relatório seja perdido entre a criação do registro e a confirmação
  de processamento pelo worker, mesmo em caso de crash do processo no meio de uma tentativa."
- ❌ "Melhorar a confiabilidade do sistema de relatórios." (não diz o que, especificamente, precisa
  ficar verdadeiro nem como alguém verificaria isso)

## 3. Escopo e exclusões

**Propósito:** delimitar, em granularidade de implementação, o que este FDD cobre e o que fica
fora — inclusive partes do RFC que pertencem a uma fase futura ou a outro módulo.

Diferente do "fora de escopo" do PRD (que fala em funcionalidade de negócio), aqui a exclusão é
técnica: um endpoint que não será implementado nesta fase, um tipo de falha que não será tratado
ainda, uma integração que fica para depois.

- ✅ "Esta versão implementa apenas o fluxo de agendamento único (`schedule_type = ONCE`). O
  agendamento recorrente (`RECURRING`), mencionado no RFC como evolução futura [RFC seção 6], não
  é implementado nesta fase — o schema já reserva a coluna, mas o worker não a processa."
- ❌ "Vamos implementar a feature toda." (não delimita nada, não ajuda ninguém a saber onde parar)

## 4. Fluxos detalhados

**Propósito:** descrever, passo a passo, cada mecanismo central da feature — o suficiente para um
desenvolvedor traduzir diretamente em código e em casos de teste.

Esta é a seção mais importante do FDD e a que mais separa um documento acionável de um que só
parece completo. Escreva em passos numerados ou em uma tabela de transições de estado, nunca em
um parágrafo de prosa que descreve o fluxo por cima. Cubra explicitamente o caminho feliz **e** o
caminho de falha (retry, fallback, estado terminal de erro) de cada mecanismo central.

- ✅ "**FLUXO-01 — Criação do job.**
  1. `POST /reports/:id/generate` valida o payload e cria uma linha em `report_jobs` com
     `status = PENDING` na mesma transação que registra o pedido de exportação.
  2. A API responde `202 Accepted` com o `job_id`, sem aguardar o processamento.
  3. Um worker dedicado faz polling em `report_jobs WHERE status = PENDING` a cada 5 segundos.
  4. Ao pegar um job, o worker marca `status = PROCESSING` com um lock otimista via `updated_at`,
     para evitar dois workers processando o mesmo job."
- ❌ "O sistema cria o job, processa em background e atualiza o status." (não dá para escrever
  código nem teste a partir disso — não diz onde, quando, nem o que acontece se falhar)

## 5. Contratos públicos

**Propósito:** especificar cada endpoint HTTP exposto ou consumido pela feature com precisão
suficiente para implementar o handler e o cliente sem adivinhar nada.

Inclua sempre: método e caminho, payload de exemplo de requisição e resposta, headers relevantes
(autenticação, assinatura, idempotência), e uma tabela de status codes com a semântica de cada um.

- ✅ "`POST /reports/:id/generate`

  Headers: `Authorization: Bearer <token>`, `Idempotency-Key: <uuid>`

  Request:
  ```json
  { "format": "csv", "period": { "from": "2025-01-01", "to": "2025-01-31" } }
  ```

  Response `202 Accepted`:
  ```json
  { "job_id": "a1b2c3", "status": "PENDING" }
  ```

  | Status | Semântica |
  | --- | --- |
  | 202 | Job aceito e enfileirado |
  | 400 | Payload inválido (período ausente ou mal formatado) |
  | 409 | Já existe um job `PENDING`/`PROCESSING` para o mesmo `Idempotency-Key` |"
- ❌ "O endpoint recebe os dados do relatório e retorna sucesso ou erro." (sem método, sem caminho,
  sem payload, sem status codes — não implementável)

## 6. Matriz de erros previstos

**Propósito:** listar cada cenário de falha previsto com um código estável, para que cliente e
observabilidade tratem cada caso de forma inequívoca.

Use o prefixo de código já estabelecido pelo projeto ou pela fonte (por exemplo, o padrão que a
hierarquia de erros existente já segue) — não invente um esquema novo de nomenclatura. Cada linha
precisa de código, causa, status HTTP e, quando fizer diferença, a ação esperada do cliente.

- ✅ "| Código | Causa | Status | Ação esperada do cliente |
  | --- | --- | --- | --- |
  | `REPORT_PERIOD_INVALID` | `from` posterior a `to`, ou intervalo maior que 90 dias | 400 | Corrigir o período e reenviar |
  | `REPORT_JOB_DUPLICATE` | `Idempotency-Key` já associada a um job ativo | 409 | Consultar o job existente por `job_id`, não reenviar |
  | `REPORT_JOB_NOT_FOUND` | `job_id` inexistente ou de outro tenant | 404 | Verificar o identificador |"
- ❌ "`REPORT_ERROR` — erro genérico ao processar relatório, status 500." (um único código genérico
  não permite ao cliente distinguir causas nem decidir se deve tentar de novo)

## 7. Estratégias de resiliência

**Propósito:** quantificar como o sistema se comporta sob falha — nada nesta seção deve ficar sem
número.

Cubra o que se aplica ao mecanismo descrito nos fluxos: timeout por tentativa, número máximo de
tentativas, fórmula de backoff, critério de fallback ou de circuit breaker, e garantia de
idempotência (o que impede duplicar o efeito de uma operação reexecutada).

- ✅ "Cada tentativa de processamento tem timeout de 60 segundos. Em caso de falha, o job volta
  para `status = PENDING` com backoff exponencial (`2^tentativa` minutos, máximo de 30 minutos)
  até 5 tentativas; na 6ª falha, o job vai para `status = DEAD_LETTER` e não é mais reprocessado
  automaticamente. O `Idempotency-Key` garante que reenvios do mesmo `POST` não criam um segundo
  job."
- ❌ "O sistema deve ser resiliente a falhas, com retry e timeout adequados." (nenhum número —
  ninguém consegue implementar "adequado")

## 8. Observabilidade

**Propósito:** nomear exatamente o que será medido e logado, para que o comportamento em produção
seja visível sem precisar instrumentar depois, correndo atrás do incidente.

Nomeie métricas (com tipo — contador, histograma), campos de log específicos e, se aplicável,
spans de tracing — nunca "vamos monitorar" sem dizer o quê.

- ✅ "Métrica `report_job_duration_seconds` (histograma, com label `status`). Log estruturado a
  cada transição de estado do job, incluindo `job_id`, `previous_status`, `new_status` e
  `attempt_count`, usando o logger existente [`src/shared/logger/index.ts`]. Span de tracing
  `report.worker.process_job` envolvendo cada tentativa de processamento."
- ❌ "Vamos adicionar logs e métricas para acompanhar o processamento." (não diz qual métrica, qual
  campo, qual span — não é acionável para quem for instrumentar)

## 9. Dependências e compatibilidade

**Propósito:** listar o que a implementação exige do ambiente e o que ela não pode quebrar em
consumidores existentes.

Inclua bibliotecas ou serviços externos novos (com versão, se relevante) e qualquer restrição de
compatibilidade retroativa — um endpoint que muda de comportamento, um formato de resposta que
precisa continuar aceito.

- ✅ "Requer a lib de fila `bullmq@^5` (já usada em outro módulo do projeto, sem nova dependência
  externa). O endpoint `GET /reports/:id` existente precisa continuar aceitando `job_id` no
  formato antigo (`report-<uuid>`) por compatibilidade com clientes já integrados, além do novo
  formato numérico."
- ❌ "Depende de algumas bibliotecas de fila e deve manter compatibilidade." (não diz qual
  biblioteca, qual versão, nem o que especificamente não pode quebrar)

## 10. Critérios de aceite técnicos

**Propósito:** dar a quem vai revisar o código uma lista verificável de "isto funciona" — no nível
de comportamento do sistema, não de valor de negócio (isso é critério de aceite do PRD).

Cada critério precisa ser verificável objetivamente, idealmente no formato dado-quando-então.

- ✅ "**CA-01.** Dado um job com 5 tentativas de processamento falhas consecutivas, o sistema move
  o registro para `status = DEAD_LETTER` e emite o log de transição correspondente, sem agendar
  nova tentativa."
- ❌ "O sistema deve funcionar corretamente em todos os cenários de erro." (não é verificável — não
  dá para responder sim ou não a isso)

## 11. Riscos e mitigação

**Propósito:** nomear o que pode dar errado especificamente nesta implementação — corrida de
concorrência, migração de dados, comportamento sob carga — diferente do risco de arquitetura do
RFC ou do risco de produto do PRD.

- ✅ "**RSC-01 — Corrida entre dois workers no mesmo job.** Se dois processos de worker fizerem
  polling no mesmo instante, ambos podem tentar processar o mesmo `job_id` antes do lock otimista
  ser aplicado. Mitigação: `UPDATE ... WHERE status = 'PENDING' AND id = ?` com verificação de
  `rowCount` antes de prosseguir, garantindo que só um worker vença a corrida."
- ❌ "**RSC-01 — Complexidade técnica.** A implementação tem algumas partes complexas que podem dar
  errado." (genérico, não indica o cenário concreto nem a mitigação)

## 12. Integração com o sistema existente

**Propósito:** a seção que só o FDD tem — ancorar a implementação em pontos reais e confirmados do
código-base, para que a feature nova se encaixe no sistema em vez de duplicar o que já existe.

Exige no mínimo **4 caminhos de arquivo reais**, cada um confirmado no código-base (não citado de
memória), com o símbolo exato afetado (`arquivo::método` ou `arquivo::classe`) e a descrição
concreta da mudança — não basta dizer que "vai se integrar" com um módulo.

- ✅ "| Arquivo | Ponto de integração | Mudança |
  | --- | --- | --- |
  | `src/modules/reports/report.service.ts::create` | Criação do relatório | Após persistir o
  pedido, inserir a linha em `report_jobs` na mesma transação Prisma já usada pelo método |
  | `src/shared/errors/http-errors.ts` | Hierarquia de erros | Novas classes `ReportJobDuplicateError`
  e `ReportJobNotFoundError` estendem `ConflictError`/`NotFoundError` já existentes, reaproveitando
  o padrão `errorCode` de `AppError` |
  | `src/shared/logger/index.ts::logger` | Observabilidade | Worker importa o logger já
  configurado (com `redactPaths` existente) em vez de criar uma instância própria |
  | `src/middlewares/auth.middleware.ts::requireRole` | Autorização do endpoint | `POST
  /reports/:id/generate` usa `requireRole('ADMIN', 'OPERATOR')` do jeito que os outros endpoints
  do módulo já usam |"
- ❌ "Vamos integrar com o sistema de pedidos e reaproveitar os erros existentes." (não nomeia
  arquivo, não nomeia método, não descreve a mudança — não dá pra saber onde mexer nem confirmar
  que o ponto citado existe)
