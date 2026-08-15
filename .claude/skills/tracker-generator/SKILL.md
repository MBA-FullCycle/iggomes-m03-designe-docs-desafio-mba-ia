---
name: tracker-generator
description: Gera e audita o Tracker de Rastreabilidade (`docs/TRACKER.md`), a tabela que mapeia cada item do PRD, do RFC, do FDD e dos ADRs à sua origem real na `TRANSCRICAO.md` ou no código. Use sempre que o usuário pedir um "tracker", uma "matriz de rastreabilidade", "docs/TRACKER.md", quiser verificar se a documentação técnica está alinhada com o que foi de fato discutido na reunião ou existe no código, ou precisar consolidar PRD/RFC/FDD/ADRs em uma referência cruzada única — mesmo que não use a palavra "tracker". Use também para revisar, atualizar ou auditar um tracker já existente, especialmente depois que qualquer um dos documentos-fonte (PRD, RFC, FDD, ADRs) for editado, já que o tracker fica desatualizado silenciosamente quando isso acontece. Este é o principal instrumento anti-alucinação do pacote de documentação: ele só deve ser gerado depois que pelo menos um dos documentos-fonte existir com conteúdo real.
---

# Geração do Tracker de rastreabilidade

## O que este documento é — e por que ele é diferente dos outros

O PRD, o RFC, o FDD e os ADRs *afirmam* coisas: requisitos, decisões, contratos, riscos. O tracker
não afirma nada de novo — ele **audita** o que os outros já afirmaram, uma linha por item, contra
duas únicas fontes de verdade aceitas: a `TRANSCRICAO.md` da reunião e o código real do
repositório. Ele não é conteúdo do desafio, é o mecanismo de checagem do desafio: a fonte diz
explicitamente que o tracker existe para "manter a integridade da documentação contra alucinações
da IA".

Isso muda o que "gerar o tracker" significa. Você não está criando informação — está **percorrendo
documentos que já foram (ou deveriam ter sido) escritos com citação de origem** e consolidando essa
citação em uma tabela única e uniforme. Se um documento-fonte não tem conteúdo real ainda (é um
rascunho vazio, como frequentemente acontece com `docs/PRD.md`, `docs/RFC.md` e `docs/FDD.md` no
início do processo), o tracker não tem o que auditar dele — diga isso ao usuário em vez de inventar
linhas para preencher a tabela.

## A regra inegociável: toda linha aponta para TRANSCRICAO ou CODIGO — nunca para outro documento

O formato obrigatório da tabela permite exatamente dois valores para a coluna **Fonte**:
`TRANSCRICAO` ou `CODIGO`. Isso é mais rígido do que as outras skills da família, que aceitam
citar `[RFC seção N]`, `[PRD seção]` ou `[ADR-NNN]` como origem de um item. No tracker, essas
citações intermediárias não são o destino — são uma etapa no caminho até a origem verdadeira, que
precisa ser resolvida (veja `references/resolucao-de-origem.md`).

Quando um item não resolve até uma citação real de transcrição ou código, existem exatamente duas
saídas legítimas:

1. **Registrar a linha com uma nota explícita de lacuna** na Localização (por exemplo, `SEM ORIGEM
   RASTREÁVEL — RFC seção 3 não cita transcrição nem código`) e contar isso contra o percentual de
   cobertura, em vez de forçar um `TRANSCRICAO` ou `CODIGO` que não existe.
2. **Omitir a linha e reportar a lacuna ao usuário** no resumo final, se incluir uma linha sem
   origem verificável distorceria mais a tabela do que ajudaria.

O que nunca é aceitável: inventar um timestamp que soa plausível para bater os 70% de linhas
`TRANSCRICAO`, citar um caminho de código que você não confirmou existir para bater as 5 linhas
`CODIGO` mínimas, ou copiar `[RFC seção 3]` para a coluna Fonte como se "RFC" fosse um valor válido.

## Fluxo

### 1. Levantar os documentos-fonte disponíveis

Liste o que existe de fato antes de começar:

- `docs/PRD.md`, `docs/RFC.md`, `docs/FDD.md`
- todo arquivo `docs/adrs/ADR-NNN-*.md` (ignore `docs/adrs/README.md`, que é só a convenção do
  diretório, não uma decisão)
- `TRANSCRICAO.md` na raiz, como referência para validar timestamps
- o código-fonte do repositório, como referência para validar caminhos

Um documento que ainda é um rascunho vazio (poucas linhas, sem seções preenchidas) não tem itens
para extrair — não force conteúdo dele. Reporte isso ao usuário como parte do relatório final, não
como um bloqueio: o tracker pode (e deve) ser regenerado conforme os outros documentos amadurecem.

### 2. Ler cada documento inteiro e extrair os itens com sua citação embutida

As skills irmãs (`prd-generator`, `rfc-generator`, `fdd-generator`, `adr-generator`) instruem cada
uma a marcar seus itens com um identificador estável (`RF-03`, `ALT-02`, `CONTRATO-01`, o próprio
`ADR-NNN`) e uma citação de origem entre colchetes logo ao lado ou na tabela de metadados da seção.
Percorra cada documento do início ao fim coletando pares **(identificador local, citação de
origem)** — não pule para o resumo executivo, os itens costumam estar espalhados pelo corpo do
documento inteiro.

Para os ADRs, o identificador principal é o próprio arquivo (`ADR-NNN`, Tipo = Decisão, origem
tirada da seção 2 ou 3). Vale a pena também extrair linhas adicionais das seções "Alternativas
Consideradas" e "Consequências" quando elas carregam uma citação própria e distinta da decisão
principal — use `ADR-NNN-ALT-01`, `ADR-NNN-CONSEQ-01` como identificador local nesses casos.

### 3. Resolver cada citação até TRANSCRICAO ou CODIGO

Leia `references/resolucao-de-origem.md` antes desta etapa — ela detalha o algoritmo de resolução
em cadeia (um item do FDD que cita `[RFC seção 3]`, cuja seção 3 por sua vez cita `[TRANSCRICAO
09:17]`, resolve para `TRANSCRICAO 09:17` no tracker) com exemplos de cadeias curtas, longas e
quebradas.

Para toda citação `[CODIGO caminho/do/arquivo]`, confirme que o caminho existe no repositório antes
de gravar a linha — não copie o caminho do documento-fonte sem checar, ele pode ter ficado
desatualizado desde que foi escrito.

Para toda citação `[TRANSCRICAO hh:mm]`, confirme contra `TRANSCRICAO.md` que aquele timestamp
existe e que o trecho ali sustenta razoavelmente o item — um timestamp que existe na transcrição
mas fala de outro assunto não é uma origem válida para este item.

### 4. Classificar o Tipo de cada item

Use o prefixo do identificador local para decidir o Tipo (Requisito Funcional, Requisito Não
Funcional, Decisão, Restrição, Trade-off, Risco, Critério de Aceite, Dependência, Premissa, Questão
em Aberto, entre outros — a coluna é aberta, não um enum fechado). A tabela completa de mapeamento
prefixo → Tipo, por documento, está em `references/resolucao-de-origem.md`.

### 5. Montar a tabela final

Use `assets/template-TRACKER.md` como esqueleto. Monte o ID do tracker prefixando o identificador
local com o documento de origem: `PRD-RF-03`, `RFC-ALT-02`, `FDD-CONTRATO-03`, `ADR-002`,
`ADR-002-ALT-01`. Preencha as seis colunas exatamente na ordem definida: ID, Documento, Tipo,
Conteúdo (resumo), Fonte, Localização. O resumo de Conteúdo é uma linha só — se está precisando de
duas frases para descrever o item, encurte até a essência, o detalhe completo já mora no documento
de origem.

Ordene a tabela por Documento (PRD, depois RFC, depois FDD, depois ADRs em ordem numérica) e, dentro
de cada documento, pela ordem em que os itens aparecem nele — isso torna o tracker fácil de conferir
lado a lado com o documento original.

### 6. Auditar antes de entregar

Releia a tabela pronta contra este piso de qualidade — os quatro primeiros itens são o critério de
aceite formal do desafio, os demais existem para que o tracker resista a uma auditoria real:

- [ ] Arquivo existe em `docs/TRACKER.md` e segue exatamente o formato de tabela de 6 colunas
- [ ] Pelo menos 80% dos itens identificáveis nos documentos-fonte disponíveis têm linha
      correspondente na tabela
- [ ] Pelo menos 70% das linhas têm Fonte = `TRANSCRICAO`, com timestamp no formato `[hh:mm] Nome`
      válido e conferido contra `TRANSCRICAO.md`
- [ ] Pelo menos 5 linhas têm Fonte = `CODIGO`, com caminho de arquivo real e confirmado no
      repositório
- [ ] Nenhuma linha usa um valor de Fonte diferente de `TRANSCRICAO` ou `CODIGO` — nenhuma linha
      diz "PRD", "RFC", "FDD" ou "ADR-NNN" na coluna Fonte
- [ ] Todo item marcado `⚠️ PREMISSA` no documento de origem foi tratado como lacuna, nunca
      forçado a uma origem que não existe
- [ ] Cada ADR existente em `docs/adrs/` (exceto o README do diretório) tem pelo menos uma linha
- [ ] Nenhum caminho de código ou timestamp de transcrição foi copiado sem checagem contra a fonte
      real

Se a contagem ficar abaixo de qualquer piso, a resposta correta é voltar às fontes e extrair com
mais cuidado — não afrouxar o critério nem completar a tabela com linhas de origem duvidosa.

## Convenções de identificadores por documento

| Documento | Prefixos locais (definidos nas skills irmãs) | Prefixo no tracker |
| --- | --- | --- |
| `docs/PRD.md` | `OBJ`, `RF`, `RNF`, `FE`, `DEC`, `DEP`, `RSC`, `CA` | `PRD-` |
| `docs/RFC.md` | `ALT`, `Q`, `P`, `RSC` | `RFC-` |
| `docs/FDD.md` | `FLUXO`, `CONTRATO`, `ERRO`, `RESIL`, `OBS`, `DEP`, `CA`, `RSC`, `INT`, `P` | `FDD-` |
| `docs/adrs/ADR-NNN-*.md` | o próprio arquivo, mais `ALT`, `CONSEQ` quando extraídos | (nenhum — usa `ADR-NNN` direto) |

O mapeamento completo prefixo → Tipo, com exemplos de linha forte e fraca para cada categoria,
está em `references/resolucao-de-origem.md`.

## Antipadrões

| Antipadrão | Por que dói | O que fazer |
| --- | --- | --- |
| Copiar `[RFC seção 3]` direto para a coluna Fonte | "RFC" não é um valor válido de Fonte — a tabela exige `TRANSCRICAO` ou `CODIGO` | Resolva a cadeia até a origem real antes de gravar a linha |
| Inventar timestamp plausível para bater os 70% de `TRANSCRICAO` | É o exato tipo de alucinação que o tracker existe para impedir | Só use timestamps confirmados em `TRANSCRICAO.md` |
| Citar caminho de código sem confirmar que existe | Quebra a rastreabilidade na primeira conferência | Verifique cada caminho no repositório antes de gravar |
| Gerar o tracker com PRD/RFC/FDD ainda em rascunho vazio | Produz uma tabela vazia ou fabricada para parecer completa | Diga ao usuário que ainda não há o que auditar; regenere quando os documentos tiverem conteúdo |
| Uma linha por documento inteiro ("PRD-01: requisitos do PRD") | Não é rastreabilidade, é um resumo disfarçado | Uma linha por item identificável, no grão que os documentos-fonte já definem |
| Deixar um ADR aceito de fora do tracker | Quebra a cobertura mínima e esconde uma decisão relevante | Todo ADR existente vira pelo menos uma linha |
| Reescrever o tracker do zero a cada chamada, sem checar o que já existe | Perde histórico e pode reverter correções manuais do usuário | Leia o `docs/TRACKER.md` atual primeiro; atualize incrementalmente quando fizer sentido |

## Ao entregar

Diga onde o arquivo foi escrito e reporte, em poucas linhas:

- quantas linhas foram geradas, por documento de origem, e o percentual de cobertura estimado
  contra os itens identificáveis nas fontes disponíveis;
- o percentual de linhas com Fonte `TRANSCRICAO` e a contagem de linhas com Fonte `CODIGO`;
- quais itens ficaram sem origem rastreável (cadeia quebrada, premissa sem citação) e foram
  registrados como lacuna ou omitidos;
- quais documentos-fonte ainda não têm conteúdo real e por isso não contribuíram itens — isso é
  esperado no início do processo, não uma falha.

Não afirme os percentuais do critério de aceite sem tê-los contado de fato na tabela final.

## Arquivos de apoio

- `references/resolucao-de-origem.md` — o algoritmo de resolução em cadeia até TRANSCRICAO/CODIGO,
  o mapeamento completo de prefixo → Tipo por documento, e exemplos de linha forte e fraca. Leia
  antes de extrair os itens.
- `assets/template-TRACKER.md` — esqueleto Markdown pronto com o cabeçalho de tabela exigido.
