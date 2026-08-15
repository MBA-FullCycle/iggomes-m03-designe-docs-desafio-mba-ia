# TRACKER — Matriz de Rastreabilidade

Este documento é o instrumento anti-alucinação do pacote. Cada item registrado no PRD, no RFC, no FDD e nos ADRs aparece aqui com sua origem resolvida até uma das **duas únicas fontes de verdade aceitas**: a reunião registrada em [`TRANSCRICAO.md`](../TRANSCRICAO.md) ou o código real do repositório.

Citações intermediárias (`[RFC seção 3]`, `[ADR-004]`) não são destino — foram resolvidas em cadeia até o timestamp ou o caminho de arquivo de origem.

**Convenção da coluna Localização:**
- `TRANSCRICAO` → `[hh:mm] Nome` do falante
- `CODIGO` → caminho real do arquivo, com `::símbolo` quando o ponto exato importa

**Verificação executada em 2026-08-14:** as 369 citações de transcrição dos documentos foram conferidas contra `TRANSCRICAO.md` (timestamp e falante), e todos os caminhos de código citados com a marcação `[CODIGO ...]` foram confirmados como existentes no repositório. Os itens que **não** resolvem para nenhuma das duas fontes estão isolados na [seção 2](#2-itens-sem-origem-rastreável-premissas-e-lacunas), fora da tabela principal, em vez de receberem uma origem forçada.

---

## 1. Matriz principal

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| -- | --------- | ---- | ----------------- | ----- | ----------- |
| PRD-OBJ-01 | docs/PRD.md | Objetivo | Entregar a notificação em menos de 10 segundos após a mudança de status | TRANSCRICAO | `[09:02] Marcos` |
| PRD-OBJ-02 | docs/PRD.md | Objetivo | Reter os 3 clientes B2B que pediram a feature, integrados até fim de novembro | TRANSCRICAO | `[09:45] Marcos` |
| PRD-OBJ-03 | docs/PRD.md | Objetivo | Eliminar o polling que os clientes fazem hoje em `GET /orders` | TRANSCRICAO | `[09:00] Marcos` |
| PRD-OBJ-04 | docs/PRD.md | Objetivo | Entregar em 3 sprints, incluindo a revisão de segurança | TRANSCRICAO | `[09:46] Larissa` |
| PRD-CTX-01 | docs/PRD.md | Contexto | Três clientes B2B nomeados pediram notificação em tempo real | TRANSCRICAO | `[09:00] Marcos` |
| PRD-CTX-02 | docs/PRD.md | Contexto | Atlas ameaça migrar para o concorrente se não houver entrega no trimestre | TRANSCRICAO | `[09:00] Marcos` |
| PRD-CTX-03 | docs/PRD.md | Contexto | O OMS não possui hoje nenhum mecanismo de notificação externa | CODIGO | `package.json` |
| PRD-CTX-04 | docs/PRD.md | Contexto | O ciclo de vida do pedido já é controlado por máquina de estados | CODIGO | `src/modules/orders/order.status.ts` |
| PRD-FE-01 | docs/PRD.md | Fora de escopo (adiado) | Aviso por e-mail ao cliente quando o webhook dele falha | TRANSCRICAO | `[09:37] Larissa` |
| PRD-FE-02 | docs/PRD.md | Fora de escopo (descartado) | Painel/dashboard visual para o cliente | TRANSCRICAO | `[09:40] Larissa` |
| PRD-FE-03 | docs/PRD.md | Fora de escopo (adiado) | Limitação da taxa de envio por cliente | TRANSCRICAO | `[09:39] Larissa` |
| PRD-FE-04 | docs/PRD.md | Fora de escopo (descartado) | Webhooks de entrada (cliente enviando para nós) | TRANSCRICAO | `[09:02] Marcos` |
| PRD-FE-05 | docs/PRD.md | Fora de escopo (descartado) | Garantia de que um aviso nunca chega duplicado (exactly-once) | TRANSCRICAO | `[09:25] Diego` |
| PRD-FE-06 | docs/PRD.md | Fora de escopo (descartado) | Garantia de ordem global entre pedidos diferentes | TRANSCRICAO | `[09:13] Larissa` |
| PRD-FE-07 | docs/PRD.md | Fora de escopo (adiado) | Expurgo/arquivamento dos eventos já entregues | TRANSCRICAO | `[09:08] Diego` |
| PRD-FE-08 | docs/PRD.md | Fora de escopo (adiado) | Restrição do CRUD de webhook a perfis específicos | TRANSCRICAO | `[09:37] Sofia` |
| PRD-RF-01 | docs/PRD.md | Requisito Funcional | Notificar automaticamente o cliente a cada mudança de status de pedido | TRANSCRICAO | `[09:00] Marcos` |
| PRD-RF-02 | docs/PRD.md | Requisito Funcional | Cadastrar destino de notificação, recebendo a credencial no ato da criação | TRANSCRICAO | `[09:31] Marcos` |
| PRD-RF-03 | docs/PRD.md | Requisito Funcional | Consultar os destinos de notificação cadastrados | TRANSCRICAO | `[09:33] Bruno` |
| PRD-RF-04 | docs/PRD.md | Requisito Funcional | Alterar um destino já cadastrado | TRANSCRICAO | `[09:33] Bruno` |
| PRD-RF-05 | docs/PRD.md | Requisito Funcional | Remover um destino cadastrado | TRANSCRICAO | `[09:33] Bruno` |
| PRD-RF-06 | docs/PRD.md | Requisito Funcional | Cada destino recebe apenas os status que o cliente selecionou | TRANSCRICAO | `[09:33] Marcos` |
| PRD-RF-07 | docs/PRD.md | Requisito Funcional | Consultar histórico de entregas com resultado, payload, resposta e tempo | TRANSCRICAO | `[09:34] Marcos` |
| PRD-RF-08 | docs/PRD.md | Requisito Funcional | Trocar a credencial de assinatura, com a anterior válida por 24h | TRANSCRICAO | `[09:21] Sofia` |
| PRD-RF-09 | docs/PRD.md | Requisito Funcional | Assinar cada aviso para o cliente validar origem e integridade | TRANSCRICAO | `[09:19] Sofia` |
| PRD-RF-10 | docs/PRD.md | Requisito Funcional | Identificador único e estável por evento, mantido em qualquer reenvio | TRANSCRICAO | `[09:25] Diego` |
| PRD-RF-11 | docs/PRD.md | Requisito Funcional | Reenviar automaticamente os avisos que falharem, até um teto de tentativas | TRANSCRICAO | `[09:15] Diego` |
| PRD-RF-12 | docs/PRD.md | Requisito Funcional | Preservar avisos que falharam em definitivo, com o motivo da falha | TRANSCRICAO | `[09:18] Diego` |
| PRD-RF-13 | docs/PRD.md | Requisito Funcional | Reprocessamento manual por administrador, com registro de quem executou | TRANSCRICAO | `[09:36] Sofia` |
| PRD-RF-14 | docs/PRD.md | Requisito Funcional | O aviso reflete o estado do pedido no instante em que o status mudou | TRANSCRICAO | `[09:52] Larissa` |
| PRD-RF-15 | docs/PRD.md | Requisito Funcional | Aviso com dados essenciais do pedido, sem a lista de itens | TRANSCRICAO | `[09:43] Diego` |
| PRD-RNF-01 | docs/PRD.md | Requisito Não Funcional | Aviso entregue em menos de 10 segundos no caminho feliz | TRANSCRICAO | `[09:02] Marcos` |
| PRD-RNF-02 | docs/PRD.md | Requisito Não Funcional | Mudança de status não pode ser bloqueada por cliente lento ou indisponível | TRANSCRICAO | `[09:04] Bruno` |
| PRD-RNF-03 | docs/PRD.md | Requisito Não Funcional | Não pode existir mudança de status sem o evento correspondente registrado | TRANSCRICAO | `[09:40] Bruno` |
| PRD-RNF-04 | docs/PRD.md | Restrição | URL do destino obrigatoriamente HTTPS; cadastro sem TLS é recusado | TRANSCRICAO | `[09:23] Sofia` |
| PRD-RNF-05 | docs/PRD.md | Requisito Não Funcional | Credencial exclusiva por destino — vazamento de uma não compromete as demais | TRANSCRICAO | `[09:21] Sofia` |
| PRD-RNF-06 | docs/PRD.md | Restrição | Conteúdo do aviso limitado a 64KB; acima disso erra, nunca trunca | TRANSCRICAO | `[09:24] Diego` |
| PRD-RNF-07 | docs/PRD.md | Requisito Não Funcional | Cliente que não responder em 10 segundos tem a tentativa tratada como falha | TRANSCRICAO | `[09:42] Diego` |
| PRD-RNF-08 | docs/PRD.md | Requisito Não Funcional | Entrega at-least-once; dedup é responsabilidade do cliente | TRANSCRICAO | `[09:24] Diego` |
| PRD-RNF-09 | docs/PRD.md | Requisito Não Funcional | Ordem garantida por pedido, não entre pedidos diferentes | TRANSCRICAO | `[09:12] Diego` |
| PRD-RNF-10 | docs/PRD.md | Requisito Não Funcional | Reprocessamento exige perfil ADMIN e é registrado para auditoria | TRANSCRICAO | `[09:36] Sofia` |
| PRD-RNF-11 | docs/PRD.md | Requisito Não Funcional | CRUD de destinos exige autenticação, sem restrição de perfil nesta fase | TRANSCRICAO | `[09:37] Sofia` |
| PRD-DEC-01 | docs/PRD.md | Trade-off | Notificação assíncrona: perde imediatismo, ganha isolamento do pedido | TRANSCRICAO | `[09:06] Diego` |
| PRD-DEC-02 | docs/PRD.md | Trade-off | Desistência após 5 tentativas em ~15h; acima disso depende de replay manual | TRANSCRICAO | `[09:17] Marcos` |
| PRD-DEC-03 | docs/PRD.md | Trade-off | Cliente é responsável por descartar avisos repetidos | TRANSCRICAO | `[09:25] Sofia` |
| PRD-DEC-04 | docs/PRD.md | Trade-off | Grace period de 24h na troca de credencial mantém a antiga válida | TRANSCRICAO | `[09:21] Sofia` |
| PRD-DEC-05 | docs/PRD.md | Trade-off | Aviso descreve o pedido como ele era, não como está | TRANSCRICAO | `[09:52] Larissa` |
| PRD-DEP-01 | docs/PRD.md | Dependência | Documentação da integração no portal do desenvolvedor | TRANSCRICAO | `[09:26] Marcos` |
| PRD-DEP-02 | docs/PRD.md | Dependência | Revisão de segurança da Sofia, ≥2 dias úteis, bloqueante para o deploy | TRANSCRICAO | `[09:46] Sofia` |
| PRD-DEP-03 | docs/PRD.md | Dependência | Clientes implementarem verificação de assinatura e dedup do lado deles | TRANSCRICAO | `[09:20] Sofia` |
| PRD-DEP-04 | docs/PRD.md | Dependência | Capacidade do time por três sprints | TRANSCRICAO | `[09:46] Larissa` |
| PRD-DEP-05 | docs/PRD.md | Dependência | Infraestrutura capaz de manter um segundo processo no ar | TRANSCRICAO | `[09:11] Diego` |
| PRD-RSC-01 | docs/PRD.md | Risco | Perda da Atlas por atraso na entrega (Média/Alto) | TRANSCRICAO | `[09:00] Marcos` |
| PRD-RSC-02 | docs/PRD.md | Risco | Cliente processa o mesmo aviso duas vezes por não deduplicar (Média/Médio) | TRANSCRICAO | `[09:25] Sofia` |
| PRD-RSC-03 | docs/PRD.md | Risco | Avisos morrem na DLQ sem ninguém perceber (Alta/Médio) | TRANSCRICAO | `[09:37] Larissa` |
| PRD-RSC-04 | docs/PRD.md | Risco | Cliente fora do ar >15h perde avisos automaticamente (Baixa/Médio) | TRANSCRICAO | `[09:17] Marcos` |
| PRD-RSC-05 | docs/PRD.md | Risco | Vazamento de credencial de cliente (Média/Alto) | TRANSCRICAO | `[09:22] Diego` |
| PRD-RSC-06 | docs/PRD.md | Risco | Rajada de 50 avisos a um mesmo cliente sem rate limiting (Média/Baixo) | TRANSCRICAO | `[09:38] Diego` |
| PRD-CA-01 | docs/PRD.md | Critério de Aceitação | Aviso chega em menos de 10s quando há destino ativo escutando o status | TRANSCRICAO | `[09:02] Marcos` |
| PRD-CA-02 | docs/PRD.md | Critério de Aceitação | Aviso não é enviado para status fora dos selecionados no destino | TRANSCRICAO | `[09:34] Bruno` |
| PRD-CA-03 | docs/PRD.md | Critério de Aceitação | Credencial retornada apenas na criação, irrecuperável depois | TRANSCRICAO | `[09:31] Marcos` |
| PRD-CA-04 | docs/PRD.md | Critério de Aceitação | Consultar, alterar e remover destinos reflete na consulta seguinte | TRANSCRICAO | `[09:33] Bruno` |
| PRD-CA-05 | docs/PRD.md | Critério de Aceitação | Histórico exibe resultado, payload, resposta e tempo por tentativa | TRANSCRICAO | `[09:34] Marcos` |
| PRD-CA-06 | docs/PRD.md | Critério de Aceitação | Após a troca, credencial anterior vale 24h e deixa de valer depois | TRANSCRICAO | `[09:21] Sofia` |
| PRD-CA-07 | docs/PRD.md | Critério de Aceitação | Assinatura confere no cliente e falha se o conteúdo for alterado | TRANSCRICAO | `[09:20] Sofia` |
| PRD-CA-08 | docs/PRD.md | Critério de Aceitação | Reenvio e replay preservam o identificador de evento original | TRANSCRICAO | `[09:25] Diego` |
| PRD-CA-09 | docs/PRD.md | Critério de Aceitação | Até 5 tentativas com intervalos crescentes; depois preserva e para | TRANSCRICAO | `[09:17] Larissa` |
| PRD-CA-10 | docs/PRD.md | Critério de Aceitação | Destino que não responde em 10s tem a tentativa encerrada como falha | TRANSCRICAO | `[09:42] Diego` |
| PRD-CA-11 | docs/PRD.md | Critério de Aceitação | Replay recusado para não-administradores e registra quem executou | TRANSCRICAO | `[09:36] Sofia` |
| PRD-CA-12 | docs/PRD.md | Critério de Aceitação | Cadastro com endereço sem HTTPS é recusado com erro de validação | TRANSCRICAO | `[09:23] Sofia` |
| PRD-CA-13 | docs/PRD.md | Critério de Aceitação | Aviso atrasado apresenta os dados do instante da transição | TRANSCRICAO | `[09:52] Larissa` |
| PRD-CA-14 | docs/PRD.md | Critério de Aceitação | Se o registro do aviso falhar, a mudança de status não é efetivada | TRANSCRICAO | `[09:40] Bruno` |
| PRD-CA-15 | docs/PRD.md | Critério de Aceitação | Mudança de status não é atrasada por destino indisponível | TRANSCRICAO | `[09:04] Bruno` |
| PRD-CA-16 | docs/PRD.md | Critério de Aceitação | Conteúdo do aviso traz dados essenciais, sem a lista de itens | TRANSCRICAO | `[09:43] Diego` |
| PRD-Q-01 | docs/PRD.md | Questão em Aberto | Sem critério para quando o rate limiting passa a ser necessário | TRANSCRICAO | `[09:39] Larissa` |
| PRD-Q-02 | docs/PRD.md | Questão em Aberto | Endurecimento do controle de quem gerencia destinos, sem data | TRANSCRICAO | `[09:37] Sofia` |
| PRD-Q-03 | docs/PRD.md | Questão em Aberto | Expurgo dos eventos entregues, sem dono nem prazo | TRANSCRICAO | `[09:08] Diego` |
| PRD-Q-04 | docs/PRD.md | Questão em Aberto | Quando suportar múltiplos processos de entrega e o efeito na ordem | TRANSCRICAO | `[09:13] Diego` |
| PRD-Q-05 | docs/PRD.md | Questão em Aberto | Baseline do volume de polling nunca medido, impede meta para OBJ-03 | TRANSCRICAO | `[09:00] Marcos` |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo HTTP síncrono dentro de `changeStatus` | TRANSCRICAO | `[09:04] Bruno` |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams ou broker de mensageria externo | TRANSCRICAO | `[09:07] Diego` |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger no MySQL para notificar o worker reativamente | TRANSCRICAO | `[09:09] Diego` |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Garantia de entrega exactly-once | TRANSCRICAO | `[09:25] Diego` |
| RFC-Q-01 | docs/RFC.md | Questão em Aberto | Rate limiting de saída — observar e decidir depois | TRANSCRICAO | `[09:39] Diego` |
| RFC-Q-02 | docs/RFC.md | Questão em Aberto | Escala para múltiplos workers sem quebrar a ordenação | TRANSCRICAO | `[09:13] Diego` |
| RFC-Q-03 | docs/RFC.md | Questão em Aberto | Arquivamento das linhas entregues, sem dono nem prazo | TRANSCRICAO | `[09:08] Diego` |
| RFC-Q-04 | docs/RFC.md | Questão em Aberto | Nível de autorização do CRUD de webhook, sem critério nem data | TRANSCRICAO | `[09:37] Sofia` |
| RFC-RSC-01 | docs/RFC.md | Risco | Queda silenciosa do worker, sem alerta automático | TRANSCRICAO | `[09:11] Diego` |
| RFC-RSC-02 | docs/RFC.md | Risco | Falha na escrita da outbox derruba a mudança de status | TRANSCRICAO | `[09:40] Bruno` |
| RFC-RSC-03 | docs/RFC.md | Risco | Eventos morrem na DLQ sem gatilho automático para alguém olhar | TRANSCRICAO | `[09:18] Diego` |
| RFC-RSC-04 | docs/RFC.md | Risco | Crescimento indefinido da outbox sem arquivamento | TRANSCRICAO | `[09:08] Diego` |
| RFC-RSC-05 | docs/RFC.md | Risco | Secrets de cliente vazando em log — `redact` não cobre `secret` | CODIGO | `src/shared/logger/index.ts` |
| RFC-IMP-01 | docs/RFC.md | Restrição | Única alteração de comportamento em código existente é em `changeStatus` | CODIGO | `src/modules/orders/order.service.ts::changeStatus` |
| RFC-IMP-02 | docs/RFC.md | Restrição | `app.ts` e `routes/index.ts` mudam só para registrar o módulo | CODIGO | `src/app.ts`, `src/routes/index.ts` |
| RFC-IMP-03 | docs/RFC.md | Restrição | Projeto passa de uma para duas unidades de execução | CODIGO | `package.json` |
| FDD-OT-01 | docs/FDD.md | Objetivo Técnico | Toda transição válida com destino ativo gera 1 linha na outbox | TRANSCRICAO | `[09:06] Diego` |
| FDD-OT-02 | docs/FDD.md | Restrição | Nenhuma chamada HTTP a terceiro dentro de `prisma.$transaction` | TRANSCRICAO | `[09:04] Bruno` |
| FDD-OT-03 | docs/FDD.md | Objetivo Técnico | Worker iniciável por `npm run worker`, independente do processo HTTP | TRANSCRICAO | `[09:11] Diego` |
| FDD-OT-04 | docs/FDD.md | Objetivo Técnico | Latência ≤2s entre commit e primeira tentativa no caminho feliz | TRANSCRICAO | `[09:09] Diego` |
| FDD-OT-05 | docs/FDD.md | Objetivo Técnico | Todo request de saída carrega os 4 headers definidos | TRANSCRICAO | `[09:44] Diego` |
| FDD-OT-06 | docs/FDD.md | Restrição | Erros herdam de `AppError` sem alterar o middleware de erro | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-OT-07 | docs/FDD.md | Restrição | Nenhuma dependência npm nova — `node:crypto` e `fetch` nativo | CODIGO | `package.json` |
| FDD-MOD-01 | docs/FDD.md | Decisão | 4 tabelas novas, ids UUID `CHAR(36)` seguindo o padrão do projeto | CODIGO | `prisma/schema.prisma` |
| FDD-MOD-02 | docs/FDD.md | Decisão | Índices em status e `created_at` para sustentar o polling | TRANSCRICAO | `[09:08] Diego` |
| FDD-MOD-03 | docs/FDD.md | Decisão | Tabela `webhook_endpoints` guarda url, secret, customer_id e estado ativo | TRANSCRICAO | `[09:21] Bruno` |
| FDD-MOD-04 | docs/FDD.md | Decisão | Tabela `webhook_dead_letter` guarda payload, motivo da falha e timestamp | TRANSCRICAO | `[09:18] Diego` |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Criação do evento na outbox dentro da transação de `changeStatus` | CODIGO | `src/modules/orders/order.service.ts::changeStatus` |
| FDD-FLUXO-01b | docs/FDD.md | Decisão | Integração via `publishWebhookEvent(tx, ...)` recebendo o tx client | TRANSCRICAO | `[09:41] Bruno` |
| FDD-FLUXO-01c | docs/FDD.md | Decisão | Filtro de status aplicado na inserção, não no envio | TRANSCRICAO | `[09:34] Bruno` |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Ciclo de polling do worker a cada 2 segundos, em batch pequeno | TRANSCRICAO | `[09:09] Diego` |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Retry com backoff 1m/5m/30m/2h/12h, até 5 tentativas | TRANSCRICAO | `[09:17] Diego` |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | Movimentação para a DLQ após esgotar as tentativas | TRANSCRICAO | `[09:18] Diego` |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | Replay manual da DLQ recolocando o evento como pendente | TRANSCRICAO | `[09:18] Diego` |
| FDD-FLUXO-06 | docs/FDD.md | Fluxo | Rotação de secret com grace period de 24h | TRANSCRICAO | `[09:21] Sofia` |
| FDD-FLUXO-07 | docs/FDD.md | Fluxo | Cálculo da assinatura HMAC-SHA256 sobre o corpo do request | TRANSCRICAO | `[09:20] Sofia` |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | `POST /api/v1/webhooks` — cadastro, devolve a secret na criação | TRANSCRICAO | `[09:31] Marcos` |
| FDD-CONTRATO-01b | docs/FDD.md | Decisão | `customerId` vai no body, não vem do JWT do operador | TRANSCRICAO | `[09:32] Larissa` |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | `GET /api/v1/webhooks` — listagem paginada por cliente | TRANSCRICAO | `[09:33] Bruno` |
| FDD-CONTRATO-02b | docs/FDD.md | Decisão | Resposta paginada no formato `{ data, pagination }` já existente | CODIGO | `src/shared/http/response.ts::paginated` |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | `PATCH /api/v1/webhooks/:id` — edição de url, eventos e estado ativo | TRANSCRICAO | `[09:33] Bruno` |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | `DELETE /api/v1/webhooks/:id` — remoção, resposta 204 | TRANSCRICAO | `[09:33] Bruno` |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | `POST /api/v1/webhooks/:id/secret/rotate` — rotação de credencial | TRANSCRICAO | `[09:21] Sofia` |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | `GET /api/v1/webhooks/:id/deliveries` — histórico de entregas | TRANSCRICAO | `[09:34] Marcos` |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | `POST /api/v1/admin/webhooks/dead-letter/:id/replay` — exige ADMIN | TRANSCRICAO | `[09:35] Diego` |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Requisição de saída: payload JSON enxuto com campos do pedido | TRANSCRICAO | `[09:43] Diego` |
| FDD-CONTRATO-08b | docs/FDD.md | Contrato | Header `X-Event-Id` com UUID do evento | TRANSCRICAO | `[09:25] Diego` |
| FDD-CONTRATO-08c | docs/FDD.md | Contrato | Headers `X-Signature`, `X-Timestamp` e `Content-Type` | TRANSCRICAO | `[09:44] Diego` |
| FDD-CONTRATO-08d | docs/FDD.md | Contrato | Header `X-Webhook-Id` para clientes com múltiplos cadastros | TRANSCRICAO | `[09:44] Sofia` |
| FDD-CONTRATO-08e | docs/FDD.md | Restrição | Itens do pedido não vão no payload; detalhe fica em `GET /orders/:id` | TRANSCRICAO | `[09:43] Diego` |
| FDD-CONTRATO-08f | docs/FDD.md | Contrato | Formato de `order_number` segue o gerado pelo sistema (`ORD-000412`) | CODIGO | `src/modules/orders/order.service.ts::reserveOrderNumber` |
| FDD-ERRO-01 | docs/FDD.md | Erro | `WEBHOOK_NOT_FOUND` (404) — cadastro inexistente | TRANSCRICAO | `[09:28] Bruno` |
| FDD-ERRO-02 | docs/FDD.md | Erro | `WEBHOOK_INVALID_URL` (400) — URL malformada ou não-HTTPS | TRANSCRICAO | `[09:23] Sofia` |
| FDD-ERRO-03 | docs/FDD.md | Erro | `WEBHOOK_SECRET_REQUIRED` (400) — operação sem secret válida | TRANSCRICAO | `[09:28] Bruno` |
| FDD-ERRO-04 | docs/FDD.md | Erro | `WEBHOOK_INVALID_EVENT_TYPE` (400) — status fora do enum `OrderStatus` | CODIGO | `prisma/schema.prisma` |
| FDD-ERRO-05 | docs/FDD.md | Erro | `WEBHOOK_DEAD_LETTER_NOT_FOUND` (404) — item de DLQ inexistente | TRANSCRICAO | `[09:35] Diego` |
| FDD-ERRO-06 | docs/FDD.md | Erro | `WEBHOOK_PAYLOAD_TOO_LARGE` — payload acima de 64KB, erra sem truncar | TRANSCRICAO | `[09:24] Diego` |
| FDD-ERRO-07 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_TIMEOUT` — cliente não respondeu em 10 segundos | TRANSCRICAO | `[09:42] Diego` |
| FDD-ERRO-08 | docs/FDD.md | Erro | `WEBHOOK_DELIVERY_FAILED` — resposta não-2xx ou erro de rede | TRANSCRICAO | `[09:34] Marcos` |
| FDD-ERRO-09 | docs/FDD.md | Decisão | Prefixo `WEBHOOK_` para todos os códigos do módulo | TRANSCRICAO | `[09:29] Larissa` |
| FDD-ERRO-10 | docs/FDD.md | Decisão | Erros genéricos reaproveitados sem criar variantes `WEBHOOK_` | CODIGO | `src/shared/errors/http-errors.ts` |
| FDD-RESIL-01 | docs/FDD.md | Restrição | Timeout de 10 segundos por tentativa de entrega | TRANSCRICAO | `[09:42] Diego` |
| FDD-RESIL-02 | docs/FDD.md | Decisão | Número máximo de 5 tentativas | TRANSCRICAO | `[09:15] Diego` |
| FDD-RESIL-03 | docs/FDD.md | Decisão | Progressão de backoff 1min/5min/30min/2h/12h (~15h) | TRANSCRICAO | `[09:17] Diego` |
| FDD-RESIL-04 | docs/FDD.md | Decisão | Destino após esgotar tentativas é a tabela de dead letter | TRANSCRICAO | `[09:18] Diego` |
| FDD-RESIL-05 | docs/FDD.md | Decisão | Intervalo de polling de 2 segundos | TRANSCRICAO | `[09:09] Diego` |
| FDD-RESIL-06 | docs/FDD.md | Decisão | Garantia at-least-once com identificador estável entre reenvios | TRANSCRICAO | `[09:24] Diego` |
| FDD-RESIL-07 | docs/FDD.md | Decisão | Atomicidade: insert na outbox dentro da transação, falha causa rollback | TRANSCRICAO | `[09:41] Diego` |
| FDD-RESIL-08 | docs/FDD.md | Restrição | Teto de 64KB por payload, com erro em vez de truncamento | TRANSCRICAO | `[09:24] Larissa` |
| FDD-OBS-07 | docs/FDD.md | Observabilidade | Logs via Pino já existente, sem dependência nova | TRANSCRICAO | `[09:29] Bruno` |
| FDD-OBS-08 | docs/FDD.md | Observabilidade | Nomes de evento de log em `snake_case`, seguindo a convenção do projeto | CODIGO | `src/server.ts` |
| FDD-OBS-09 | docs/FDD.md | Observabilidade | Log de auditoria do replay com o usuário que executou | TRANSCRICAO | `[09:36] Sofia` |
| FDD-OBS-10 | docs/FDD.md | Restrição | `redactPaths` não cobre `secret` hoje e precisa ser estendido | CODIGO | `src/shared/logger/index.ts` |
| FDD-OBS-11 | docs/FDD.md | Restrição | Projeto não tem tracing distribuído; correlação via `requestId` existente | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-DEP-01 | docs/FDD.md | Dependência | MySQL via Prisma 5.22.0, com migration nova sem alterar tabelas atuais | CODIGO | `prisma/schema.prisma` |
| FDD-DEP-02 | docs/FDD.md | Dependência | HMAC-SHA256 via `node:crypto`, sem pacote novo | TRANSCRICAO | `[09:20] Sofia` |
| FDD-DEP-03 | docs/FDD.md | Dependência | `fetch` global disponível sob `"node": ">=20"` declarado em engines | CODIGO | `package.json` |
| FDD-DEP-04 | docs/FDD.md | Dependência | Geração de UUID via `uuid` já presente ou `@default(uuid())` do Prisma | CODIGO | `package.json` |
| FDD-DEP-05 | docs/FDD.md | Dependência | Logging com `pino` já em uso, sem mudança de versão | CODIGO | `package.json` |
| FDD-DEP-06 | docs/FDD.md | Dependência | Validação com `zod` já em uso | CODIGO | `package.json` |
| FDD-DEP-07 | docs/FDD.md | Dependência | Script `npm run worker` e segunda unidade de deploy | TRANSCRICAO | `[09:11] Larissa` |
| FDD-CA-01 | docs/FDD.md | Critério de Aceite | Transição com destino ativo cria 1 linha `PENDING` na mesma transação | TRANSCRICAO | `[09:06] Diego` |
| FDD-CA-02 | docs/FDD.md | Critério de Aceite | Cliente sem destino escutando o status não gera linha na outbox | TRANSCRICAO | `[09:34] Bruno` |
| FDD-CA-03 | docs/FDD.md | Critério de Aceite | Falha na inserção da outbox faz rollback da mudança de status | TRANSCRICAO | `[09:40] Bruno` |
| FDD-CA-04 | docs/FDD.md | Critério de Aceite | Entrega ocorre em no máximo 2 segundos após o commit | TRANSCRICAO | `[09:09] Diego` |
| FDD-CA-05 | docs/FDD.md | Critério de Aceite | 5 falhas levam o evento à DLQ, marcado `FAILED` na outbox | TRANSCRICAO | `[09:17] Larissa` |
| FDD-CA-06 | docs/FDD.md | Critério de Aceite | Tentativa abortada em 10s é registrada como timeout | TRANSCRICAO | `[09:42] Diego` |
| FDD-CA-07 | docs/FDD.md | Critério de Aceite | HMAC calculado pelo cliente é idêntico ao enviado em `X-Signature` | TRANSCRICAO | `[09:20] Sofia` |
| FDD-CA-08 | docs/FDD.md | Critério de Aceite | Retry e replay preservam o mesmo `X-Event-Id` | TRANSCRICAO | `[09:25] Diego` |
| FDD-CA-09 | docs/FDD.md | Critério de Aceite | Secret anterior deixa de existir após 24 horas da rotação | TRANSCRICAO | `[09:21] Sofia` |
| FDD-CA-10 | docs/FDD.md | Critério de Aceite | `OPERATOR` chamando o replay recebe 403 | CODIGO | `src/middlewares/auth.middleware.ts::requireRole` |
| FDD-CA-11 | docs/FDD.md | Critério de Aceite | Replay grava `replayedById` e emite log de auditoria | TRANSCRICAO | `[09:36] Sofia` |
| FDD-CA-12 | docs/FDD.md | Critério de Aceite | URL `http://` no cadastro retorna 400 `WEBHOOK_INVALID_URL` | TRANSCRICAO | `[09:23] Sofia` |
| FDD-CA-13 | docs/FDD.md | Critério de Aceite | `npm run lint` e `npm run test` passam sem dependência nova | CODIGO | `package.json` |
| FDD-RSC-03 | docs/FDD.md | Risco | Rajada ao worker voltar de parada longa sobrecarrega os clientes | TRANSCRICAO | `[09:38] Diego` |
| FDD-RSC-04 | docs/FDD.md | Risco | Escrita na outbox aumenta o tempo de lock na transação de estoque | CODIGO | `src/modules/orders/order.service.ts::debitStock` |
| FDD-RSC-05 | docs/FDD.md | Risco | Evento morto na DLQ sem alerta, com replay apenas manual | TRANSCRICAO | `[09:37] Larissa` |
| FDD-RSC-06 | docs/FDD.md | Risco | Secret de cliente gravada em log por falta de `redact` | CODIGO | `src/shared/logger/index.ts` |
| FDD-INT-01 | docs/FDD.md | Integração | `changeStatus` passa a chamar `publishWebhookEvent` na mesma transação | CODIGO | `src/modules/orders/order.service.ts::changeStatus` |
| FDD-INT-02 | docs/FDD.md | Integração | Erros do módulo herdam de `AppError` com `statusCode`/`errorCode` | CODIGO | `src/shared/errors/app-error.ts::AppError` |
| FDD-INT-03 | docs/FDD.md | Integração | Middleware de erro já formata `AppError` — nenhuma alteração necessária | CODIGO | `src/middlewares/error.middleware.ts::errorMiddleware` |
| FDD-INT-04 | docs/FDD.md | Integração | `requireRole('ADMIN')` aplicado na rota de replay de DLQ | CODIGO | `src/middlewares/auth.middleware.ts::requireRole` |
| FDD-INT-05 | docs/FDD.md | Integração | Schemas Zod validados pelo middleware `validate` existente | CODIGO | `src/middlewares/validate.middleware.ts::validate` |
| FDD-INT-06 | docs/FDD.md | Integração | Logger Pino reutilizado; `redactPaths` estendido para `secret` | CODIGO | `src/shared/logger/index.ts::logger` |
| FDD-INT-07 | docs/FDD.md | Integração | Worker instancia o próprio client pela factory existente | CODIGO | `src/config/database.ts::createPrismaClient` |
| FDD-INT-08 | docs/FDD.md | Integração | `src/server.ts` serve de molde para o entrypoint do worker | CODIGO | `src/server.ts` |
| FDD-INT-09 | docs/FDD.md | Integração | `buildControllers` e `buildApiRouter` registram o módulo novo | CODIGO | `src/app.ts::buildControllers` |
| FDD-INT-10 | docs/FDD.md | Integração | `prisma/schema.prisma` ganha 4 modelos, sem alterar os existentes | CODIGO | `prisma/schema.prisma` |
| FDD-INT-11 | docs/FDD.md | Integração | Helper `paginated` reutilizado nas listagens do módulo | CODIGO | `src/shared/http/response.ts::paginated` |
| FDD-INT-12 | docs/FDD.md | Integração | `canTransition` é a fonte de verdade dos status aceitos em `events` | CODIGO | `src/modules/orders/order.status.ts::canTransition` |
| FDD-Q-01 | docs/FDD.md | Questão em Aberto | Onde o teto de 64KB é aplicado: na inserção ou no envio | TRANSCRICAO | `[09:23] Sofia` |
| FDD-Q-02 | docs/FDD.md | Questão em Aberto | Secrets cifradas em repouso ou em texto — não definido | TRANSCRICAO | `[09:46] Sofia` |
| FDD-Q-03 | docs/FDD.md | Questão em Aberto | Tracing distribuído: projeto não tem instrumentação hoje | CODIGO | `package.json` |
| FDD-Q-04 | docs/FDD.md | Questão em Aberto | Retenção do histórico de entregas, sem janela definida | TRANSCRICAO | `[09:34] Marcos` |
| FDD-Q-05 | docs/FDD.md | Questão em Aberto | Supervisão do processo worker (restart, orquestrador) não definida | TRANSCRICAO | `[09:11] Diego` |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Padrão Outbox no MySQL, insert na mesma transação da mudança de status | TRANSCRICAO | `[09:06] Diego` |
| ADR-001-CTX-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Restrição | `changeStatus` já executa 4 escritas acopladas em uma transação | CODIGO | `src/modules/orders/order.service.ts::changeStatus` |
| ADR-001-ALT-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Disparo HTTP síncrono — trava outros pedidos e forçaria rollback | TRANSCRICAO | `[09:04] Bruno` |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Redis Streams — infra nova, overengineering para time pequeno | TRANSCRICAO | `[09:07] Diego` |
| ADR-001-CONSEQ-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência (negativa) | Disponibilidade de `changeStatus` passa a depender da escrita na outbox | TRANSCRICAO | `[09:40] Bruno` |
| ADR-001-CONSEQ-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Consequência (negativa) | Crescimento da tabela; arquivamento fora do escopo desta feature | TRANSCRICAO | `[09:08] Diego` |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Worker em processo separado com polling de 2 segundos | TRANSCRICAO | `[09:09] Diego` |
| ADR-002-CTX-01 | docs/adrs/ADR-002-worker-separado-em-polling.md | Restrição | Projeto tem hoje um único entrypoint, sem processo de background | CODIGO | `src/server.ts` |
| ADR-002-ALT-01 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Trigger no MySQL — não há `LISTEN/NOTIFY` como no PostgreSQL | TRANSCRICAO | `[09:09] Diego` |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Worker dentro do processo da API — restart da API derruba a entrega | TRANSCRICAO | `[09:11] Diego` |
| ADR-002-CONSEQ-01 | docs/adrs/ADR-002-worker-separado-em-polling.md | Consequência (positiva) | Latência de fila ≤2s, dentro do orçamento de 10s | TRANSCRICAO | `[09:10] Larissa` |
| ADR-002-CONSEQ-02 | docs/adrs/ADR-002-worker-separado-em-polling.md | Consequência (negativa) | Single-worker é SPOF; escalar quebra a ordenação | TRANSCRICAO | `[09:13] Diego` |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Retry com backoff 1m/5m/30m/2h/12h, 5 tentativas, depois DLQ | TRANSCRICAO | `[09:17] Larissa` |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa descartada | 3 tentativas — não cobre indisponibilidade de 2h já ocorrida | TRANSCRICAO | `[09:16] Diego` |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa descartada | Retry indefinido — evento fica pendurado se o cliente sumiu | TRANSCRICAO | `[09:15] Diego` |
| ADR-003-ALT-03 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa descartada | Marcar `failed` na própria outbox — polui a leitura da tabela | TRANSCRICAO | `[09:18] Diego` |
| ADR-003-CONSEQ-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência (positiva) | Janela de ~15h aceita explicitamente pelo PM | TRANSCRICAO | `[09:17] Marcos` |
| ADR-003-CONSEQ-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Consequência (negativa) | Recuperação depende de alguém perceber; não há alerta nesta fase | TRANSCRICAO | `[09:37] Larissa` |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Decisão | HMAC-SHA256 com secret única por endpoint e rotação com grace de 24h | TRANSCRICAO | `[09:22] Sofia` |
| ADR-004-CTX-01 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Contexto | Escopo exclusivamente outbound dispensa verificação no sentido inverso | TRANSCRICAO | `[09:03] Sofia` |
| ADR-004-ALT-01 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Alternativa descartada | Secret global da plataforma — "se vaza uma, vaza tudo" | TRANSCRICAO | `[09:21] Sofia` |
| ADR-004-CONSEQ-01 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Consequência (positiva) | Rotação motivada por caso real de secret vazada em log de cliente | TRANSCRICAO | `[09:22] Diego` |
| ADR-004-CONSEQ-02 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Consequência (negativa) | `redactPaths` do Pino não cobre `secret` — registraria a chave em log | CODIGO | `src/shared/logger/index.ts` |
| ADR-004-CONSEQ-03 | docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md | Consequência (negativa) | Revisão de segurança bloqueante consome ≥2 dias úteis do cronograma | TRANSCRICAO | `[09:46] Sofia` |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md | Decisão | Entrega at-least-once com dedup pelo cliente via `X-Event-Id` | TRANSCRICAO | `[09:25] Diego` |
| ADR-005-ALT-01 | docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md | Alternativa descartada | Exactly-once — exigiria coordenação dos dois lados | TRANSCRICAO | `[09:25] Diego` |
| ADR-005-CONSEQ-01 | docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md | Consequência (negativa) | Transfere trabalho e risco de dedup para o cliente | TRANSCRICAO | `[09:25] Sofia` |
| ADR-005-CONSEQ-02 | docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md | Consequência (negativa) | Cria dependência de documentação no portal do desenvolvedor | TRANSCRICAO | `[09:26] Marcos` |
| ADR-006 | docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md | Decisão | Payload renderizado e persistido na inserção (snapshot do instante) | TRANSCRICAO | `[09:52] Larissa` |
| ADR-006-CTX-01 | docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md | Decisão | Id da outbox é UUID, seguindo o padrão do resto do projeto | CODIGO | `prisma/schema.prisma` |
| ADR-006-ALT-01 | docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md | Alternativa descartada | Guardar só `order_id` e renderizar no envio — geraria "caso esquisito" | TRANSCRICAO | `[09:51] Bruno` |
| ADR-006-CONSEQ-01 | docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md | Consequência (negativa) | Cliente pode receber dados desatualizados em entrega muito atrasada | TRANSCRICAO | `[09:43] Diego` |
| ADR-006-CONSEQ-02 | docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md | Consequência (negativa) | Linha maior na outbox, o que torna necessário o teto de 64KB | TRANSCRICAO | `[09:24] Diego` |
| ADR-007 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Decisão | Webhooks como módulo convencional, reusando os padrões do projeto | TRANSCRICAO | `[09:30] Larissa` |
| ADR-007-CTX-01 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Restrição | Padrão de módulo controller/service/repository/routes/schemas | CODIGO | `src/modules/orders/` |
| ADR-007-INT-01 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Decisão | Módulo em `src/modules/webhooks` seguindo a mesma divisão | TRANSCRICAO | `[09:27] Bruno` |
| ADR-007-INT-02 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Decisão | Códigos de erro no molde de `INSUFFICIENT_STOCK` com prefixo `WEBHOOK_` | CODIGO | `src/shared/errors/http-errors.ts` |
| ADR-007-INT-03 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Decisão | Middleware de erro centralizado não precisa de alteração | TRANSCRICAO | `[09:29] Bruno` |
| ADR-007-INT-04 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Decisão | `requireRole` reaproveitado para exigir ADMIN no replay | TRANSCRICAO | `[09:36] Larissa` |
| ADR-007-ALT-01 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Alternativa descartada | Compartilhar o mesmo `PrismaClient` — é por processo, não compartilhável | TRANSCRICAO | `[09:30] Bruno` |
| ADR-007-CONSEQ-01 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Consequência (negativa) | Padrão reaproveitado é HTTP-first e não cobre o caminho do worker | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR-007-CONSEQ-02 | docs/adrs/ADR-007-reuso-dos-padroes-existentes.md | Consequência (negativa) | `app.ts` e `routes/index.ts` precisam ser tocados para registrar o módulo | CODIGO | `src/app.ts` |

---

## 2. Itens sem origem rastreável (premissas e lacunas)

Estes itens **não** aparecem na tabela principal porque não resolvem para `TRANSCRICAO` nem para `CODIGO`. Estão registrados aqui, marcados nos documentos de origem com ⚠️, em vez de receberem uma origem forçada — é exatamente o comportamento que o tracker existe para garantir.

| ID | Documento | Tipo | Conteúdo (resumo) | Por que não é rastreável |
| -- | --------- | ---- | ----------------- | ------------------------ |
| PRD-P-01 | docs/PRD.md | ⚠️ Premissa | Data completa da reunião assumida; a fonte registra só "quinta-feira, 09:00" | A transcrição não traz a data; sem impacto sobre requisitos |
| PRD-P-02 | docs/PRD.md | ⚠️ Premissa | "Fim do trimestre" e "fim de novembro" tratados como o mesmo marco | Os dois prazos aparecem na fonte (`[09:00] Marcos`, `[09:45] Marcos`) mas nunca são conciliados |
| RFC-P-01 | docs/RFC.md | ⚠️ Premissa | Armazenamento das secrets em repouso (cifrado ou texto) | A reunião definiu secret por endpoint e rotação, mas nunca tratou do armazenamento |
| RFC-P-02 | docs/RFC.md | ⚠️ Premissa | Supervisão do processo worker (restart, orquestrador) | A reunião decidiu o processo separado, sem definir quem o mantém no ar |
| RFC-P-03 | docs/RFC.md | ⚠️ Premissa | Datas dos documentos referem-se à redação, não à reunião | Mesma lacuna de PRD-P-01 |
| RFC-RSC-01-MIT | docs/RFC.md | ⚠️ Mitigação não discutida | Heartbeat do worker e alarme sobre idade do evento pendente mais antigo | O risco vem da fonte; a mitigação foi derivada, não discutida na reunião |
| RFC-RSC-03-MIT | docs/RFC.md | ⚠️ Mitigação não discutida | Métrica de contagem da DLQ com alarme | Monitoramento não foi discutido em nenhum momento da reunião |
| FDD-RESIL-09 | docs/FDD.md | ⚠️ Premissa | Claim condicional `PENDING → PROCESSING` antes do envio | Mecanismo necessário para sustentar o single-worker sob restart; não discutido |
| FDD-RESIL-10 | docs/FDD.md | ⚠️ Premissa | Recuperação de eventos presos em `PROCESSING` há mais de 60s | Decorre de FDD-RESIL-09; não discutido |
| FDD-ERRO-09b | docs/FDD.md | ⚠️ Premissa | `WEBHOOK_INACTIVE` (409) para destino com `active = false` | O campo `active` vem de `[09:21] Bruno`, mas o cenário nunca foi tratado |
| FDD-ERRO-10b | docs/FDD.md | ⚠️ Premissa | `WEBHOOK_ROTATION_IN_PROGRESS` (409) para rotações encadeadas | O grace period vem de `[09:21] Sofia`, mas rotações sobrepostas não foram tratadas |
| FDD-OBS-01..06 | docs/FDD.md | ⚠️ Premissa | Nomes e tipos das 6 métricas propostas | Instrumentação de métricas não foi discutida na reunião |
| FDD-RSC-01 | docs/FDD.md | ⚠️ Premissa | Risco de duas instâncias do worker processando o mesmo evento | Derivado de FDD-RESIL-09; a reunião assumiu single-worker sem tratar o caso |
| FDD-RSC-02 | docs/FDD.md | ⚠️ Premissa | Risco de evento preso em `PROCESSING` | Derivado de FDD-RESIL-09 |
| FDD-P-04 | docs/FDD.md | ⚠️ Premissa | Prefixo `whsec_` no formato textual da secret | Convenção de mercado adotada; a fonte só define que a secret é gerada por nós |
| ADR-004-ALT-02 | docs/adrs/ADR-004-...md | ⚠️ Alternativa não discutida | mTLS ou assinatura assimétrica | Incluída por ser a alternativa de mercado mais séria ao HMAC; não debatida na reunião |
| ADR-005-ALT-02 | docs/adrs/ADR-005-...md | ⚠️ Alternativa não discutida | Deduplicação do nosso lado antes do envio | Incluída como resposta à objeção de Sofia em `[09:25]`; não debatida |
| ADR-007-ALT-02 | docs/adrs/ADR-007-...md | ⚠️ Alternativa não discutida | Hierarquia de erros própria do módulo | Incluída porque o worker não é handler HTTP; refutada pelo código, não pela reunião |

---

## 3. Cobertura

Contagem executada sobre a tabela final da seção 1, não estimada:

| Métrica | Valor | Piso exigido | Situação |
| --- | --- | --- | --- |
| Linhas na matriz principal | **236** | — | — |
| Linhas com Fonte = `TRANSCRICAO` | **189** (80,1%) | ≥ 70% | ✅ |
| Linhas com Fonte = `CODIGO` | **47** (19,9%) | ≥ 5 | ✅ |
| Linhas com Fonte fora de `TRANSCRICAO`/`CODIGO` | **0** | 0 | ✅ |
| Itens identificáveis com linha correspondente | **236 de 254** (92,9%) | ≥ 80% | ✅ |
| Itens sem origem rastreável, isolados na seção 2 | 18 (7,1%) | — | — |
| ADRs com pelo menos uma linha | **7 de 7** | 7 de 7 | ✅ |

**Distribuição por documento de origem:** PRD 79 linhas · RFC 16 · FDD 99 · ADRs 42.

Todos os timestamps da coluna Localização foram conferidos contra `TRANSCRICAO.md`, incluindo o nome do falante; todos os caminhos de código foram confirmados como existentes no repositório. Os únicos caminhos citados nos documentos que ainda não existem — `src/worker.ts` e `src/modules/webhooks/webhook.worker.ts` — são arquivos a serem **criados** pela feature, nomeados na própria reunião (`[09:11] Larissa`, `[09:28] Bruno`), e por isso nunca aparecem marcados como `[CODIGO ...]`.
