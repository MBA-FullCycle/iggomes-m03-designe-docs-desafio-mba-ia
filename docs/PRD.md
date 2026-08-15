# PRD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| **Status** | Rascunho — aguardando revisão dos participantes da reunião |
| **Autor(es)** | Derivado da reunião técnica de quinta-feira, 09:00, conduzida por Larissa (Tech Lead), com Marcos (Product Manager), Bruno (Eng. Pleno — Time de Pedidos), Diego (Eng. Sênior — Plataforma) e Sofia (Eng. de Segurança) |
| **Data** | 2026-08-14 |
| **Fontes** | Primária: [`TRANSCRICAO.md`](../TRANSCRICAO.md) · Secundária: código-base do Order Management System |
| **Documentos relacionados** | [RFC](./RFC.md) · [FDD](./FDD.md) · [ADR-001 a ADR-007](./adrs/) · [TRACKER](./TRACKER.md) |

> Convenção de origem usada neste documento: `[TRANSCRICAO hh:mm Nome]` para falas da reunião,
> `[CODIGO caminho/do/arquivo]` para o código existente. Itens marcados com ⚠️ PREMISSA não têm
> origem direta na fonte e precisam de confirmação.

---

## 1. Resumo e contexto da feature

O Order Management System (OMS) controla hoje todo o ciclo de vida do pedido — criação, pagamento, processamento, envio, entrega e cancelamento — com máquina de estados e histórico de mudanças `[CODIGO src/modules/orders/order.status.ts]`. O que ele **não** faz é avisar ninguém quando esse ciclo avança: não existe mecanismo de notificação externa, evento ou webhook no sistema `[CODIGO package.json]`.

Esta feature preenche exatamente esse vácuo. O OMS passa a notificar automaticamente os clientes B2B, por HTTP, sempre que o status de um pedido deles muda. O cliente cadastra um endereço, escolhe quais status quer receber, e passa a ser avisado em vez de ficar perguntando.

A demanda é formal e nominal: Atlas Comercial, MaxDistribuição e Nova Cargo pediram a capacidade na semana anterior à reunião `[TRANSCRICAO 09:00 Marcos]`. A Atlas condicionou a permanência na plataforma à entrega até o fim do trimestre.

## 2. Problema e motivação

**Problema.** Os clientes B2B descobrem mudanças de status batendo repetidamente em `GET /orders` até notar que algo mudou. É polling puro, feito por eles, contra a nossa API `[TRANSCRICAO 09:00 Marcos]`, `[CODIGO src/modules/orders/order.routes.ts]`.

**Quem é afetado.** Três clientes B2B nomeados na reunião — Atlas Comercial, MaxDistribuição e Nova Cargo — e, indiretamente, a própria plataforma, que absorve o custo de servir consultas repetidas que na maioria das vezes não retornam novidade `[TRANSCRICAO 09:00 Marcos]`.

**Custo de não resolver.** A integração fica "lenta e cara" para o cliente, nas palavras do PM, e obriga operação manual do lado dele. O custo concreto e declarado é churn: a Atlas sinalizou que pode migrar para o concorrente se a capacidade não for entregue até o fim do trimestre `[TRANSCRICAO 09:00 Marcos]`.

## 3. Público-alvo e cenários de uso

| Perfil | Necessidade | Cenário de uso | Origem |
| --- | --- | --- | --- |
| Cliente B2B integrado via API (Atlas, MaxDistribuição, Nova Cargo) | Saber, sem perguntar, quando um pedido dele muda de status | O sistema do cliente recebe o aviso de `SHIPPED` e dispara automaticamente o processo interno de conferência, sem ninguém consultando a nossa API | `[TRANSCRICAO 09:00 Marcos]`, `[TRANSCRICAO 09:02 Marcos]` |
| Cliente B2B com múltiplas integrações | Direcionar status diferentes para destinos diferentes e saber qual cadastro recebeu cada aviso | O cliente cadastra um endpoint que só escuta `SHIPPED` e `DELIVERED` para o time de logística, e outro para o financeiro | `[TRANSCRICAO 09:33 Marcos]`, `[TRANSCRICAO 09:44 Sofia]` |
| Cliente B2B diagnosticando a própria integração | Entender por que um aviso não chegou | Após uma indisponibilidade, o cliente consulta o histórico de entregas e vê quais tentativas falharam, com que resposta e em quanto tempo | `[TRANSCRICAO 09:34 Marcos]` |
| Operador/administrador da plataforma | Recuperar avisos que falharam em definitivo | Um cliente ficou 20 horas fora; o administrador reprocessa manualmente os eventos que morreram | `[TRANSCRICAO 09:35 Diego]`, `[TRANSCRICAO 09:36 Sofia]` |

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Valor atual | Meta | Como medir | Origem |
| --- | --- | --- | --- | --- | --- | --- |
| OBJ-01 | Entregar a notificação dentro do que o cliente considera "tempo real" | Tempo entre a mudança de status e a chegada do aviso ao cliente | Inexistente (não há notificação hoje) | **< 10 segundos**, com alvo interno de ~2s no caminho feliz | Diferença entre o horário do commit da mudança e o da entrega bem-sucedida | `[TRANSCRICAO 09:02 Marcos]`, `[TRANSCRICAO 09:10 Larissa]` |
| OBJ-02 | Reter os clientes B2B que ameaçaram migrar | Clientes B2B integrados via webhook | 0 | **3 clientes** (Atlas, MaxDistribuição, Nova Cargo) integrados até o fim de novembro | Endpoints ativos cadastrados por cliente | `[TRANSCRICAO 09:00 Marcos]`, `[TRANSCRICAO 09:45 Marcos]` |
| OBJ-03 | Eliminar o polling que os clientes fazem hoje | Volume de chamadas a `GET /orders` originadas dos três clientes | Não medido | Redução substancial após a integração — sem meta numérica definida na reunião | Contagem de requisições por cliente | `[TRANSCRICAO 09:00 Marcos]` |
| OBJ-04 | Entregar dentro da capacidade estimada do time | Sprints até o deploy, incluindo revisão de segurança | — | **3 sprints** | Acompanhamento de sprint | `[TRANSCRICAO 09:46 Larissa]` |

> OBJ-03 não tem meta quantitativa porque a reunião não definiu uma; o baseline também nunca foi medido. Registrado como lacuna em Q-05, não preenchido com número inventado.

## 5. Escopo

### 5.1 Incluso nesta fase

- Notificação automática por HTTP a cada mudança de status de pedido, para os clientes que a tiverem configurado — `[TRANSCRICAO 09:00 Marcos]`
- Autogestão do cadastro de webhooks pela API: criar, listar, editar e remover — `[TRANSCRICAO 09:31 Marcos]`, `[TRANSCRICAO 09:33 Bruno]`
- Seleção, por endpoint, de quais status o cliente quer receber — `[TRANSCRICAO 09:33 Marcos]`
- Assinatura de cada notificação, com credencial exclusiva por endpoint e rotação autônoma — `[TRANSCRICAO 09:20 Sofia]`, `[TRANSCRICAO 09:21 Sofia]`
- Reenvio automático em caso de falha, com desistência após um teto de tentativas — `[TRANSCRICAO 09:15 Diego]`
- Consulta ao histórico de entregas por endpoint — `[TRANSCRICAO 09:34 Marcos]`
- Reprocessamento manual, restrito a administradores e auditado, de avisos que falharam em definitivo — `[TRANSCRICAO 09:35 Diego]`, `[TRANSCRICAO 09:36 Sofia]`

### 5.2 Fora de escopo

| ID | Item | Motivo | Situação | Origem |
| --- | --- | --- | --- | --- |
| FE-01 | Aviso por e-mail ao cliente quando o webhook dele está falhando | Fica para a fase seguinte, depois de medir o impacto real | **Adiado** | `[TRANSCRICAO 09:37 Marcos]`, `[TRANSCRICAO 09:37 Larissa]` |
| FE-02 | Painel/dashboard visual para o cliente acompanhar os webhooks | Nesta fase a entrega é só de endpoints; painel é projeto separado do time de frontend | **Descartado nesta fase** | `[TRANSCRICAO 09:39 Marcos]`, `[TRANSCRICAO 09:40 Larissa]` |
| FE-03 | Limitação da taxa de envio para um mesmo cliente | Decidido observar em produção e implementar só se virar problema real | **Adiado** | `[TRANSCRICAO 09:38 Diego]`, `[TRANSCRICAO 09:39 Larissa]` |
| FE-04 | Webhooks de entrada (cliente enviando eventos para nós) | Os clientes querem receber, não enviar; escopo é exclusivamente de saída | **Descartado** | `[TRANSCRICAO 09:02 Sofia]`, `[TRANSCRICAO 09:02 Marcos]` |
| FE-05 | Garantia de que um aviso nunca chega duplicado | Exigiria coordenação entre os dois lados; o cliente passa a ser responsável por descartar repetições | **Descartado** | `[TRANSCRICAO 09:25 Diego]` |
| FE-06 | Garantia de ordem global entre pedidos diferentes | Os clientes nunca pediram; a ordem é garantida por pedido | **Descartado nesta fase** | `[TRANSCRICAO 09:13 Larissa]`, `[TRANSCRICAO 09:14 Marcos]` |
| FE-07 | Expurgo/arquivamento dos registros de eventos já entregues | Reconhecido como necessário, deixado explicitamente fora desta feature | **Adiado** | `[TRANSCRICAO 09:08 Diego]` |
| FE-08 | Restrição do cadastro de webhooks a perfis específicos | Por enquanto qualquer perfil autenticado pode gerenciar; endurecimento admitido para o futuro | **Adiado** | `[TRANSCRICAO 09:36 Marcos]`, `[TRANSCRICAO 09:37 Sofia]` |

## 6. Requisitos funcionais

| ID | Requisito | Prioridade | Origem |
| --- | --- | --- | --- |
| RF-01 | O sistema deve notificar automaticamente o cliente sempre que o status de um pedido dele mudar | Must | `[TRANSCRICAO 09:00 Marcos]` |
| RF-02 | O cliente deve conseguir cadastrar um destino de notificação, informando o endereço e os status que quer receber, e recebe no ato a credencial usada para validar os avisos | Must | `[TRANSCRICAO 09:31 Marcos]` |
| RF-03 | O cliente deve conseguir consultar os destinos de notificação já cadastrados | Must | `[TRANSCRICAO 09:33 Bruno]` |
| RF-04 | O cliente deve conseguir alterar um destino já cadastrado | Must | `[TRANSCRICAO 09:33 Bruno]` |
| RF-05 | O cliente deve conseguir remover um destino cadastrado | Must | `[TRANSCRICAO 09:33 Bruno]` |
| RF-06 | Cada destino deve receber apenas os status que o cliente selecionou para ele | Must | `[TRANSCRICAO 09:33 Marcos]`, `[TRANSCRICAO 09:34 Bruno]` |
| RF-07 | O cliente deve conseguir consultar o histórico de entregas de um destino, com resultado, conteúdo enviado, resposta recebida e tempo de resposta | Must | `[TRANSCRICAO 09:34 Marcos]` |
| RF-08 | O cliente deve conseguir solicitar a troca da credencial de assinatura, mantendo a anterior válida por 24 horas para migrar seus sistemas | Must | `[TRANSCRICAO 09:21 Sofia]` |
| RF-09 | Cada aviso enviado deve ser assinado, permitindo ao cliente verificar que veio de nós e que não foi adulterado | Must | `[TRANSCRICAO 09:19 Sofia]`, `[TRANSCRICAO 09:20 Sofia]` |
| RF-10 | Cada aviso deve carregar um identificador único e estável, que permanece o mesmo em qualquer reenvio do mesmo evento | Must | `[TRANSCRICAO 09:25 Diego]` |
| RF-11 | O sistema deve reenviar automaticamente os avisos que falharem, até um teto de tentativas espaçadas ao longo de aproximadamente 15 horas | Must | `[TRANSCRICAO 09:15 Diego]`, `[TRANSCRICAO 09:17 Diego]` |
| RF-12 | Avisos que falharem em todas as tentativas devem ser preservados com o motivo da falha, para consulta e reprocessamento | Must | `[TRANSCRICAO 09:18 Diego]` |
| RF-13 | Um administrador deve conseguir reprocessar manualmente um aviso que falhou em definitivo, e o sistema deve registrar quem executou a operação | Must | `[TRANSCRICAO 09:35 Diego]`, `[TRANSCRICAO 09:36 Sofia]` |
| RF-14 | O conteúdo do aviso deve refletir o estado do pedido no instante em que o status mudou, mesmo que a entrega ocorra horas depois | Must | `[TRANSCRICAO 09:52 Larissa]` |
| RF-15 | O aviso deve conter os dados essenciais do pedido e da transição, sem a lista de itens; o detalhamento continua disponível na consulta ao pedido | Should | `[TRANSCRICAO 09:43 Diego]` |

## 7. Requisitos não funcionais

| ID | Requisito | Categoria | Origem |
| --- | --- | --- | --- |
| RNF-01 | O aviso deve chegar ao cliente em menos de 10 segundos após a mudança de status, no caminho feliz | Desempenho | `[TRANSCRICAO 09:02 Marcos]`, `[TRANSCRICAO 09:10 Larissa]` |
| RNF-02 | A mudança de status de um pedido não pode ser bloqueada, atrasada ou revertida pela indisponibilidade ou lentidão de um cliente | Confiabilidade | `[TRANSCRICAO 09:04 Bruno]`, `[TRANSCRICAO 09:06 Diego]` |
| RNF-03 | Não pode existir mudança de status registrada sem que o aviso correspondente tenha sido registrado para envio | Confiabilidade | `[TRANSCRICAO 09:40 Bruno]`, `[TRANSCRICAO 09:41 Diego]` |
| RNF-04 | O endereço de destino cadastrado deve obrigatoriamente usar HTTPS; cadastro sem TLS é recusado | Segurança | `[TRANSCRICAO 09:23 Sofia]` |
| RNF-05 | Cada destino deve ter credencial de assinatura exclusiva — o vazamento da credencial de um cliente não pode comprometer os demais | Segurança | `[TRANSCRICAO 09:21 Sofia]` |
| RNF-06 | O conteúdo de um aviso é limitado a 64KB; acima disso o envio falha, e nunca é truncado | Limites | `[TRANSCRICAO 09:23 Sofia]`, `[TRANSCRICAO 09:24 Diego]`, `[TRANSCRICAO 09:24 Larissa]` |
| RNF-07 | Um cliente que não responder em até 10 segundos tem a tentativa tratada como falha | Confiabilidade | `[TRANSCRICAO 09:42 Diego]` |
| RNF-08 | A entrega é garantida ao menos uma vez: o mesmo aviso pode chegar mais de uma vez, e o cliente é responsável por descartar repetições pelo identificador do evento | Confiabilidade | `[TRANSCRICAO 09:24 Diego]`, `[TRANSCRICAO 09:25 Diego]` |
| RNF-09 | A ordem de chegada é garantida entre avisos de um mesmo pedido, não entre pedidos diferentes | Confiabilidade | `[TRANSCRICAO 09:12 Diego]`, `[TRANSCRICAO 09:13 Larissa]` |
| RNF-10 | O reprocessamento manual exige perfil de administrador e é registrado para auditoria | Segurança | `[TRANSCRICAO 09:36 Sofia]`, `[TRANSCRICAO 09:36 Larissa]` |
| RNF-11 | O gerenciamento de destinos exige autenticação, sem restrição adicional de perfil nesta fase | Segurança | `[TRANSCRICAO 09:36 Marcos]`, `[TRANSCRICAO 09:37 Sofia]` |

## 8. Decisões e trade-offs principais

### DEC-01 — A notificação é assíncrona, não instantânea

- **Decisão:** o aviso sai fora do fluxo da mudança de status, com latência típica de poucos segundos.
- **Alternativa considerada:** disparar a chamada ao cliente no mesmo instante da mudança de status.
- **Motivo da rejeição:** um cliente lento passaria a atrasar mudanças de status de outros pedidos, e um cliente fora do ar obrigaria a reverter uma mudança de status legítima.
- **O que aceitamos perder:** a notificação não é imediata. Existe uma latência de alguns segundos — aceita porque o cliente definiu "tempo real" como qualquer coisa abaixo de 10 segundos.
- **Origem:** `[TRANSCRICAO 09:04 Bruno]`, `[TRANSCRICAO 09:06 Diego]`, `[TRANSCRICAO 09:02 Marcos]` · [ADR-001](./adrs/ADR-001-outbox-no-mysql.md)

### DEC-02 — Desistimos do aviso após cinco tentativas em ~15 horas

- **Decisão:** o sistema reenvia até cinco vezes, com intervalos crescentes ao longo de quase 15 horas, e então para.
- **Alternativa considerada:** três tentativas em uma janela curta (mais agressiva) ou reenvio indefinido.
- **Motivo da rejeição:** três tentativas em meia hora matariam avisos de um cliente em manutenção planejada de duas horas — cenário que já ocorreu. Reenvio indefinido deixaria avisos pendurados para sempre quando o cliente simplesmente sumisse.
- **O que aceitamos perder:** um cliente indisponível por mais de 15 horas deixa de receber automaticamente e passa a depender de reprocessamento manual pela nossa operação. O PM aceitou explicitamente esse limite.
- **Origem:** `[TRANSCRICAO 09:16 Diego]`, `[TRANSCRICAO 09:17 Marcos]` · [ADR-003](./adrs/ADR-003-retry-com-backoff-e-dlq.md)

### DEC-03 — O cliente é responsável por descartar avisos repetidos

- **Decisão:** garantimos que o aviso chega ao menos uma vez, e cada aviso carrega um identificador estável para que o cliente reconheça repetições.
- **Alternativa considerada:** garantir que cada aviso chegue exatamente uma vez.
- **Motivo da rejeição:** exigiria coordenação entre os dois lados e complexidade desproporcional ao prazo; é também o padrão que Stripe e GitHub adotam.
- **O que aceitamos perder:** transferimos trabalho de integração para o cliente e criamos uma dependência de documentação — um cliente que ignorar a orientação processará o mesmo evento duas vezes.
- **Origem:** `[TRANSCRICAO 09:25 Diego]`, `[TRANSCRICAO 09:25 Sofia]`, `[TRANSCRICAO 09:26 Marcos]` · [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-event-id.md)

### DEC-04 — A troca de credencial mantém a anterior válida por 24 horas

- **Decisão:** ao pedir uma credencial nova, o cliente continua com a antiga funcionando por 24 horas.
- **Alternativa considerada:** invalidar a credencial anterior no ato da troca.
- **Motivo da rejeição:** o cliente precisa de tempo para atualizar os próprios sistemas sem quebrar a integração — o time já teve um cliente que vazou a credencial em log e precisou trocá-la.
- **O que aceitamos perder:** durante 24 horas duas credenciais são válidas, inclusive a que motivou a troca. A troca não é, portanto, o mecanismo de resposta imediata a um vazamento — para isso é preciso desativar o destino.
- **Origem:** `[TRANSCRICAO 09:21 Sofia]`, `[TRANSCRICAO 09:22 Diego]` · [ADR-004](./adrs/ADR-004-hmac-sha256-secret-por-endpoint.md)

### DEC-05 — O aviso descreve o pedido como ele era, não como está

- **Decisão:** o conteúdo do aviso é congelado no instante da mudança de status.
- **Alternativa considerada:** montar o conteúdo no momento do envio, com os dados mais recentes.
- **Motivo da rejeição:** um aviso entregue horas depois descreveria um estado que não corresponde à transição que ele anuncia.
- **O que aceitamos perder:** um aviso muito atrasado pode trazer dados desatualizados; o cliente que precisar do estado atual deve consultar o pedido.
- **Origem:** `[TRANSCRICAO 09:51 Bruno]`, `[TRANSCRICAO 09:52 Larissa]` · [ADR-006](./adrs/ADR-006-snapshot-do-payload-na-outbox.md)

## 9. Dependências

| ID | Dependência | Tipo | Impacto se ausente | Origem |
| --- | --- | --- | --- | --- |
| DEP-01 | Documentação da integração no portal do desenvolvedor, com destaque para a possibilidade de aviso repetido | Organizacional | Clientes integram errado; a repetição de avisos vira volume de suporte | `[TRANSCRICAO 09:26 Marcos]`, `[TRANSCRICAO 09:40 Marcos]` |
| DEP-02 | Revisão de segurança conduzida por Sofia, com no mínimo dois dias úteis reservados antes do deploy | Organizacional | Deploy bloqueado; a feature não sobe sem essa revisão | `[TRANSCRICAO 09:46 Sofia]`, `[TRANSCRICAO 09:49 Sofia]` |
| DEP-03 | Clientes implementarem a verificação da assinatura e o descarte de avisos repetidos do lado deles | Externa | O cliente não consegue confiar na origem do aviso, ou processa o mesmo evento duas vezes | `[TRANSCRICAO 09:20 Sofia]`, `[TRANSCRICAO 09:25 Diego]` |
| DEP-04 | Capacidade de execução do time por três sprints | Organizacional | Prazo de fim de novembro não é cumprido, com risco de churn (RSC-01) | `[TRANSCRICAO 09:46 Larissa]` |
| DEP-05 | Infraestrutura capaz de manter no ar um segundo processo além da API | Técnica | Sem o processo de entrega no ar, nenhum aviso é enviado | `[TRANSCRICAO 09:11 Diego]` · [RFC P-02](./RFC.md#5-questões-em-aberto) |

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação | Responsável | Origem |
| --- | --- | --- | --- | --- | --- | --- |
| RSC-01 | **Perda da Atlas Comercial por atraso.** A cliente condicionou a permanência à entrega até o fim do trimestre; a estimativa de três sprints inclui uma revisão de segurança bloqueante no fim | Média | Alto | Prazo confirmado com a cliente logo após a reunião; revisão de segurança agendada já no planejamento, não no fim | Marcos / Larissa | `[TRANSCRICAO 09:00 Marcos]`, `[TRANSCRICAO 09:45 Marcos]`, `[TRANSCRICAO 09:47 Marcos]` |
| RSC-02 | **Cliente processa o mesmo aviso duas vezes.** A garantia é de ao menos uma entrega, e o descarte de repetições depende do cliente | Média | Médio | Documentação destacada no portal do desenvolvedor (DEP-01) e identificador de evento estável em todo reenvio | Marcos | `[TRANSCRICAO 09:25 Sofia]`, `[TRANSCRICAO 09:26 Marcos]` |
| RSC-03 | **Avisos morrem sem ninguém perceber.** Esgotadas as tentativas, o aviso fica guardado aguardando reprocessamento manual — e não há alerta automático nem aviso ao cliente nesta fase | Alta | Médio | Acompanhamento interno da fila de falhas permanentes; o aviso por e-mail ao cliente é o primeiro candidato da próxima fase (FE-01) | Larissa | `[TRANSCRICAO 09:18 Diego]`, `[TRANSCRICAO 09:37 Larissa]` |
| RSC-04 | **Cliente fora do ar por mais de 15 horas perde avisos.** Após a última tentativa o sistema desiste automaticamente | Baixa | Médio | Reprocessamento manual pela operação (RF-13); limite aceito explicitamente pelo PM na reunião | Larissa | `[TRANSCRICAO 09:17 Marcos]` |
| RSC-05 | **Vazamento de credencial de cliente.** As credenciais passam a existir na plataforma como ativo sensível, e o time já teve um caso de cliente vazando a própria credencial em log | Média | Alto | Credencial exclusiva por destino limita o alcance do vazamento (RNF-05); troca autônoma pelo cliente (RF-08); revisão de segurança bloqueante (DEP-02) | Sofia | `[TRANSCRICAO 09:21 Sofia]`, `[TRANSCRICAO 09:22 Diego]` |
| RSC-06 | **Rajada de avisos a um mesmo cliente.** Cinquenta pedidos mudando de status em um minuto geram cinquenta chamadas seguidas, e não há limitação de taxa nesta fase (FE-03) | Média | Baixo | Observar em produção e implementar limitação se virar problema real, conforme decidido na reunião | Diego | `[TRANSCRICAO 09:38 Diego]`, `[TRANSCRICAO 09:39 Larissa]` |

## 11. Critérios de aceitação

| ID | Critério | Requisitos cobertos |
| --- | --- | --- |
| CA-01 | Ao mudar o status de um pedido cujo cliente tem destino ativo escutando aquele status, o aviso chega ao destino em menos de 10 segundos | RF-01, RNF-01 |
| CA-02 | O aviso **não** é enviado quando o status alterado não está entre os selecionados naquele destino | RF-06 |
| CA-03 | O cadastro de um destino retorna a credencial de assinatura na resposta da criação, e ela não é recuperável em consultas posteriores | RF-02, RNF-05 |
| CA-04 | Consultar, alterar e remover destinos funciona para os destinos do cliente, com as alterações refletidas na consulta seguinte | RF-03, RF-04, RF-05 |
| CA-05 | O histórico de entregas de um destino exibe, por tentativa, o resultado, o conteúdo enviado, a resposta recebida e o tempo de resposta | RF-07 |
| CA-06 | Após solicitar a troca de credencial, a nova é retornada e a anterior permanece registrada como válida por 24 horas, deixando de valer depois disso | RF-08, DEC-04 |
| CA-07 | A assinatura enviada em cada aviso confere quando recalculada pelo cliente com a credencial do destino, e não confere se o conteúdo for alterado | RF-09, RNF-05 |
| CA-08 | Um aviso reenviado por falha ou por reprocessamento manual chega com o mesmo identificador de evento do envio original | RF-10, RNF-08 |
| CA-09 | Um destino que responde com erro recebe até cinco tentativas, com intervalos crescentes; após a quinta, o aviso é preservado com o motivo da falha e deixa de ser reenviado automaticamente | RF-11, RF-12 |
| CA-10 | Um destino que não responde em 10 segundos tem a tentativa encerrada e contabilizada como falha | RNF-07 |
| CA-11 | O reprocessamento manual é recusado para usuários sem perfil de administrador e, quando executado, registra quem o fez | RF-13, RNF-10 |
| CA-12 | Cadastrar um destino com endereço sem HTTPS é recusado com erro de validação | RNF-04 |
| CA-13 | Um aviso entregue horas após a mudança de status apresenta os dados do pedido como estavam no instante da transição | RF-14, DEC-05 |
| CA-14 | Se o registro do aviso não puder ser criado, a mudança de status não é efetivada | RNF-03 |
| CA-15 | Uma mudança de status não é atrasada nem revertida quando o destino do cliente está indisponível ou lento | RNF-02 |
| CA-16 | O conteúdo do aviso traz os dados essenciais do pedido e da transição, sem a lista de itens | RF-15 |

## 12. Estratégia de testes e validação

| Nível | O que valida nesta feature |
| --- | --- |
| Unitário | Cálculo e verificação da assinatura; progressão dos intervalos entre tentativas; filtro de status por destino; validação do endereço HTTPS e do limite de 64KB |
| Integração | Atomicidade entre a mudança de status e o registro do aviso, inclusive o caso de rollback (CA-14); ciclo completo de tentativa, falha, reenvio e preservação após a quinta falha; troca de credencial e comportamento nas 24 horas de convivência |
| Ponta a ponta | Do `PATCH` de status até a chegada do aviso a um destino de teste, com verificação de assinatura e identificadores; reprocessamento manual por administrador e recusa para não administradores |
| Carga / resiliência | Comportamento com destino lento (perto do limite de 10s) e destino indisponível; acúmulo e drenagem da fila após uma parada prolongada do processo de entrega (RSC-06); ausência de impacto na latência de `PATCH` de status sob concorrência |
| Segurança | Revisão conduzida por Sofia, com no mínimo dois dias úteis, cobrindo a geração da credencial e a assinatura — **bloqueante para o deploy** `[TRANSCRICAO 09:46 Sofia]`; verificação de que credenciais não aparecem em log |

**Validação pós-deploy.** Acompanhar, nas duas primeiras semanas, o tempo entre a mudança de status e a entrega (OBJ-01), a taxa de sucesso na primeira tentativa e o volume de avisos que chegam a falhar em definitivo. Sucesso: os três clientes integrados e entrega abaixo de 10 segundos no caminho feliz. Sinal de reversão ou correção urgente: aumento na taxa de erro do `PATCH` de status de pedidos, que indicaria que o registro do aviso está afetando a operação principal (RNF-02, RNF-03).

## 13. Questões em aberto e premissas

| ID | Item | Tipo | O que resolveria | Origem |
| --- | --- | --- | --- | --- |
| Q-01 | Limitação da taxa de envio: sem critério definido para quando "vira problema" e passa a ser necessária | Questão em aberto | Medição em produção; decisão de Diego | `[TRANSCRICAO 09:38 Diego]`, `[TRANSCRICAO 09:39 Larissa]` |
| Q-02 | Endurecimento do controle de quem pode gerenciar destinos de notificação | Questão em aberto | Decisão de Sofia, sem data definida | `[TRANSCRICAO 09:37 Sofia]` |
| Q-03 | Expurgo dos registros de avisos já entregues: reconhecido como necessário, sem dono nem prazo | Questão em aberto | Atribuição de dono; hoje sem responsável | `[TRANSCRICAO 09:08 Diego]` |
| Q-04 | Quando (e se) a plataforma passará a suportar mais de um processo de entrega em paralelo, e o efeito disso sobre a ordem dos avisos | Questão em aberto | Decisão de Diego; tratado como "problema do futuro" | `[TRANSCRICAO 09:13 Diego]` |
| Q-05 | Baseline do volume de polling dos três clientes (OBJ-03) nunca foi medido, o que impede definir meta de redução | Questão em aberto | Instrumentar a contagem de requisições por cliente antes do deploy | `[TRANSCRICAO 09:00 Marcos]` |
| P-01 | ⚠️ A data da reunião não consta na fonte (apenas "quinta-feira, 09:00"); as datas deste pacote referem-se à redação dos documentos | ⚠️ Premissa | Confirmação pelo convite da call; sem impacto sobre requisitos | — |
| P-02 | ⚠️ O prazo "fim do trimestre" citado pela Atlas e o "fim de novembro" citado depois foram tratados como o mesmo marco | ⚠️ Premissa | Confirmação de Marcos com a cliente; se forem marcos distintos, OBJ-02 muda de data | `[TRANSCRICAO 09:00 Marcos]`, `[TRANSCRICAO 09:45 Marcos]` |
