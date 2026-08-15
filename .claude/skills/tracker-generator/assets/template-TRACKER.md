<!--
Ordene as linhas por Documento (PRD, RFC, FDD, ADRs em ordem numérica) e, dentro de cada
documento, pela ordem em que os itens aparecem nele. Remova este comentário e as linhas de
exemplo abaixo no documento final — elas ilustram o formato, não são conteúdo real.

Fonte aceita apenas dois valores: TRANSCRICAO ou CODIGO. Nunca "PRD", "RFC", "FDD" ou "ADR-NNN" —
essas são citações intermediárias que precisam ser resolvidas até a origem final antes de virar
linha aqui (veja references/resolucao-de-origem.md).
-->

# Tracker de Rastreabilidade

Mapeia cada item registrado no PRD, no RFC, no FDD e nos ADRs à sua origem real na
`TRANSCRICAO.md` ou no código do repositório. Regenere ou atualize este arquivo sempre que um dos
documentos-fonte mudar — um tracker desatualizado é pior do que a ausência de um, porque passa
confiança falsa.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| -- | --------- | ---- | ------------------ | ----- | ----------- |
| `PRD-RF-01` | `docs/PRD.md` | Requisito Funcional | <resumo de uma linha> | TRANSCRICAO | `[hh:mm] Nome` |
| `RFC-ALT-01` | `docs/RFC.md` | Trade-off | <resumo de uma linha> | TRANSCRICAO | `[hh:mm] Nome` |
| `FDD-CONTRATO-01` | `docs/FDD.md` | Requisito Funcional | <resumo de uma linha> | CODIGO | `src/modules/.../arquivo.ts` |
| `ADR-001` | `docs/adrs/ADR-001-titulo.md` | Decisão | <resumo de uma linha> | CODIGO | `prisma/schema.prisma` |

## Cobertura

<Preencha ao final, com os números realmente contados na tabela acima — não estime.>

- Itens identificáveis nos documentos-fonte disponíveis: <N>
- Linhas no tracker: <N> (<XX%> de cobertura)
- Linhas com Fonte = TRANSCRICAO: <N> (<XX%>)
- Linhas com Fonte = CODIGO: <N>
- Itens sem origem rastreável (lacuna): <N> — <liste os IDs, se houver>
