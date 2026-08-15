# O que pertence a cada seção do PRD

Referência de redação. Cada seção traz o propósito, o que precisa estar lá e um par de exemplos
de conteúdo forte e fraco.

> Os exemplos usam uma feature fictícia — **exportação agendada de relatórios de vendas** — em
> um domínio deliberadamente diferente do que você provavelmente está documentando. Isso é
> proposital: os exemplos servem para mostrar a *forma* de um bom item de PRD, nunca para serem
> copiados como conteúdo. Todo conteúdo do seu PRD vem da sua fonte.

## Índice

1. [Resumo e contexto da feature](#1-resumo-e-contexto-da-feature)
2. [Problema e motivação](#2-problema-e-motivação)
3. [Público-alvo e cenários de uso](#3-público-alvo-e-cenários-de-uso)
4. [Objetivos e métricas de sucesso](#4-objetivos-e-métricas-de-sucesso)
5. [Escopo](#5-escopo)
6. [Requisitos funcionais](#6-requisitos-funcionais)
7. [Requisitos não funcionais](#7-requisitos-não-funcionais)
8. [Decisões e trade-offs principais](#8-decisões-e-trade-offs-principais)
9. [Dependências](#9-dependências)
10. [Riscos e mitigação](#10-riscos-e-mitigação)
11. [Critérios de aceitação](#11-critérios-de-aceitação)
12. [Estratégia de testes e validação](#12-estratégia-de-testes-e-validação)
13. [Questões em aberto e premissas](#13-questões-em-aberto-e-premissas)

---

## 1. Resumo e contexto da feature

**Propósito:** dar a alguém que nunca ouviu falar da feature o suficiente para decidir se
precisa continuar lendo.

Um a três parágrafos: o que é a feature, para quem, qual sistema ela toca e por que agora.
O "por que agora" é o que separa contexto de descrição — prazo comercial, pedido formal de
cliente, custo crescente, obrigação regulatória.

- ✅ "O time de vendas hoje reexecuta manualmente o relatório de fechamento todo primeiro dia
  útil do mês, e o pedido de automatização entrou como condição contratual de dois contratos
  renovados neste trimestre."
- ❌ "Esta feature moderniza a plataforma com um sistema de relatórios robusto e escalável."

## 2. Problema e motivação

**Propósito:** descrever a dor em termos de quem a sente, antes de qualquer solução.

Separe: qual é o problema, quem é afetado, qual o custo de não resolver. Se a fonte tiver
números — custo, volume, tempo perdido, clientes em risco — eles moram aqui.

Evite descrever a solução disfarçada de problema. "Falta um agendador de exportação" não é um
problema, é a ausência da solução escolhida. O problema é o que acontece com o usuário hoje.

- ✅ "Cada analista gasta cerca de duas horas por mês reexecutando e conferindo o mesmo
  relatório, e o atraso na conferência já atrasou o fechamento contábil duas vezes no semestre."
- ❌ "O sistema não possui exportação agendada."

## 3. Público-alvo e cenários de uso

**Propósito:** nomear quem usa e em que situação, para que os requisitos tenham dono.

Liste os perfis reais citados na fonte — cliente, operador interno, administrador, time de
suporte. Para cada um, um ou dois cenários concretos, em linguagem de negócio.

Persona não citada na fonte é invenção. Se a fonte só fala de "analistas de vendas", não crie a
"Maria, gerente de 34 anos que adora eficiência".

- ✅ "Analista de vendas: precisa do relatório de fechamento na caixa de entrada no primeiro dia
  útil, sem abrir o sistema."
- ❌ "Usuários em geral que desejam maior praticidade."

## 4. Objetivos e métricas de sucesso

**Propósito:** definir como saberemos, depois do deploy, se a feature funcionou.

Cada objetivo precisa de: métrica, valor atual (quando conhecido), meta quantitativa e forma de
medição. Pelo menos um objetivo tem que ter meta numérica — sem isso o PRD não é auditável.

Quando a fonte der um número explícito, esse número é a meta. Quando não der, registre a lacuna
na seção 13 em vez de inventar um percentual que soa razoável.

| ID | Objetivo | Métrica | Valor atual | Meta | Como medir | Origem |
| --- | --- | --- | --- | --- | --- | --- |
| OBJ-01 | Eliminar a execução manual do fechamento mensal | Horas de analista gastas por mês | ~2h por analista | 0h | Comparação antes/depois com os 6 analistas | `[TICKET SLS-233]` |

- ❌ "Melhorar a satisfação dos usuários." (sem métrica, sem meta, sem origem)

## 5. Escopo

**Propósito:** traçar a fronteira do que será construído nesta fase.

Duas listas. **Incluso** resume as capacidades entregues, sem repetir a lista completa de
requisitos funcionais. **Fora de escopo** é a seção mais valiosa do PRD e a mais negligenciada:
liste o que foi explicitamente descartado ou adiado, com o motivo e, quando existir, a fase
futura em que volta a ser discutido.

Um item fora de escopo é forte quando alguém realmente pediu e recebeu não como resposta.
Mínimo de dois itens com origem rastreável.

| ID | Item fora de escopo | Motivo | Situação | Origem |
| --- | --- | --- | --- | --- |
| FE-01 | Envio do relatório por e-mail | Depende do serviço de e-mail transacional, que está em migração | Adiado para a fase 2 | `[REUNIAO 14:20]` |
| FE-02 | Editor visual de colunas do relatório | Escopo do time de frontend, projeto separado | Descartado nesta fase | `[REUNIAO 14:31]` |

- ❌ "Fora de escopo: internacionalização, performance, migração de dados." (lista genérica que
  ninguém pediu — não é decisão, é enchimento)

## 6. Requisitos funcionais

**Propósito:** dizer o que o sistema precisa fazer, de forma testável e independente de
implementação.

Cada requisito: identificador, enunciado verificável, prioridade e origem. Um requisito por
linha — quando aparece "e" no meio, geralmente são dois requisitos.

Escreva em termos de capacidade observável. O detalhe do endpoint, do payload e do header
pertence ao FDD; o PRD diz que a capacidade existe.

| ID | Requisito | Prioridade | Origem |
| --- | --- | --- | --- |
| RF-01 | O analista deve poder agendar a geração recorrente de um relatório | Must | `[REUNIAO 14:05]` |
| RF-02 | O analista deve poder consultar o histórico das execuções de um agendamento, com resultado e horário | Must | `[REUNIAO 14:12]` |

- ❌ "O sistema deve ter uma tabela `report_schedules` com índice em `next_run_at`." (altura de
  FDD)

Prioridade: use MoSCoW (Must / Should / Could / Won't) ou a escala já adotada pelo time. Só
atribua prioridade que a fonte sustente; na dúvida, `Must` para o que foi fechado como
obrigatório e registre a dúvida como lacuna na seção 13.

## 7. Requisitos não funcionais

**Propósito:** as restrições de qualidade que valem para toda a feature.

Cubra as categorias que a fonte tocar — e apenas essas. As mais comuns:

- **Desempenho e latência:** janelas de tempo, timeouts, throughput esperado
- **Confiabilidade:** garantias de execução, comportamento sob falha, tolerância a indisponibilidade
- **Segurança:** autenticação, autorização, integridade, criptografia em trânsito, segredos
- **Limites e cotas:** tamanho máximo, número de tentativas, retenção de dados
- **Observabilidade:** o que precisa ser medido e auditado
- **Compatibilidade:** o que não pode quebrar no sistema existente

RNF também é testável. "Deve ser seguro" não é requisito; "o arquivo exportado só pode ser
baixado pelo usuário que criou o agendamento" é.

| ID | Requisito não funcional | Categoria | Origem |
| --- | --- | --- | --- |
| RNF-01 | Um relatório agendado não pode exceder 50MB; acima disso a execução falha em vez de truncar | Limites | `[REUNIAO 14:26]` |

## 8. Decisões e trade-offs principais

**Propósito:** registrar as escolhas que fecham portas e que alguém fora do time de engenharia
precisa conhecer, com o que se ganhou e o que se perdeu.

Esta é a seção mais confundida do PRD, especialmente quando a fonte é uma reunião técnica em que
quase tudo que foi decidido é arquitetural. O critério não é "a decisão é técnica ou de
negócio" — é **a quem a consequência pertence**:

> Registre a decisão pela consequência que ela impõe ao usuário, ao cliente ou ao negócio.
> Descreva a consequência, não o mecanismo. Se você não consegue enunciá-la sem nomear a
> tecnologia, ela é do RFC ou do ADR, não do PRD.

Aplicando o critério a decisões arquiteturais reais:

| Decisão tomada | Entra no PRD? | Como fica |
| --- | --- | --- |
| Reprocessar no máximo 5 vezes antes de desistir | ✅ Sim | O cliente indisponível por muito tempo deixa de receber automaticamente e depende de reprocessamento manual — isso é uma promessa de negócio |
| Gerar o arquivo de forma assíncrona em vez de na requisição | ✅ Sim | O usuário não recebe o arquivo na hora; recebe um aviso quando ficar pronto — muda a experiência |
| Usar `cron` do sistema em vez de biblioteca de agendamento | ❌ Não | Nenhuma consequência observável pelo usuário; é decisão de ADR |
| Guardar o agendamento em tabela nova em vez de reaproveitar a existente | ❌ Não | Invisível fora do código |

Formato: decisão → alternativa considerada → por que foi rejeitada → o que aceitamos perder.
O valor da seção está na alternativa rejeitada e no motivo da rejeição — sem isso, é só uma
lista de escolhas.

- ✅ "A geração do relatório é assíncrona: o usuário solicita e é avisado quando o arquivo fica
  pronto. Geração síncrona na requisição foi considerada e rejeitada porque relatórios do
  fechamento mensal levam minutos e derrubariam a requisição por timeout; aceitamos que o
  usuário não tenha o arquivo imediatamente na tela."
- ❌ "Optamos pela melhor arquitetura disponível, equilibrando performance e simplicidade."

Se a fonte tiver decisões arquiteturais que não passam nesse teste, não as jogue fora: liste-as
no seu relatório final ao usuário como material para o RFC ou para os ADRs.

## 9. Dependências

**Propósito:** listar o que precisa existir, estar disponível ou ser feito por outro time.

Inclua dependências técnicas (componente existente que será alterado), organizacionais (revisão
de segurança, aprovação), externas (fornecedor, cliente) e de dados. Para cada uma, diga o que
acontece se ela não vier.

| ID | Dependência | Tipo | Impacto se ausente | Origem |
| --- | --- | --- | --- | --- |
| DEP-01 | Revisão de segurança antes do deploy, com 2 dias úteis reservados | Organizacional | Bloqueia a entrega | `[REUNIAO 14:46]` |

## 10. Riscos e mitigação

**Propósito:** antecipar o que pode dar errado enquanto ainda dá para agir.

Cada risco: descrição concreta, probabilidade, impacto, mitigação acionável e, quando a fonte
permitir, responsável. Mínimo de dois riscos com as três dimensões preenchidas.

Risco bom é específico desta feature e tem gatilho identificável. "Atraso no cronograma" e
"complexidade técnica" servem para qualquer projeto e por isso não informam nada.

| ID | Risco | Probabilidade | Impacto | Mitigação | Origem |
| --- | --- | --- | --- | --- | --- |
| RSC-01 | Vários agendamentos no mesmo horário de pico degradam o banco de produção | Média | Alto | Distribuir a execução em janelas e limitar concorrência | `[REUNIAO 14:38]` |

## 11. Critérios de aceitação

**Propósito:** definir o que precisa ser verdade para a feature ser considerada pronta.

Cada critério é verificável por sim ou não. Em conjunto, os critérios precisam cobrir todo
requisito `Must` das seções 6 e 7 — referencie os identificadores correspondentes, porque é isso
que fecha o ciclo entre requisito e validação e permite auditar a cobertura.

Formato de lista de verificação ou Dado/Quando/Então, conforme a prática do time. Evite critério
que dependa de julgamento subjetivo ("a experiência deve ser fluida").

- ✅ "Dado um agendamento mensal ativo, quando chega o primeiro dia útil do mês, então o
  relatório é gerado e fica disponível para download. (RF-01, RNF-01)"
- ❌ "O sistema funciona corretamente."

## 12. Estratégia de testes e validação

**Propósito:** dizer como cada tipo de garantia será verificado, em nível de estratégia.

Cubra os níveis pertinentes — unitário, integração, ponta a ponta, carga, segurança — e o que
cada um valida **nesta** feature. Inclua também a validação pós-deploy: qual métrica será
observada, por quanto tempo, e o que caracteriza sucesso ou reversão.

Não é um plano de teste detalhado nem uma lista de casos; é a estratégia e a cobertura esperada.

- ✅ "Cenários de falha na geração (timeout da consulta, volume acima do limite, falha de
  escrita) devem ser cobertos por testes de integração, validando que o agendamento não é
  perdido e que o usuário é informado da falha."
- ❌ "Serão feitos testes unitários e de integração." (verdadeiro para qualquer feature)

## 13. Questões em aberto e premissas

**Propósito:** tornar visível o que ainda não foi decidido e o que foi assumido sem fonte.

Esta seção é obrigatória sempre que existir ao menos uma lacuna ou premissa — o que é quase
sempre. Ela é o contrapeso da regra de não inventar: em vez de preencher o vazio com um número
plausível, o vazio fica registrado com nome, dono e o que resolveria.

Separe os dois tipos:

- **Questão em aberto:** foi levantada na fonte e não foi decidida. Diga quem decide e quando.
- **⚠️ Premissa:** você precisou assumir algo para o documento fechar. Diga o que assumiu, por
  que, e o que acontece se a suposição estiver errada.

| Item | Tipo | O que resolveria | Origem |
| --- | --- | --- | --- |
| Limite de agendamentos simultâneos por usuário | Questão em aberto | Definição do time de plataforma após medir o uso real | `[REUNIAO 14:39]` |
| Assumido que o relatório usa o fuso do usuário, não o do servidor | ⚠️ Premissa | Confirmação com o PM; se for o do servidor, RF-01 muda | — |
