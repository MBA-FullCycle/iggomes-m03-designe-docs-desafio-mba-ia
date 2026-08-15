---
name: rfc-generator
description: Gera e refina RFCs (Request for Comments) técnicos rastreáveis a partir de fontes reais — transcrição de reunião, PRD existente, notas de arquitetura, thread de discussão ou código existente. Use sempre que o usuário pedir um RFC, uma "proposta técnica", "docs/RFC.md", quiser transformar uma decisão de arquitetura discutida em reunião em um documento formal de proposta, ou precisar estruturar contexto técnico, proposta de solução, alternativas consideradas e descartadas, questões em aberto, impacto e riscos, e links para ADRs — mesmo que não use a sigla "RFC". Use também para revisar, auditar, resumir ou corrigir um RFC que já existe, inclusive para checar se ele está mantendo o tamanho conciso esperado (2 a 4 páginas) sem duplicar o detalhamento do FDD.
---

# Geração de RFC rastreável

## O que este documento é — e onde ele para

O RFC responde **o que propomos e por quê**, na altura de arquitetura. Ele descreve a solução
técnica escolhida, as alternativas reais que perderam, e as perguntas que a equipe ainda precisa
responder antes de seguir.

O RFC vive entre dois outros documentos e não pode invadir nenhum dos dois:

- **Acima, o PRD** responde por que a feature existe e o que ela precisa fazer, em termos de
  negócio e usuário — métrica, persona, requisito funcional. O RFC não repete essa camada; ele a
  referencia e assume que ela já foi decidida.
- **Abaixo, o FDD** responde como construir em detalhe — schema exato de tabela, nome de classe,
  payload de endpoint, diagrama de sequência. O RFC descreve a forma da solução (quais
  componentes existem, como se conectam, que padrão resolve o problema), não a receita de
  implementação.

O teste prático: se a frase só faz sentido para quem vai escrever a migration ou o handler, ela
desceu demais — isso é FDD. Se ela só faz sentido junto de uma métrica de negócio ou uma persona,
ela subiu demais — isso é PRD. Se ela descreve uma escolha de arquitetura que qualquer
implementador precisaria conhecer antes de começar a codar, está na altura certa.

Nomear um arquivo, rota ou função real (`order.service.ts::changeStatus`, `requireRole`) não é,
por si só, descer para o FDD — muitas vezes é exatamente essa citação que dá lastro à proposta ou
ao risco. O que caracteriza FDD é a *profundidade* do detalhe: schema completo de tabela, payload
campo a campo, assinatura completa de função nova. Citar um identificador existente para justificar
uma decisão fica na altura do RFC; especificar um identificador novo em detalhe é FDD.

Diferente do PRD e do FDD, o RFC também carrega um orçamento de página: **2 a 4 páginas**. Isso
não é estético — é o que garante que a equipe de revisão realmente lê o documento inteiro antes
de aprovar. Se uma seção está crescendo além disso, o conteúdo provavelmente pertence ao FDD
(detalhe de implementação) ou a um ADR (justificativa extensa de uma decisão pontual), não ao
corpo do RFC.

## A regra inegociável: nada sem origem

Cada afirmação do RFC nasce de uma fonte identificável — e isso vale com força dobrada para a
seção de alternativas, porque é a mais fácil de preencher com invenção plausível. Um RFC com uma
alternativa de palha (proposta ninguém defendeu, só para o time "vencer" o debate) é pior que um
RFC com uma alternativa a menos: ele finge um debate que não aconteceu.

Quando faltar informação para uma seção obrigatória, existem exatamente três saídas legítimas:

1. **Perguntar** ao usuário, se a resposta muda o conteúdo de forma relevante.
2. **Registrar como lacuna**, na seção "Questões em aberto" (se foi levantado na reunião e não
   fechado) ou apontando explicitamente para o usuário que a fonte não sustenta o mínimo exigido
   (por exemplo, menos de 2 alternativas reais discutidas).
3. **Marcar como premissa explícita**, com `⚠️ PREMISSA` e a justificativa de por que foi assumida.

O que nunca é aceitável: inventar uma segunda alternativa para bater o mínimo de duas, inventar
um trade-off que soa técnico mas não foi dito, ou criar um link de ADR para um arquivo que ainda
não existe.

## Fluxo

### 1. Reunir as fontes

Identifique as fontes de verdade antes de escrever qualquer coisa: transcrição da reunião, PRD
relacionado (quando existir), notas de arquitetura, ADRs já escritos, código existente. Se o
usuário apontou um arquivo, esse arquivo é a fonte primária.

Pergunte apenas o que muda o conteúdo do documento e você não consegue deduzir. Fonte, destino e
idioma já indicados pelo usuário não se perguntam de novo — na ausência de indicação, o destino
padrão é `docs/RFC.md` e o idioma é o das fontes.

O PRD, quando existir, é fonte **secundária** de contexto: use-o para saber o que já foi
decidido em nível de negócio e evitar repeti-lo, não para derivar novas alternativas técnicas. O
código existente também é secundário: descreve as restrições reais do sistema atual (o timeout
que existe hoje, o componente que será alterado), nunca uma fonte de decisões novas.

Se o PRD ainda não foi escrito (é comum ele estar em rascunho vazio, esperando sua vez), não force
uma citação `[PRD seção]` que não existe — trate a transcrição como a fonte direta do contexto de
negócio necessário e cite-a normalmente.

### 2. Ler a fonte inteira, do começo ao fim

Leia a fonte completa antes de escrever. Em transcrições técnicas, as alternativas descartadas
costumam aparecer no meio da discussão — quando alguém propõe algo, é contestado e o grupo segue
em frente — e não no resumo final, que geralmente só registra a decisão vencedora. Pular para o
fechamento é a forma mais comum de perder o material da seção 4.

### 3. Extrair e classificar as evidências

Percorra a fonte registrando cada item candidato com sua localização exata. Classifique cada um:

| Classe | O que é | Para onde vai no RFC |
| --- | --- | --- |
| `PROPOSTA` | A abordagem que foi de fato escolhida | Proposta técnica |
| `ALTERNATIVA` | Discutida de verdade e descartada, com trade-off nomeado | Alternativas consideradas |
| `ABERTA` | Levantada na fonte e sem decisão fechada — inclui tanto o que ficou sem resposta quanto o que foi adiado sem um dono nomeado | Questões em aberto |
| `RISCO` | Ameaça concreta identificada à proposta | Impacto e riscos |
| `CONTEXTO` | Restrição técnica atual, motivo da mudança | Contexto e problema |
| `PRODUTO` | Requisito de negócio, métrica, persona, escopo | **Não entra no RFC** — já é altura de PRD |
| `IMPLEMENTAÇÃO` | Schema exato, nome de classe, payload, rota específica | **Não entra no RFC** — sinalize para o FDD |

Três armadilhas nessa etapa:

- **Alternativa real ≠ alternativa hipotética.** Só vira `ALTERNATIVA` o que teve um proponente
  identificável na fonte e foi de fato posto na mesa. Uma alternativa que você imaginaria que
  "provavelmente foi discutida" não entra.
- **Trade-off ≠ motivo vago.** "Era mais simples" ou "a equipe preferiu a outra" não é trade-off.
  O trade-off nomeia a consequência técnica específica que pesou contra a alternativa.
- **Decisão técnica ≠ decisão de produto.** Nem toda decisão da reunião que "parece importante"
  vira material de RFC. Se a consequência só é visível em uma métrica de negócio ou persona, é
  `PRODUTO` e pertence ao PRD, não à proposta técnica.
- **Fonte rica não obriga incluir tudo.** Reuniões técnicas longas às vezes sustentam 5, 6, 8
  alternativas reais. Incluir todas estoura o orçamento de página. Priorize as que tiveram o
  debate mais decisivo ou o maior impacto na arquitetura escolhida; o mínimo de 2 é piso, não meta.

### 4. Montar o inventário rastreado

Antes de redigir, consolide as evidências em uma tabela de trabalho — item, classe, origem,
citação curta. Ela é o rascunho que garante que cada linha do RFC tem lastro, e depois alimenta
diretamente um tracker de rastreabilidade, se o projeto tiver um.

Use um arquivo de rascunho fora do repositório do usuário para isso, não polua `docs/`.

### 5. Escrever o RFC

Use `assets/template-RFC.md` como esqueleto e `references/secoes-do-rfc.md` para saber o que
pertence a cada seção, com exemplos de conteúdo forte e fraco. As sete seções obrigatórias, além
dos metadados de cabeçalho, são:

1. Resumo executivo (TL;DR)
2. Contexto e problema
3. Proposta técnica
4. Alternativas consideradas
5. Questões em aberto
6. Impacto e riscos
7. Decisões relacionadas

Preencha os **Revisores** do cabeçalho com os participantes da reunião fonte — é a exigência
específica deste documento, diferente do PRD, cujos autores não precisam ser os revisores.

Seção obrigatória sem material suficiente vira uma linha honesta que nomeia a lacuna — nunca
texto de enchimento. Se a fonte sustenta só uma alternativa real, diga isso ao usuário em vez de
inventar a segunda.

### 6. Auditar antes de entregar

Releia o documento pronto contra este piso de qualidade. Se algum item falhar, corrija antes
de responder ao usuário:

- [ ] Arquivo existe em Markdown, salvo em `docs/RFC.md` (ou destino indicado pelo usuário)
- [ ] Contém as sete seções obrigatórias mais o cabeçalho de metadados
- [ ] "Alternativas consideradas" lista no mínimo 2 alternativas reais discutidas e descartadas
      na fonte, cada uma com o trade-off específico que motivou o descarte
- [ ] "Questões em aberto" lista no mínimo 2 pontos levantados na fonte e não decididos ou
      adiados, cada um com origem
- [ ] "Decisões relacionadas" referencia, com link funcional, no mínimo 2 ADRs — ou registra
      explicitamente quais decisões ainda precisam virar ADR
- [ ] Revisores do cabeçalho são os participantes reais da reunião fonte
- [ ] O documento cabe no orçamento de 2 a 4 páginas — nenhuma seção descreu para o nível de
      detalhe do FDD (schema, classe, payload) nem subiu para métrica/persona do PRD
- [ ] Nenhuma alternativa, trade-off ou link de ADR foi inventado; toda linha sem origem direta
      está marcada como ⚠️ PREMISSA ou registrada como lacuna

Um RFC derivado de uma reunião técnica real raramente tem só uma alternativa discutida ou só uma
questão em aberto. Se você extraiu o mínimo exato de cada seção, releia a fonte antes de
entregar — é provável que tenha varreu por cima.

## Convenções de identificadores

Identificadores estáveis são o que permite ao PRD, ao FDD, aos ADRs, ao tracker e aos testes
apontarem para o mesmo item sem ambiguidade. Numere sequencialmente e não reaproveite número de
item removido.

| Prefixo | Uso |
| --- | --- |
| `ALT-NN` | Alternativa considerada e descartada |
| `Q-NN` | Questão em aberto |
| `P-NN` | Premissa assumida (⚠️) |
| `RSC-NN` | Risco |

Formato de origem: `[TRANSCRICAO hh:mm]`, `[CODIGO caminho/do/arquivo]`, `[PRD seção]`,
`[ADR-NNN]`. Mantenha o formato uniforme dentro do documento.

## Antipadrões

| Antipadrão | Por que dói | O que fazer |
| --- | --- | --- |
| Alternativa de palha inventada para o time "vencer" o debate | Não é rastreável, é teatro de decisão | Use só alternativas que alguém defendeu de verdade na fonte |
| "Consideramos X mas escolhemos Y" sem dizer por quê | Registra a decisão sem explicá-la | Nomeie o trade-off técnico específico que decidiu |
| RFC de 10 páginas com schema completo e diagrama de classes | É altura de FDD; duplica e diverge na primeira mudança | Mova o detalhe para o FDD e referencie a partir do RFC |
| "Questões em aberto: performance, segurança" genéricas | Não veio da reunião, é enchimento | Só questões que alguém de fato levantou e não fechou |
| Link de ADR para arquivo que ainda não existe | Parece verificado e não é — quebra a rastreabilidade do tracker | Liste como decisão pendente de virar ADR até o arquivo existir |
| Repetir a lista de requisitos funcionais ou métricas do PRD | Duplicação que diverge na primeira mudança | Referencie o PRD por seção, não copie o conteúdo |
| Risco genérico ("complexidade técnica", "pode dar errado") | Vale para qualquer proposta, não informa nada | Risco concreto desta proposta, com gatilho e mitigação acionável |

## Ao entregar

Diga onde o arquivo foi escrito e reporte, em poucas linhas:

- quantas alternativas foram extraídas e, se ficou abaixo de 2, por que a fonte não sustentava
  mais;
- quantos ADRs foram linkados e quantas decisões ficaram pendentes de virar ADR;
- quais premissas você precisou marcar e por quê;
- quais questões em aberto ficaram registradas;
- qualquer conteúdo identificado na fonte mas deixado de fora do RFC por ser altura de PRD
  (métrica, persona) ou de FDD (schema, endpoint, classe) — isso poupa retrabalho no próximo
  documento.

Não afirme cobertura total da fonte se você não a varreu inteira.

## Arquivos de apoio

- `references/secoes-do-rfc.md` — o que pertence a cada uma das 7 seções, com exemplos de
  conteúdo forte e fraco. Leia antes de redigir.
- `assets/template-RFC.md` — esqueleto Markdown pronto, com as tabelas e colunas de origem.
