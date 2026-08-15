# O que pertence a cada seção do ADR

Referência de redação. Cada seção traz o propósito, o que precisa estar lá e um par de exemplos
de conteúdo forte e fraco.

> Os exemplos continuam a mesma feature fictícia da família de skills — **exportação/agendamento
> de relatórios de vendas** — desta vez aplicada a uma única decisão dentro dela: usar uma tabela
> própria no banco existente como fila de processamento, em vez de um serviço de fila gerenciado.
> Os exemplos servem para mostrar a *forma* de um bom item, nunca para serem copiados como
> conteúdo. Todo conteúdo do seu ADR vem da sua fonte.

## Índice

1. [Status](#1-status)
2. [Contexto](#2-contexto)
3. [Decisão](#3-decisão)
4. [Alternativas Consideradas](#4-alternativas-consideradas)
5. [Consequências](#5-consequências)
6. [Decisões relacionadas (recomendada)](#6-decisões-relacionadas-recomendada)

---

## 1. Status

**Propósito:** dizer, em uma linha, em que estágio de vida esta decisão está — sem obrigar quem lê
a inferir isso do resto do texto.

Use sempre um dos valores do ciclo de vida (`Proposta`, `Aceita`, `Rejeitada`, `Descontinuada`,
`Substituída por ADR-NNN`), nunca uma descrição livre. Inclua a data e, quando a fonte permitir,
quem participou da decisão — isso é o que permite a alguém saber se vale a pena revisitar a
decisão ou se ela já está consolidada.

- ✅ "**Aceita** — 2025-03-12. Decisores: Diego (Tech Lead), Larissa (Eng. Backend)."
- ❌ "Em andamento, ainda discutindo." (não é um valor do ciclo de vida; deixa ambíguo se a decisão
  já vale para quem for implementar)

## 2. Contexto

**Propósito:** dar só o necessário para entender por que *esta* decisão específica precisou ser
tomada — não o problema de negócio inteiro (PRD) nem a proposta de arquitetura completa (RFC).

Se o RFC já define a necessidade mais ampla, referencie-o em uma frase e vá direto à restrição ou
à pergunta que gerou esta decisão pontual.

- ✅ "Os relatórios agendados precisam ser processados de forma assíncrona [RFC-0004 seção 3].
  Antes de decidir como o job entra na fila de processamento, era preciso escolher entre usar a
  infraestrutura de banco já existente ou subir um serviço de fila gerenciado. O time opera hoje
  só MySQL em produção, sem serviço de fila no stack atual [CODIGO docker-compose.yml]."
- ❌ "Precisamos processar relatórios de forma assíncrona." (repete o RFC inteiro sem trazer o
  contexto específico desta decisão — não explica por que essa escolha, entre quais opções)

## 3. Decisão

**Propósito:** declarar, sem hedge, o que foi decidido — uma frase que qualquer pessoa da equipe
consegue repetir sem ambiguidade.

Evite listar as opções aqui; a decisão é a escolha, não o processo. As opções que perderam vão na
seção seguinte.

- ✅ "Usar uma tabela `report_jobs` no MySQL existente como fila de processamento, lida por um
  worker dedicado via polling a cada 5 segundos."
- ❌ "Vamos usar alguma forma de fila, possivelmente baseada no banco." (não é uma decisão, é uma
  direção vaga — não dá para saber se foi de fato resolvido)

## 4. Alternativas Consideradas

**Propósito:** mostrar que a decisão não foi a única opção óbvia, e nomear o trade-off específico
que a descartou.

Diferente do RFC (que exige 2 alternativas *realmente debatidas*), o ADR aceita 1 alternativa real
discutida **ou** plausível. Priorize sempre a que foi de fato levantada na fonte. Só recorra a uma
alternativa plausível-mas-não-discutida quando a fonte não registrou debate algum para esta
decisão — e, quando isso acontecer, marque com ⚠️ para deixar claro que não veio da reunião.

- ✅ (discutida na fonte) "**Fila gerenciada (ex.: SQS/RabbitMQ).** Considerada por oferecer
  entrega garantida e escalabilidade nativa fora do banco principal. Descartada porque introduz um
  serviço de infraestrutura novo para um time de 3 pessoas sem operação de fila hoje — custo
  operacional desproporcional ao volume atual (dezenas de relatórios por dia). [TRANSCRICAO 15:07]"
- ✅ (plausível, marcada) "**Redis com Streams.** ⚠️ Não discutida na fonte — incluída por ser uma
  alternativa real de mercado para filas leves. Traria o mesmo custo de operar um serviço adicional
  que a fila gerenciada, sem o ganho de garantias de entrega mais fortes que ela ofereceria."
- ❌ "Poderíamos ter feito de outro jeito, mas essa foi a opção que fez mais sentido." (não nomeia
  a alternativa nem o trade-off — não sobrevive a um "por quê" e não indica se veio da fonte)

## 5. Consequências

**Propósito:** registrar o custo real da decisão junto do benefício — é o que torna o ADR uma
defesa honesta, não uma justificativa retroativa.

Sempre inclua pelo menos um item positivo e um negativo. O item negativo não precisa ser grave;
precisa ser real e específico o suficiente para que alguém saiba o que monitorar ou aceitar.

- ✅ "**Positivas:** nenhuma infraestrutura nova para operar; a mesma transação que grava o
  agendamento grava o job, eliminando qualquer inconsistência entre os dois.
  **Negativas:** o MySQL não tem mecanismo nativo de notificação (diferente do LISTEN/NOTIFY do
  Postgres), então o worker depende de polling — o que impõe uma latência mínima de alguns
  segundos em vez de entrega quase instantânea."
- ❌ "Isso vai deixar o sistema mais robusto." (só benefício, vago, não diz o custo — não defende a
  decisão, só a elogia)

## 6. Decisões relacionadas (recomendada)

**Propósito:** apontar para o RFC que motivou esta decisão, para ADRs que ela supersede ou que a
supersedem, e para a seção do FDD que detalha a implementação — sem repetir o conteúdo de nenhum
deles aqui.

Esta seção não está na lista mínima da fonte, mas evita duas armadilhas comuns: o ADR repetir o
"Contexto" inteiro do RFC, e um ADR substituído continuar parecendo válido para quem não sabe que
existe um mais novo.

- ✅ "Implementa a proposta de processamento assíncrono do [RFC-0004, seção 3](../RFC.md#3-proposta-técnica).
  Detalhamento de implementação em [FDD, seção 4](../FDD.md#4-fluxos-detalhados). Não substitui
  nenhum ADR anterior."
- ❌ (omitir a seção quando existe um RFC ou um ADR anterior relevante, deixando o leitor sem saber
  que aquele contexto mais amplo existe)
