# O que pertence a cada seção do RFC

Referência de redação. Cada seção traz o propósito, o que precisa estar lá e um par de exemplos
de conteúdo forte e fraco.

> Os exemplos usam a mesma feature fictícia do `prd-generator` — **exportação agendada de
> relatórios de vendas** — para mostrar como o RFC continua de onde o PRD parou: a decisão de
> negócio já foi tomada (exportação assíncrona, agendada), e este documento propõe *como*
> construir isso em nível de arquitetura. Os exemplos servem para mostrar a *forma* de um bom
> item de RFC, nunca para serem copiados como conteúdo. Todo conteúdo do seu RFC vem da sua fonte.

## Índice

1. [Resumo executivo (TL;DR)](#1-resumo-executivo-tldr)
2. [Contexto e problema](#2-contexto-e-problema)
3. [Proposta técnica](#3-proposta-técnica)
4. [Alternativas consideradas](#4-alternativas-consideradas)
5. [Questões em aberto](#5-questões-em-aberto)
6. [Impacto e riscos](#6-impacto-e-riscos)
7. [Decisões relacionadas](#7-decisões-relacionadas)

---

## 1. Resumo executivo (TL;DR)

**Propósito:** dar a quem só vai ler três frases o suficiente para saber o que está sendo
proposto e decidir se precisa ler o resto.

Não é um teaser nem um resumo do problema — é um resumo da *proposta*. Alguém que leia só esta
seção deve conseguir responder "o que vão construir e por quê" sem abrir o resto do documento.

- ✅ "Propomos gerar os relatórios agendados de forma assíncrona, usando um worker separado que
  lê uma fila de agendamentos e grava o arquivo em storage de objetos. A geração síncrona foi
  descartada por estourar o timeout da requisição em relatórios grandes. O usuário passa a ser
  avisado por notificação quando o arquivo fica pronto, em vez de esperar na tela."
- ❌ "Este RFC descreve a arquitetura da feature de exportação de relatórios." (não diz o que foi
  proposto, só que existe uma proposta)

## 2. Contexto e problema

**Propósito:** explicar o motivo *técnico* desta proposta — não repetir o problema de negócio
inteiro, que já mora no PRD.

Se o PRD já registrou a dor do usuário e a decisão de negócio, o RFC referencia isso em uma
frase e vai direto ao que é específico da arquitetura: uma restrição do sistema atual, um volume
que o desenho existente não suporta, um requisito não funcional do PRD que obriga uma escolha
técnica.

- ✅ "O PRD define que o relatório deve ficar disponível em até 5 minutos após o horário agendado
  (RNF-01) [PRD seção 7]. O gerador de relatórios atual roda de forma síncrona dentro da própria
  requisição HTTP e tem timeout de 30 segundos [CODIGO src/modules/reports/report.service.ts],
  incompatível com relatórios de fechamento mensal que levam de 2 a 4 minutos para processar."
- ❌ "Os analistas perdem tempo gerando relatórios manualmente todo mês e isso atrapalha o
  fechamento contábil." (isso é o problema de negócio do PRD, não o problema técnico do RFC)

## 3. Proposta técnica

**Propósito:** descrever a solução em nível de arquitetura — os componentes, como eles se
conectam, que padrão resolve o problema — sem descer ao detalhe que só interessa a quem vai
codar.

O teste prático é o mesmo que separa PRD de RFC, aplicado um nível abaixo: se a frase só faz
sentido para alguém que vai escrever a migration ou o handler, ela desceu para o FDD. Se ela
descreve a forma da solução — quais peças existem e como conversam — está na altura certa.

- ✅ "A geração passa a ser assíncrona: a requisição do usuário apenas cria o agendamento; um
  worker dedicado, disparado por uma fila, processa a geração fora do ciclo de requisição e grava
  o arquivo em um bucket de objetos. O usuário é notificado quando o arquivo fica disponível."
- ❌ "Criamos uma tabela `report_jobs` com colunas `id BIGINT`, `status ENUM(...)`, índice
  composto em `(status, scheduled_at)`, e um endpoint `POST /reports/:id/generate` que enfileira
  via `BullMQ`." (isso é FDD — nome de tabela, tipo de coluna, biblioteca específica, rota exata)

Um jeito rápido de calibrar: a proposta técnica de um RFC bem escrito cabe, tipicamente, em um
ou dois parágrafos por componente. Se você está descrevendo um componente e sente necessidade de
listar cada campo dele, pare — isso é material para o FDD.

## 4. Alternativas consideradas

**Propósito:** mostrar que a decisão não foi óbvia, e por que a abordagem escolhida venceu as
outras que foram levadas a sério.

Esta é a seção onde a tentação de inventar é maior — e onde inventar é mais fácil de perceber.
Uma alternativa de palha ("poderíamos não fazer nada", "poderíamos usar uma tecnologia
aleatória que ninguém mencionou") não passa no teste de rastreabilidade: ela não tem um momento
na reunião em que alguém a defendeu. Só entram alternativas que tiveram um proponente real e
foram de fato discutidas antes de serem descartadas.

O valor da seção está inteiro no trade-off — "escolhemos X porque Y tinha esse problema
específico". Sem o trade-off nomeado, a seção vira uma lista de nomes de tecnologia sem
explicação.

- ✅ "**ALT-01 — Processamento síncrono com timeout estendido.** Aumentar o timeout da requisição
  para 5 minutos foi considerado por ser a mudança mais simples. Descartado porque prende uma
  conexão HTTP por minutos, esgotando o pool de conexões do serviço sob carga de fim de mês,
  quando múltiplos agendamentos disparam próximos do mesmo horário. [TRANSCRICAO 15:02]"
- ❌ "**ALT-01 — Outra abordagem.** Consideramos outras formas de fazer isso, mas a escolhida era
  melhor." (sem descrição da alternativa, sem trade-off, sem origem — não sobrevive a uma
  pergunta de "por quê")

Se a fonte só tiver uma alternativa real discutida, isso é uma lacuna genuína, não um problema
para resolver inventando uma segunda. Registre para o usuário que a fonte não sustenta o mínimo
de duas alternativas e pergunte se há outra ata ou se o critério fica com uma nota de exceção.

## 5. Questões em aberto

**Propósito:** registrar o que foi levantado sobre a proposta e ainda não foi fechado, para que
ninguém trate uma decisão pendente como se já tivesse sido tomada.

Diferente do PRD, aqui "questões em aberto" é seção obrigatória própria, não um apêndice. Cada
item precisa ter vindo de um momento real da reunião em que alguém levantou a dúvida e o grupo
não a fechou — inclui tanto pontos explicitamente adiados ("decidimos isso na próxima reunião")
quanto pontos que ficaram sem resposta.

- ✅ "**Q-01 — Retenção dos arquivos gerados.** Não ficou definido por quanto tempo os arquivos
  de relatório ficam disponíveis no bucket antes de expirar. Levantado pelo time de storage, sem
  decisão. [TRANSCRICAO 15:41]"
- ❌ "**Q-01 — Performance geral.** Precisamos garantir que tudo seja rápido." (não veio de um
  ponto real da discussão, é preenchimento genérico)

Premissas assumidas por você — não pela reunião — entram na subseção separada de premissas, cada
uma marcada com ⚠️ e o que a confirmaria. Não misture as duas: uma questão em aberto tem dono na
reunião; uma premissa tem você como autor.

## 6. Impacto e riscos

**Propósito:** nomear o que a mudança toca fora do próprio código novo, e o que pode dar errado
especificamente nesta proposta.

**Impacto** é sobre superfície: quais sistemas, times ou fluxos existentes são afetados pela
proposta — não é uma lista de tarefas, é uma lista de coisas que alguém de fora precisa saber
antes de a mudança acontecer.

**Riscos** segue o mesmo padrão do PRD: risco concreto, com probabilidade, impacto e mitigação
acionável — mas aqui o foco é risco de execução da arquitetura proposta, não risco de produto.

- ✅ "**Impacto.** O endpoint atual de geração síncrona (`POST /reports/:id/export`) muda de
  comportamento: passa a retornar 202 com um identificador de acompanhamento em vez do arquivo
  direto, o que quebra qualquer integração existente que espere o arquivo na resposta.
  [CODIGO src/modules/reports/report.controller.ts]"
- ❌ "**Impacto.** A feature vai melhorar o sistema." (não diz o que muda nem para quem)
- ✅ "**RSC-01 — Backlog da fila em pico de fim de mês.** Se todos os agendamentos mensais
  disparam no mesmo dia, o worker pode acumular fila e atrasar a entrega além dos 5 minutos
  prometidos. Probabilidade: Alta. Impacto: Médio. Mitigação: escalonar horários de disparo por
  cliente em vez de um horário fixo único. [TRANSCRICAO 15:55]"
- ❌ "**RSC-01 — Complexidade técnica.** O projeto tem certa complexidade." (genérico, vale para
  qualquer proposta, não indica gatilho nem mitigação)

## 7. Decisões relacionadas

**Propósito:** apontar para os ADRs que registram, em detalhe, cada decisão importante contida
nesta proposta — sem repetir o conteúdo deles aqui.

O RFC descreve a proposta como um todo; o ADR defende uma decisão pontual dela (por exemplo, "por
que fila em vez de cron", "por que bucket de objetos em vez de disco local") com alternativas e
consequências específicas daquela escolha. Um RFC sólido aponta para pelo menos dois ADRs.

Nunca crie o link antes de o arquivo do ADR existir — um link morto é pior do que a ausência da
seção, porque parece verificado e não é. Quando a decisão merece um ADR mas ele ainda não foi
escrito, liste a decisão como pendente e diga isso claramente no relatório final ao usuário.

- ✅ "| [ADR-002](../adrs/ADR-002-worker-assincrono-para-geracao.md) | Geração assíncrona via
  worker dedicado | Existente |"
- ❌ "| ADR-002 | Geração assíncrona | — |" (sem link — não dá para verificar se o arquivo existe
  nem o que ele diz)
