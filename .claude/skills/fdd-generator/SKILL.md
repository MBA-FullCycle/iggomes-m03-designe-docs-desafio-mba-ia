---
name: fdd-generator
description: Gera e refina FDDs (Feature Design Document / documento de design de implementação) rastreáveis e acionáveis a partir de fontes reais — RFC aprovado, ADRs, transcrição de reunião e código existente. Use sempre que o usuário pedir um FDD, "documento de design de implementação", "docs/FDD.md", quiser detalhar fluxos, contratos de API, matriz de erros, resiliência ou observabilidade de uma feature já decidida em RFC, ou precisar traduzir uma decisão de arquitetura em algo que um desenvolvedor consiga pegar e codar imediatamente — mesmo que não use a sigla "FDD". Use também para revisar, auditar, completar ou corrigir um FDD que já existe, inclusive para checar se as integrações citadas apontam para arquivos reais do código-base.
---

# Geração de FDD rastreável

## O que este documento é — e onde ele para

O FDD responde **como construir**, na altura de implementação. É o documento mais técnico da
cadeia — precisa estar acionável o suficiente para um desenvolvedor abrir o editor e começar a
codar sem precisar perguntar mais nada.

O FDD vive abaixo de dois outros documentos e não pode repetir o trabalho de nenhum dos dois:

- **O RFC** já decidiu a arquitetura — quais componentes existem e como se conectam. O FDD não
  reabre esse debate; ele parte da proposta aprovada e a detalha até o nível de schema exato,
  payload completo, nome de método. Se o RFC disse "um worker dedicado processa a fila", o FDD diz
  exatamente como o worker lê a fila, o que acontece em cada tentativa e o que grava no banco.
- **Os ADRs** já defenderam cada decisão pontual com alternativas e consequências. O FDD não
  reargumenta por que uma escolha venceu outra; ele assume a decisão como fechada e escreve a
  receita de implementação dela.

O teste prático: se uma frase explica *por que* uma arquitetura vence outra, ela subiu demais —
isso é RFC ou ADR. Se ela só faz sentido para quem já aceitou a arquitetura e precisa saber o que
o endpoint retorna num payload inválido, ou quantas tentativas um retry faz antes de cair pra DLQ,
está na altura certa.

Diferente do `prd-generator` e do `rfc-generator`, aqui o **código existente deixa de ser fonte
secundária e vira fonte primária**: a seção de integração exige nomear arquivos reais do
código-base e descrever como cada um muda. Um FDD que não abre o código para verificar nome de
classe, assinatura de método e caminho de arquivo está adivinhando, não documentando.

## A regra inegociável: nada sem origem

Cada afirmação do FDD nasce de uma fonte identificável — RFC aprovado, ADR, código real ou
transcrição. Isso vale com força redobrada para dois pontos que são fáceis de preencher com
invenção plausível:

- a **matriz de erros**, porque um código de erro genérico soa plausível mesmo sem ter sido
  discutido;
- a **integração com o sistema existente**, porque citar um arquivo ou método que não existe
  parece verificado e não é.

Quando faltar informação para uma seção obrigatória, existem exatamente três saídas legítimas:

1. **Perguntar** ao usuário, se a resposta muda o conteúdo de forma relevante — por exemplo, um
   número de timeout que ninguém definiu ainda.
2. **Registrar como lacuna**, na seção "Questões em aberto e premissas", nomeando exatamente o
   que falta decidir.
3. **Marcar como premissa explícita**, com `⚠️ PREMISSA` e a justificativa técnica de por que foi
   assumida (por exemplo, um valor de timeout que segue o padrão já usado em outro endpoint do
   sistema).

O que nunca é aceitável: inventar um código de erro no padrão certo mas que ninguém definiu,
citar um arquivo ou método que você não confirmou existir, ou preencher um número de retry/timeout
"que parece razoável" sem origem e sem marcá-lo como premissa.

## Fluxo

### 1. Reunir as fontes

Para o FDD, a ordem de precedência das fontes é diferente da do PRD e do RFC:

1. **RFC aprovado** — é a fonte da arquitetura que este documento vai detalhar. Se `docs/RFC.md`
   existir e não for um rascunho vazio, comece por ele.
2. **ADRs** — cada decisão pontual já defendida vira uma restrição de implementação aqui.
3. **Código existente** — descreve o estado real do sistema com que a feature precisa se
   integrar: classes, métodos, padrões de erro, middlewares. Nesta seção, ao contrário do PRD e
   do RFC, o código deixa de ser coadjuvante e vira insumo obrigatório da seção de integração.
4. **Transcrição da reunião** — cobre detalhes de implementação que foram discutidos ao vivo mas
   não couberam no RFC (que tem orçamento de página curto): um número de retry mencionado, uma
   decisão de nome de header, um limite de tamanho de payload.

Se o RFC ainda não foi escrito ou está em rascunho vazio, não force uma citação `[RFC seção]` que
não existe — trate a transcrição como fonte direta da arquitetura também, do mesmo jeito que o
`rfc-generator` faz quando o PRD ainda não existe, e sinalize isso ao usuário no relatório final.

Pergunte apenas o que muda o conteúdo do documento e você não consegue deduzir. Fonte, destino e
idioma já indicados pelo usuário não se perguntam de novo — na ausência de indicação, o destino
padrão é `docs/FDD.md` e o idioma é o das fontes.

### 2. Ler as fontes inteiras, do começo ao fim

Leia o RFC, os ADRs disponíveis e a transcrição por completo antes de escrever. Números concretos
— quantidade de tentativas de retry, valor de timeout, tamanho máximo de payload, formato de um
header — costumam aparecer como uma frase solta no meio da discussão técnica da reunião, não no
resumo final nem no RFC (que resume em vez de especificar). É ali que o FDD ganha o detalhe que o
RFC não carrega.

### 3. Explorar o código antes de escrever a seção de integração

Antes de nomear qualquer arquivo na seção "Integração com o sistema existente", confirme que ele
existe e que o método ou classe citado é real — use busca no código-base, não confie na memória do
que "provavelmente" está lá. Um caminho ou assinatura errados quebram a rastreabilidade do
documento tão quanto um requisito inventado.

### 4. Extrair e classificar as evidências

Percorra as fontes registrando cada item candidato com sua localização exata. Classifique cada um:

| Classe | O que é | Para onde vai no FDD |
| --- | --- | --- |
| `DECISÃO` | Arquitetura já fechada no RFC ou em um ADR | Base dos fluxos e contratos — não é redebatida |
| `FLUXO` | Passo a passo de um mecanismo (criação, processamento, retry, fallback) | Fluxos detalhados |
| `CONTRATO` | Endpoint, payload, header, código de status | Contratos públicos |
| `ERRO` | Cenário de falha com código, causa e status HTTP | Matriz de erros previstos |
| `RESILIÊNCIA` | Timeout, número de tentativas, fórmula de backoff, fallback | Estratégias de resiliência |
| `OBSERVABILIDADE` | Métrica, campo de log, span de tracing nomeados | Observabilidade |
| `DEPENDÊNCIA` | Biblioteca, serviço externo, versão, restrição de compatibilidade | Dependências e compatibilidade |
| `INTEGRAÇÃO` | Ponto real do código que a feature estende ou reutiliza | Integração com o sistema existente |
| `RISCO-IMPL` | Risco concreto de implementação (corrida, migração, edge case) | Riscos e mitigação |
| `DEBATE-ARQUITETURA` | Alternativa e trade-off de design | **Não entra no FDD** — já é altura de RFC/ADR |
| `PRODUTO` | Requisito de negócio, métrica, persona | **Não entra no FDD** — já é altura de PRD |

Duas armadilhas específicas desta etapa:

- **Decisão ≠ debate.** O FDD parte da decisão já tomada. Se você se pegar escrevendo "consideramos
  X mas escolhemos Y", isso pertence ao RFC ou a um ADR — no FDD, só existe Y, detalhado.
- **Detalhe plausível ≠ detalhe real.** Um timeout de "30 segundos" ou um retry de "3 tentativas"
  só entra se veio da fonte. Se você precisa de um número e a fonte não o deu, ele é premissa
  (⚠️) ou pergunta ao usuário — nunca preenchimento silencioso.

### 5. Montar o inventário rastreado

Antes de redigir, consolide as evidências em uma tabela de trabalho — item, classe, origem,
citação curta — inclusive os caminhos de arquivo já confirmados no passo 3. Essa tabela alimenta
diretamente o `docs/TRACKER.md`, se o projeto tiver um.

Use um arquivo de rascunho fora do repositório do usuário para isso, não polua `docs/`.

### 6. Escrever o FDD

Use `assets/template-FDD.md` como esqueleto e `references/secoes-do-fdd.md` para saber o que
pertence a cada seção, com exemplos de conteúdo forte e fraco. As doze seções obrigatórias são:

1. Contexto e motivação técnica
2. Objetivos técnicos
3. Escopo e exclusões
4. Fluxos detalhados
5. Contratos públicos
6. Matriz de erros previstos
7. Estratégias de resiliência
8. Observabilidade
9. Dependências e compatibilidade
10. Critérios de aceite técnicos
11. Riscos e mitigação
12. Integração com o sistema existente

A décima terceira seção do template, **Questões em aberto e premissas**, é obrigatória sempre que
houver ao menos uma lacuna ou premissa marcada — o que é quase sempre, já que o FDD lida com
números e nomes concretos que nem sempre a fonte fecha por completo.

A seção 12 (Integração com o sistema existente) exige no mínimo **4 caminhos de arquivo reais**,
cada um com a descrição de como a feature estende ou reutiliza aquele ponto do código — por
exemplo, como um método existente passa a disparar um evento, ou como uma classe de erro existente
é estendida em vez de recriada. Menos de 4 caminhos reais é motivo para voltar ao código antes de
entregar, não para completar com uma descrição vaga.

Seção obrigatória sem material suficiente vira uma linha honesta que nomeia a lacuna — nunca texto
de enchimento. Se a fonte não define um número de retry, diga isso ao usuário em vez de inventar
um valor "razoável" sem marcá-lo como premissa.

### 7. Auditar antes de entregar

Releia o documento pronto contra este piso de qualidade. Se algum item falhar, corrija antes de
responder ao usuário:

- [ ] Arquivo existe em Markdown, salvo em `docs/FDD.md` (ou destino indicado pelo usuário)
- [ ] Contém as doze seções obrigatórias mais o cabeçalho de metadados
- [ ] "Fluxos detalhados" descreve passo a passo (não em prosa corrida) pelo menos os mecanismos
      centrais da feature — criação, processamento, tratamento de falha
- [ ] "Contratos públicos" traz pelo menos 1 endpoint com payload de exemplo, headers e tabela de
      status codes com semântica
- [ ] "Matriz de erros previstos" usa um prefixo de código consistente (o padrão já estabelecido
      pelo projeto ou pela fonte) e mapeia cada erro a status HTTP e causa
- [ ] "Estratégias de resiliência" traz números concretos (timeout, tentativas, fórmula de
      backoff) — nenhum "vamos garantir resiliência" sem quantificação
- [ ] "Observabilidade" nomeia métricas, campos de log ou spans específicos — não "vamos
      monitorar"
- [ ] "Integração com o sistema existente" nomeia no mínimo 4 arquivos reais, cada um confirmado
      no código-base, com a mudança concreta descrita
- [ ] "Riscos e mitigação" traz pelo menos 2 riscos de implementação com mitigação acionável
- [ ] Nenhum caminho de arquivo, classe ou método foi citado sem confirmação no código
- [ ] Nenhuma linha sem origem direta passou sem marcação de premissa ou registro de lacuna

Um FDD derivado de um RFC e de um código-base reais raramente tem só 1 fluxo ou só 1 contrato de
endpoint. Se você extraiu o mínimo exato de cada seção, releia as fontes antes de entregar — é
provável que tenha varrido por cima, especialmente do código.

## Convenções de identificadores

Identificadores estáveis são o que permite ao PRD, ao RFC, aos ADRs, ao tracker e aos testes
apontarem para o mesmo item sem ambiguidade. Numere sequencialmente e não reaproveite número de
item removido. Ao referenciar um item do FDD a partir do `docs/TRACKER.md`, prefixe com `FDD-`
(por exemplo, `FDD-CONTRATO-03`) — dentro do próprio FDD, o identificador local não carrega esse
prefixo.

| Prefixo | Uso |
| --- | --- |
| `FLUXO-NN` | Fluxo detalhado (criação, processamento, retry, fallback) |
| `CONTRATO-NN` | Endpoint ou contrato público |
| `ERRO-NN` | Erro previsto na matriz de erros |
| `RESIL-NN` | Estratégia de resiliência |
| `OBS-NN` | Item de observabilidade |
| `DEP-NN` | Dependência ou restrição de compatibilidade |
| `CA-NN` | Critério de aceite técnico |
| `RSC-NN` | Risco de implementação |
| `INT-NN` | Ponto de integração com o sistema existente |
| `P-NN` | Premissa assumida (⚠️) |

Formato de origem: `[RFC seção N]`, `[ADR-NNN]`, `[CODIGO caminho/do/arquivo::método]`,
`[TRANSCRICAO hh:mm]`. Mantenha o formato uniforme dentro do documento, e sempre que citar código
use o formato `caminho::símbolo` quando o símbolo (método, classe) for o ponto exato da mudança.

## Antipadrões

| Antipadrão | Por que dói | O que fazer |
| --- | --- | --- |
| Reabrir "por que X e não Y" que o RFC já decidiu | Duplica o RFC e diverge na primeira mudança | Assuma a decisão fechada; se a dúvida é real, é lacuna a levar de volta ao RFC/ADR, não a resolver aqui |
| Código de erro no padrão certo mas nunca definido pela fonte | Parece verificado, mas é invenção | Só entra na matriz o que a fonte (RFC/ADR/reunião) sustenta; o resto é premissa marcada |
| "Vamos garantir resiliência e boa observabilidade" sem números | Não dá para implementar nem testar | Timeout, contagem de retry, fórmula de backoff, nome de métrica — sempre concretos |
| Citar `order.service.ts` sem nomear o método afetado | Não dá pra saber onde mexer | Cite `arquivo::método` e descreva a mudança concreta naquele ponto |
| Caminho de arquivo ou classe que não existe no código | Quebra a rastreabilidade assim que alguém confere | Confirme no código antes de citar; se o ponto de integração ainda não existe, diga isso explicitamente |
| Contrato de endpoint sem payload de exemplo | Obriga o desenvolvedor a adivinhar o formato | Sempre incluir request/response de exemplo, headers e status codes |
| Copiar o diagrama de componentes do RFC sem descer um nível | É repetição, não é o "como" que o FDD promete | Detalhe cada componente do RFC até o nível de fluxo, contrato e erro |

## Ao entregar

Diga onde o arquivo foi escrito e reporte, em poucas linhas:

- quantos fluxos, contratos e erros foram documentados, e de qual fonte cada bloco principal veio
  (RFC, ADR, código, reunião);
- quantos e quais dos 4+ caminhos de arquivo da seção de integração foram confirmados no código;
- quais premissas você precisou marcar e por quê;
- quais lacunas ficaram registradas em "Questões em aberto e premissas";
- qualquer conteúdo identificado na fonte mas deixado de fora do FDD por ser altura de RFC/ADR
  (debate de arquitetura) ou de PRD (requisito de negócio) — isso poupa retrabalho e evita que a
  mesma decisão seja documentada duas vezes em lugares diferentes.

Não afirme cobertura total das fontes se você não as varreu inteiras, nem afirme ter confirmado um
caminho de código que não checou de fato.

## Arquivos de apoio

- `references/secoes-do-fdd.md` — o que pertence a cada uma das 12 seções, com exemplos de
  conteúdo forte e fraco. Leia antes de redigir.
- `assets/template-FDD.md` — esqueleto Markdown pronto, com as tabelas e colunas de origem.
