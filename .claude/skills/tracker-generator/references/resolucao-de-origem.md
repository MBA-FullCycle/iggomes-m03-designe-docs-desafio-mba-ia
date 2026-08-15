# Resolução de origem e mapeamento de tipos

## Por que a resolução em cadeia existe

As outras skills da família citam origem de duas formas diferentes:

- **Origem primária**, direto na fonte de verdade: `[TRANSCRICAO hh:mm]` ou `[CODIGO caminho]`.
- **Origem secundária**, apontando para outro documento já rastreado: `[PRD seção N]`,
  `[RFC seção N]`, `[ADR-NNN]`.

A segunda forma é legítima dentro do RFC, do FDD e dos ADRs — evita repetir a justificativa inteira
toda vez que um documento se apoia em uma decisão já registrada em outro lugar. Mas o formato da
tabela do tracker só aceita `TRANSCRICAO` ou `CODIGO` na coluna Fonte. Isso significa que toda
citação secundária precisa ser **seguida até o fim** antes de virar uma linha do tracker.

## O algoritmo

1. Pegue a citação de origem do item, como está no documento onde ele vive.
2. Se já é `[TRANSCRICAO hh:mm]` ou `[CODIGO caminho]`, pare — essa é a Fonte/Localização final.
3. Se é uma citação secundária (`[PRD seção N]`, `[RFC seção N]`, `[ADR-NNN]`), abra o documento
   apontado e localize a seção ou item citado.
4. Repita o passo 1 a partir da citação de origem *daquele* item — a cadeia pode ter mais de um
   salto (FDD → RFC → PRD → TRANSCRICAO não é incomum).
5. Se a cadeia termina em um item marcado `⚠️ PREMISSA` sem citação de origem própria, ou em uma
   seção que não carrega nenhuma citação, a cadeia está quebrada — não force um `TRANSCRICAO` ou
   `CODIGO`. Registre como lacuna.

Um salto perdido no meio do caminho (por exemplo, esquecer de abrir o RFC e assumir que a citação
do FDD já é final) é o erro mais comum nesta etapa — é fácil parar cedo demais.

## Exemplos de cadeia

**Cadeia curta (1 salto), o caso mais comum:**

Item `FDD-ERRO-04` cita `[TRANSCRICAO 11:42]` diretamente na matriz de erros do FDD.
→ Fonte = `TRANSCRICAO`, Localização = `[11:42] <nome do falante, conferido na transcrição>`.
Nenhuma resolução adicional necessária.

**Cadeia média (2 saltos):**

Item `FDD-INT-02` cita `[RFC seção 3]`. Abrindo `docs/RFC.md`, a seção 3 (Proposta técnica) cita
`[CODIGO src/modules/orders/order.service.ts::changeStatus]` ao descrever o ponto de integração.
→ Fonte = `CODIGO`, Localização = `src/modules/orders/order.service.ts`. O Documento da linha no
tracker continua sendo `docs/FDD.md` (onde o item vive), não `docs/RFC.md`.

**Cadeia longa (3 saltos):**

Item `ADR-004` (Contexto) cita `[RFC seção 2]`. A seção 2 do RFC cita `[PRD seção 5]`. A seção 5
do PRD cita `[TRANSCRICAO 09:17]`.
→ Fonte = `TRANSCRICAO`, Localização = `[09:17] <falante>`.

**Cadeia quebrada:**

Item `RFC-ALT-03` cita `[ADR-002]`. Abrindo o ADR-002, a alternativa correspondente está marcada
`⚠️ Não discutida na fonte — incluída por ser tecnicamente plausível`, sem citação de transcrição
ou código.
→ Sem origem rastreável. Registre a linha com Localização `SEM ORIGEM RASTREÁVEL — ADR-002
alternativa não discutida na fonte original` ou omita e reporte a lacuna ao usuário.

## Mapeamento de prefixo → Tipo

A coluna Tipo é aberta ("entre outros" no formato exigido) — use o bom senso quando um item não se
encaixa perfeitamente numa categoria. Estas são as correspondências mais diretas:

### `docs/PRD.md` (prefixo no tracker: `PRD-`)

| Prefixo local | Tipo sugerido |
| --- | --- |
| `OBJ-NN` | Objetivo / Métrica de negócio |
| `RF-NN` | Requisito Funcional |
| `RNF-NN` | Requisito Não Funcional |
| `FE-NN` | Restrição (fora de escopo) |
| `DEC-NN` | Decisão |
| `DEP-NN` | Dependência |
| `RSC-NN` | Risco |
| `CA-NN` | Critério de Aceite |

### `docs/RFC.md` (prefixo no tracker: `RFC-`)

| Prefixo local | Tipo sugerido |
| --- | --- |
| `ALT-NN` | Trade-off (alternativa descartada) |
| `Q-NN` | Questão em Aberto |
| `P-NN` | Premissa |
| `RSC-NN` | Risco |

### `docs/FDD.md` (prefixo no tracker: `FDD-`)

| Prefixo local | Tipo sugerido |
| --- | --- |
| `FLUXO-NN` | Requisito Funcional |
| `CONTRATO-NN` | Requisito Funcional (contrato público) |
| `ERRO-NN` | Restrição (comportamento de erro) |
| `RESIL-NN` | Requisito Não Funcional |
| `OBS-NN` | Requisito Não Funcional |
| `DEP-NN` | Dependência |
| `CA-NN` | Critério de Aceite |
| `RSC-NN` | Risco |
| `INT-NN` | Restrição (integração com o existente) |
| `P-NN` | Premissa |

### `docs/adrs/ADR-NNN-*.md` (sem prefixo adicional — usa `ADR-NNN` direto)

| Identificador local | Tipo sugerido |
| --- | --- |
| `ADR-NNN` (a decisão em si) | Decisão |
| `ADR-NNN-ALT-NN` (alternativa considerada) | Trade-off |
| `ADR-NNN-CONSEQ-NN` (consequência nomeada) | Trade-off ou Restrição, conforme o conteúdo |

## Exemplos de linha forte e fraca

**Forte:**

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| `FDD-ERRO-04` | `docs/FDD.md` | Restrição | Webhook com assinatura HMAC inválida retorna 401 sem reprocessar | TRANSCRICAO | `[11:42] Bruno` |
| `ADR-002` | `docs/adrs/ADR-002-outbox-no-mysql.md` | Decisão | Outbox pattern implementado em tabela MySQL própria, sem broker externo | CODIGO | `prisma/schema.prisma` |

Cada linha aponta para uma origem que qualquer leitor consegue abrir e conferir em segundos.

**Fraca (evite):**

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| `RFC-ALT-01` | `docs/RFC.md` | Trade-off | Alternativa considerada | RFC | seção 4 |

Dois problemas: "Alternativa considerada" não resume o conteúdo real do item, e `RFC` não é um
valor válido de Fonte — é a citação intermediária que precisava ter sido resolvida até
`TRANSCRICAO` ou `CODIGO` antes de virar linha da tabela.
