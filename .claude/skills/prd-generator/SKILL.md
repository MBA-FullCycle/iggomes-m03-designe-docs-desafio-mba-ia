---
name: prd-generator
description: Gera e refina PRDs (Product Requirements Document) rastreáveis a partir de fontes reais — transcrição de reunião, notas, ticket, thread de discussão ou código existente. Use sempre que o usuário pedir um PRD, "documento de requisitos de produto", "docs/PRD.md", quiser transformar uma reunião ou discussão em requisitos de produto, ou precisar estruturar problema, público-alvo, escopo, requisitos funcionais e não funcionais, métricas de sucesso, riscos e critérios de aceitação de uma feature — mesmo que não use a sigla "PRD". Use também para revisar, auditar, completar ou corrigir um PRD que já existe.
---

# Geração de PRD rastreável

## O que este documento é — e onde ele para

O PRD responde **por quê** e **o quê**, na altura de produto e negócio. Ele descreve o problema,
quem sofre com ele, o que a solução precisa fazer e como saberemos que deu certo.

O PRD **não** responde **como construir**. Esquema de tabela, nome de classe, biblioteca,
diagrama de sequência, contrato de endpoint detalhado — nada disso entra aqui. Esse conteúdo
pertence ao RFC (arquitetura) e ao FDD (implementação). Quando um documento invade a altura do
outro, o time passa a ter duas fontes da verdade que divergem na primeira mudança.

O teste prático: se um requisito só faz sentido para quem vai codar, ele desceu demais.
Se ele é verdade para qualquer implementação possível, está na altura certa.

Exemplo da fronteira:
- ✅ PRD: "O arquivo exportado deve ficar disponível em até 5 minutos após a solicitação."
- ❌ PRD: "Um job na fila `exports` grava o CSV em um bucket S3." (isso é RFC/FDD)

Cuidado com a leitura preguiçosa dessa regra: uma decisão técnica que produz consequência
visível para o usuário — um limite de tentativas, uma latência aceita, uma responsabilidade
transferida ao cliente — **pertence ao PRD**, descrita pela consequência e não pelo mecanismo.
A seção 8 do arquivo de referência traz o teste para separar os dois casos.

## A regra inegociável: nada sem origem

Cada afirmação do PRD nasce de uma fonte identificável. Um PRD com requisito inventado é pior
que um PRD incompleto: o time implementa algo que ninguém pediu, e a lacuna real continua
invisível. Por isso todo requisito, objetivo, métrica, risco e item de escopo carrega uma
referência de origem — timestamp da transcrição, caminho de arquivo, link do ticket.

Quando faltar informação para uma seção obrigatória, existem exatamente três saídas legítimas:

1. **Perguntar** ao usuário, se a resposta muda o conteúdo de forma relevante.
2. **Registrar como lacuna** na seção "Questões em aberto e premissas", nomeando o que falta.
3. **Marcar como premissa explícita**, com `⚠️ PREMISSA` e a justificativa de por que foi assumida.

O que nunca é aceitável: preencher a lacuna com um número plausível, uma persona genérica ou
uma métrica de mercado como se tivessem saído da fonte. "P95 < 200ms" e "aumentar o NPS em 20%"
são invenções quando ninguém disse isso.

## Fluxo

### 1. Reunir as fontes

Identifique as fontes de verdade antes de escrever qualquer coisa: transcrição, documento de
discovery, ticket, código existente. Se o usuário apontou um arquivo, esse arquivo é a fonte
primária.

Pergunte apenas o que muda o conteúdo do documento e você não consegue deduzir. Fonte, destino e
idioma já indicados pelo usuário não se perguntam de novo — na ausência de indicação, o destino
padrão é `docs/PRD.md` e o idioma é o das fontes.

O código existente é fonte **secundária**: serve para descrever o estado atual do sistema e as
restrições reais que ele impõe, nunca para derivar requisitos novos. No PRD, cite código apenas
quando ele sustenta uma afirmação de produto — o comportamento de hoje, o componente que será
afetado, um limite que já existe. Um punhado de citações costuma bastar; se `[CODIGO ...]`
aparecer em toda linha, o documento provavelmente desceu para a altura do FDD.

### 2. Ler a fonte inteira, do começo ao fim

Leia a fonte completa antes de escrever. Resumo de resumo é onde o requisito se perde e a
alucinação entra. Em transcrições, os trechos mais decisivos costumam estar em dois lugares
fáceis de pular: no fechamento (o resumo final que o condutor faz) e nas conversas de corredor
depois que parte dos participantes saiu.

### 3. Extrair e classificar as evidências

Percorra a fonte registrando cada item candidato com sua localização exata. Classifique cada um:

| Classe | O que é | Para onde vai no PRD |
| --- | --- | --- |
| `DECIDIDO` | Fechado explicitamente | Requisitos, escopo incluso, decisões |
| `ADIADO` | Reconhecido, jogado para outra fase | Fora de escopo (com a fase futura nomeada) |
| `DESCARTADO` | Rejeitado explicitamente | Fora de escopo (com o motivo da rejeição) |
| `EM ABERTO` | Levantado e não resolvido | Questões em aberto, riscos |
| `CONTEXTO` | Situação atual, dor, número de negócio | Problema, motivação, público, métricas |
| `IMPLEMENTAÇÃO` | Detalhe de como construir | **Não entra no PRD** — sinalize para o RFC/FDD |

Três armadilhas nessa etapa:

- **Proposta ≠ decisão.** Alguém sugerir "3 tentativas" e o grupo fechar em 5 significa que o
  requisito é 5. A proposta descartada não vira requisito, mas costuma ser excelente material
  para a seção de decisões e trade-offs, que ganha valor justamente ao mostrar o que foi
  rejeitado e por quê.
- **Adiado ≠ descartado.** "Fica pra próxima fase" e "não vamos fazer" produzem entradas
  diferentes em "Fora de escopo". A distinção importa para o roadmap.
- **Menção ≠ requisito.** Nem tudo que foi citado vira linha do PRD. Identificar o que fica de
  fora é tão importante quanto identificar o que entra.

### 4. Montar o inventário rastreado

Antes de redigir, consolide as evidências em uma tabela de trabalho — item, classe, origem,
citação curta. Ela é o rascunho que garante que cada linha do PRD tem lastro, e depois alimenta
diretamente um tracker de rastreabilidade, se o projeto tiver um.

Use um arquivo de rascunho fora do repositório do usuário para isso, não polua `docs/`.

### 5. Escrever o PRD

Use `assets/template-PRD.md` como esqueleto e `references/secoes-do-prd.md` para saber o que
pertence a cada seção, com exemplos de conteúdo forte e fraco. As doze seções obrigatórias são:

1. Resumo e contexto da feature
2. Problema e motivação
3. Público-alvo e cenários de uso
4. Objetivos e métricas de sucesso
5. Escopo (incluso e fora de escopo)
6. Requisitos funcionais
7. Requisitos não funcionais
8. Decisões e trade-offs principais
9. Dependências
10. Riscos e mitigação
11. Critérios de aceitação
12. Estratégia de testes e validação

A décima terceira seção do template, **Questões em aberto e premissas**, é obrigatória sempre que
houver ao menos uma lacuna ou premissa marcada — o que é quase sempre. Ela é o que dá saída
honesta à regra de não inventar. Outras seções adicionais (glossário, roadmap de fases futuras)
são bem-vindas quando há material real para elas.

Seção obrigatória sem material vira uma linha honesta — "nenhuma dependência externa foi
identificada na fonte" — e não texto de enchimento.

### 6. Auditar antes de entregar

Releia o documento pronto contra este piso de qualidade. Se algum item falhar, corrija antes
de responder ao usuário:

- [ ] Cada requisito funcional e não funcional tem identificador e origem preenchida
- [ ] Pelo menos um objetivo tem métrica e meta quantitativa vindas da fonte
- [ ] "Fora de escopo" lista no mínimo 2 itens explicitamente adiados ou descartados na fonte
- [ ] Riscos trazem probabilidade, impacto e mitigação — no mínimo 2
- [ ] Critérios de aceitação são verificáveis: cada um pode ser respondido com sim ou não
- [ ] Todo requisito de prioridade `Must` aparece em pelo menos um critério de aceitação
- [ ] Nenhuma seção obrigatória ficou ausente; a seção 13 existe se houver lacuna ou premissa
- [ ] Nenhum detalhe de implementação vazou para dentro do PRD
- [ ] Nenhuma linha sem origem passou sem marcação de premissa ou lacuna

Um PRD de feature real raramente tem menos de 8 requisitos funcionais. Se você extraiu 4 de uma
fonte densa, provavelmente varreu por cima — volte à fonte antes de entregar.

## Convenções de identificadores

Identificadores estáveis são o que permite ao RFC, ao FDD, aos ADRs, ao tracker e aos testes
apontarem para o mesmo requisito sem ambiguidade. Numere sequencialmente e não reaproveite
número de item removido.

| Prefixo | Uso |
| --- | --- |
| `OBJ-NN` | Objetivo de negócio |
| `RF-NN` | Requisito funcional |
| `RNF-NN` | Requisito não funcional |
| `FE-NN` | Item fora de escopo |
| `DEC-NN` | Decisão / trade-off |
| `DEP-NN` | Dependência |
| `RSC-NN` | Risco |
| `CA-NN` | Critério de aceitação |

Formato de origem: `[TRANSCRICAO 09:33]`, `[CODIGO src/modules/orders/order.service.ts]`,
`[TICKET OMS-412]`. Mantenha o formato uniforme dentro do documento.

## Antipadrões

| Antipadrão | Por que dói | O que fazer |
| --- | --- | --- |
| Requisito vago ("deve ser rápido", "boa usabilidade") | Não dá para testar nem para recusar uma entrega | Quantifique com o número que veio da fonte, ou marque como lacuna |
| Métrica sem baseline nem meta | Não mede sucesso, só descreve intenção | Métrica + valor atual + meta + como medir |
| Risco genérico ("atraso no cronograma") | Vale para qualquer projeto, logo não informa nada | Risco concreto desta feature, com gatilho e mitigação acionável |
| "Fora de escopo: performance, segurança" | Recusa de responsabilidade, não decisão | Nomeie o item que alguém pediu e foi negado, com o motivo |
| Copiar detalhes do RFC/FDD | Duplicação que diverge na primeira mudança | Deixe a decisão técnica no seu documento e referencie |
| Persona inventada | Requisito construído sobre alguém que não existe | Use os usuários reais citados na fonte |
| Aceitar a primeira versão sem reler a fonte | É onde a alucinação sobrevive | Auditoria da etapa 6, sempre |

## Ao entregar

Diga onde o arquivo foi escrito e reporte, em poucas linhas:

- quantos requisitos funcionais e não funcionais foram extraídos;
- quais premissas você precisou marcar e por quê;
- quais lacunas ficaram abertas e o que resolveria cada uma;
- qualquer conteúdo que você identificou na fonte mas deixou de fora do PRD por ser altura de
  RFC/FDD — isso poupa retrabalho no próximo documento.

Não afirme cobertura total da fonte se você não varreu a fonte inteira.

## Arquivos de apoio

- `references/secoes-do-prd.md` — o que pertence a cada uma das 12 seções, com exemplos de
  conteúdo forte e fraco. Leia antes de redigir.
- `assets/template-PRD.md` — esqueleto Markdown pronto, com as tabelas e colunas de origem.
