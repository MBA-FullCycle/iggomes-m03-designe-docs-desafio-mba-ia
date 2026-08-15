---
name: adr-generator
description: Gera e refina um conjunto de ADRs (Architectural Decision Records, formato MADR) rastreáveis — um arquivo por decisão, nomeado `ADR-NNN-titulo-em-kebab-case.md`, dentro de `docs/adrs/`. Use sempre que o usuário pedir um ADR, "registro de decisão arquitetural", quiser documentar por que uma decisão técnica foi tomada e quais alternativas perderam, precisar transformar as decisões de uma reunião ou de um RFC já escrito em registros individuais, ou quiser garantir que as principais decisões arquiteturais de uma feature fiquem documentadas com contexto, decisão, alternativas e consequências — mesmo que não use a sigla "ADR" ou o termo "MADR". Use também para revisar, numerar, renumerar, superseder ou auditar um conjunto de ADRs existente, inclusive para checar se pelo menos um ADR referencia arquivos reais do código e se o conjunto cobre as decisões relevantes sem duplicar o RFC.
---

# Geração de ADRs rastreáveis

## O que este documento é — e onde ele para

Um ADR registra **uma decisão**, e só uma. Ele existe para que, daqui a seis meses, alguém possa
perguntar "por que não fizemos X em vez disso?" e a resposta já esteja escrita — com a alternativa
real que perdeu e o custo concreto que a decisão vencedora trouxe junto.

Isso separa o ADR dos outros documentos da família:

- **O RFC** pesa alternativas de arquitetura no nível da proposta inteira, e é intencionalmente
  conciso (2-4 páginas) — ele não tem espaço para defender cada decisão em profundidade. O RFC
  aponta para os ADRs em vez de reargumentar cada um. O ADR faz o trabalho que o RFC terceirizou:
  ir fundo em uma única escolha.
- **O FDD** documenta *como construir* o que já foi decidido — schema, payload, fluxo. O ADR não
  compete com isso; ele documenta *por que* a decisão foi essa e não outra. Um ADR que descreve o
  schema completo da tabela em vez de discutir a escolha do padrão desceu para o FDD.

O teste prático para saber se uma decisão merece seu próprio ADR: se alguém puder razoavelmente
perguntar "por que não X?" sobre esse ponto específico, e a resposta exigir nomear uma alternativa
real e um trade-off real, ela merece um ADR. Se é só "o jeito natural de construir o que já foi
decidido, sem alternativa real em disputa", é detalhe de FDD, não uma decisão nova.

**Um ADR, uma decisão.** Nunca junte duas decisões em um arquivo só porque as duas são "sobre a
mesma feature" — isso quebra o propósito do formato, que é cada decisão poder ser citada,
revisada e superseder de forma independente.

## A regra inegociável: nada sem origem — com uma flexibilização específica do ADR

Cada afirmação nasce de uma fonte identificável: a reunião, o RFC, um ADR anterior, ou o código
existente. Isso vale com força redobrada para dois pontos:

- **Alternativas consideradas.** Aqui o ADR é mais flexível que o RFC: a fonte pede pelo menos 1
  alternativa **real discutida ou plausível** — não as 2 alternativas estritamente debatidas que
  o RFC exige. Prefira sempre a alternativa que foi de fato levantada na reunião, com timestamp.
  Quando a fonte não registrou nenhum debate para essa decisão específica (o grupo foi direto à
  escolha), é aceitável incluir uma alternativa que um profissional real consideraria — mas ela
  precisa ser genuinamente plausível, nunca um espantalho, e precisa vir marcada com
  ⚠️ **não discutida na fonte** para que ninguém a confunda com algo que o time debateu de fato.
- **Consequências.** Uma decisão só está de fato defendida se a seção nomeia pelo menos um custo
  real, não só benefícios. Um ADR que só lista vantagens é propaganda da decisão, não um registro
  honesto dela.

Quando faltar informação para uma seção obrigatória, existem exatamente três saídas legítimas:

1. **Perguntar** ao usuário, se a resposta muda o conteúdo de forma relevante.
2. **Registrar como lacuna** — por exemplo, se a fonte não deixa claro quem participou da decisão.
3. **Marcar como premissa explícita**, com `⚠️ PREMISSA`.

O que nunca é aceitável: inventar uma alternativa disfarçada de discutida, omitir toda consequência
negativa, ou citar um arquivo de código que você não confirmou existir.

## Fluxo

### 1. Reunir as fontes

Para ADRs, a ordem de precedência é:

1. **RFC**, quando existir — a seção "Decisões relacionadas" dele costuma listar exatamente quais
   decisões ainda precisam virar ADR (marcadas como "Pendente de criação"). Comece por ali.
2. **Transcrição da reunião** — fonte primária do contexto, de quem defendeu cada alternativa e do
   trade-off que decidiu a escolha.
3. **ADRs já existentes em `docs/adrs/`** — liste o que já existe antes de numerar qualquer coisa
   nova, para não colidir número nem contradizer silenciosamente uma decisão já aceita (decisão
   que mudou vira um novo ADR que supersede o antigo, nunca uma edição silenciosa — ver seção de
   convenções abaixo).
4. **Código existente** — obrigatório para pelo menos 1 ADR do conjunto: confirme no código o
   arquivo, módulo, classe ou padrão citado antes de referenciá-lo.

Se o RFC ainda não existir ou estiver em rascunho vazio, trate a transcrição como fonte direta da
arquitetura também, do mesmo jeito que o `rfc-generator` faz quando o PRD ainda não existe, e
sinalize isso no relatório final.

Pergunte apenas o que muda o conteúdo do documento e você não consegue deduzir — por exemplo,
quantas decisões a fonte realmente sustenta como ADR, se isso ficar abaixo do que foi pedido.
Destino padrão: um arquivo por decisão em `docs/adrs/`; idioma o das fontes.

### 2. Ler a fonte inteira, do começo ao fim

Leia a transcrição e o RFC completos antes de escrever qualquer ADR. Em reuniões técnicas, a
alternativa que perdeu e o motivo específico do descarte aparecem no meio do debate — quando
alguém propõe, é contestado e o grupo converge — não no resumo final, que só registra a decisão
vencedora.

### 3. Identificar quais decisões merecem virar ADR

Antes de escrever, monte a lista de decisões candidatas e aplique o teste da seção anterior a cada
uma. Classifique:

| Classe | O que é | Vira ADR? |
| --- | --- | --- |
| `DECISÃO-ARQUITETURAL` | Escolha de padrão ou mecanismo com alternativa real que perdeu | Sim |
| `DECISÃO-CONVENÇÃO` | Reuso deliberado de um padrão já existente no projeto em vez de criar algo novo | Sim, quando foi uma escolha consciente, não a única opção óbvia |
| `DECISÃO-TÁTICA` | Detalhe de implementação sem alternativa real em disputa (nome de campo, formato de header) | Normalmente não — fica no FDD; vira ADR só se a fonte pedir explicitamente |
| `CONTEXTO-CÓDIGO` | Arquivo, módulo, classe ou padrão real citado como restrição ou como algo reaproveitado | Evidência de apoio dentro do ADR relevante, não uma seção própria |

Se a fonte (por exemplo, um requisito do desafio ou do usuário) já enumera explicitamente o
conjunto de decisões esperado e uma cobertura mínima (como "pelo menos 5 das 6 decisões X, Y, Z"),
essa lista é autoritativa — trate-a como o ponto de partida da classificação, não como uma
sugestão a ser reinterpretada livremente.

### 4. Montar o inventário rastreado

Consolide as evidências em uma tabela de trabalho — decisão, classe, alternativas com origem,
consequências com origem, arquivos de código confirmados. Ordene as decisões de um jeito que
favoreça a leitura (por exemplo, decisões das quais outras dependem antes das que dependem delas)
para decidir a numeração final.

Use um arquivo de rascunho fora do repositório do usuário para isso, não polua `docs/`.

### 5. Escrever os ADRs

Use `assets/template-ADR.md` como esqueleto (um arquivo por decisão) e
`references/secoes-do-adr.md` para saber o que pertence a cada seção, com exemplos de conteúdo
forte e fraco. As cinco seções obrigatórias por ADR são:

1. Status
2. Contexto
3. Decisão
4. Alternativas Consideradas (mínimo 1 — real discutida ou plausível, marcada quando não veio da
   fonte)
5. Consequências (positivas **e** negativas, com trade-off explícito)

Uma sexta seção, **Decisões relacionadas**, é recomendada sempre que houver um RFC ou outros ADRs
para linkar — evita que o ADR repita o "Contexto" inteiro do RFC e mantém os documentos
apontando uns para os outros em vez de duplicando conteúdo.

Antes de fechar o conjunto:

- Confirme que pelo menos 1 ADR cita, de fato, um arquivo/módulo/classe real do código-base —
  verificado no código, não citado de memória.
- Confirme que o conjunto cobre a cota mínima de decisões que a fonte pediu, se houver uma. Se
  ficar abaixo, isso é uma lacuna a reportar ao usuário, não motivo para inventar uma decisão
  que a fonte não sustenta.
- Se o RFC já existir e tiver uma linha "Pendente de criação" para uma decisão que você acabou de
  documentar, atualize o link daquela linha para apontar para o ADR novo (ou avise o usuário para
  fazê-lo) — um link pendente que já tem arquivo correspondente é uma inconsistência fácil de
  evitar.

Seção obrigatória sem material suficiente vira uma linha honesta que nomeia a lacuna — nunca texto
de enchimento.

### 6. Auditar antes de entregar

Releia o conjunto pronto contra este piso de qualidade. Se algum item falhar, corrija antes de
responder ao usuário:

- [ ] `docs/adrs/` contém entre 5 e 8 arquivos (ou a quantidade que a fonte pedir) no formato
      `ADR-NNN-titulo-em-kebab-case.md`
- [ ] Cada ADR contém as cinco seções obrigatórias: Status, Contexto, Decisão, Alternativas
      Consideradas, Consequências
- [ ] Cada "Alternativas Consideradas" tem no mínimo 1 alternativa, marcada com
      ⚠️ quando não veio de um debate real registrado na fonte
- [ ] Cada "Consequências" traz ao menos 1 item positivo e 1 negativo — nenhum ADR só com upside
- [ ] O conjunto cobre a cota mínima de decisões exigida pela fonte, quando ela existir
- [ ] Pelo menos 1 ADR referencia, com caminho confirmado, um arquivo/módulo/classe real do código
- [ ] Numeração sequencial (`ADR-NNN`), sem lacunas nem número reaproveitado, contínua com o que já
      existia em `docs/adrs/`
- [ ] Nenhuma decisão está duplicada em dois ADRs diferentes
- [ ] Nenhum ADR reescreveu silenciosamente uma decisão já "Aceita" — mudanças viraram um novo ADR
      que supersede o anterior
- [ ] Links cruzados entre RFC e ADRs resolvem — nenhum aponta para um arquivo inexistente

Um conjunto de ADRs derivado de uma reunião técnica real raramente cobre menos de metade das
decisões que a fonte contém. Se você fechou exatamente a cota mínima, releia a transcrição antes
de entregar — é provável que tenha uma decisão real ainda sem registro.

## Convenções de identificadores e numeração

- **Nome de arquivo:** `ADR-NNN-titulo-em-kebab-case.md`, com `NNN` de 3 dígitos com zero à
  esquerda (`001`, `002`, ...). Exemplo: `ADR-001-outbox-no-mysql.md`.
- **Numeração sequencial e permanente.** Antes de numerar um ADR novo, liste o que já existe em
  `docs/adrs/` e continue do maior número encontrado. Nunca reaproveite o número de um ADR
  rejeitado, descontinuado ou substituído — o número marca um lugar na história da decisão, não
  um slot reutilizável.
- **Ciclo de vida do campo Status:** `Proposta` → `Aceita` → (`Rejeitada` | `Descontinuada` |
  `Substituída por ADR-NNN`). Use `Rejeitada` apenas quando a própria fonte apresentou e descartou
  uma decisão como opção de primeiro nível — alternativas que perderam dentro da decisão vencedora
  vivem em "Alternativas Consideradas" de outro ADR, não como um ADR `Rejeitada` à parte.
- **Nunca edite silenciosamente um ADR `Aceita`.** Se uma decisão aceita muda depois, escreva um
  ADR novo que a supersede — com seção "Decisões relacionadas" apontando para o antigo — e
  atualize o Status do antigo para `Substituída por ADR-NNN`. Reescrever a Decisão ou as
  Consequências de um ADR já aceito apaga o registro histórico que é a razão de o ADR existir.

Formato de origem: `[TRANSCRICAO hh:mm]`, `[CODIGO caminho/do/arquivo::símbolo]`,
`[RFC seção N]`, `[ADR-NNN]`. Mantenha o formato uniforme dentro do documento.

## Antipadrões

| Antipadrão | Por que dói | O que fazer |
| --- | --- | --- |
| ADR só com consequências positivas | Vira propaganda da decisão, não um registro honesto | Nomeie ao menos um custo real, mesmo que pequeno |
| Alternativa espantalho disfarçada de discutida | Não é rastreável, é teatro de decisão | Marque explicitamente ⚠️ quando a alternativa não veio de um debate real |
| Duas decisões em um único arquivo | Impede superseder ou citar uma sem arrastar a outra | Um ADR por decisão, sempre |
| Editar um ADR `Aceita` para refletir uma mudança de ideia | Apaga o histórico que justifica o formato existir | Escreva um novo ADR que supersede o antigo |
| ADR com o schema completo da tabela, payload inteiro do endpoint | É altura de FDD, duplica e diverge na primeira mudança | Mantenha o ADR na decisão; mova o detalhe de implementação para o FDD |
| Citar um arquivo ou classe que não existe no código | Parece verificado e não é — quebra a rastreabilidade | Confirme no código-base antes de citar |
| Repetir o "Contexto" inteiro do RFC ou do PRD | Duplicação que diverge na primeira mudança | Referencie o RFC/PRD em uma frase e vá direto ao que é específico desta decisão |

## Ao entregar

Diga onde os arquivos foram escritos e reporte, em poucas linhas:

- quantos ADRs foram criados, com a faixa de numeração usada;
- quais das decisões exigidas pela fonte (quando houver uma lista explícita) foram cobertas e
  qual, se alguma, ficou de fora e por quê;
- qual ADR cita código real, e qual arquivo/módulo/classe;
- quais alternativas vieram marcadas como "não discutidas na fonte" e por que foram incluídas
  mesmo assim;
- quais premissas você precisou marcar;
- qualquer link que você atualizou no RFC (de "Pendente de criação" para o ADR novo).

Não afirme cobertura total da fonte se você não a varreu inteira.

## Arquivos de apoio

- `references/secoes-do-adr.md` — o que pertence a cada uma das seções do ADR, com exemplos de
  conteúdo forte e fraco. Leia antes de redigir.
- `assets/template-ADR.md` — esqueleto Markdown pronto para um ADR individual.
