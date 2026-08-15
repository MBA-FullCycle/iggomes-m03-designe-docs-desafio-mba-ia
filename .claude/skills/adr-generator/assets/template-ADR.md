<!--
Um arquivo por decisão. Nomeie como ADR-NNN-titulo-em-kebab-case.md (ex: ADR-001-outbox-no-mysql.md),
com NNN sequencial de 3 dígitos, continuando a partir do maior número já existente em docs/adrs/.
Remova este comentário no documento final.
-->

# ADR-NNN — <Título da decisão, em poucas palavras>

## 1. Status

<Um dos valores: Proposta / Aceita / Rejeitada / Descontinuada / Substituída por ADR-NNN.
Inclua a data e, quando a fonte permitir, quem participou da decisão.>

**Status:** <valor> — <AAAA-MM-DD>
**Decisores:** <nomes ou papéis, quando a fonte identificar>

## 2. Contexto

<O que motivou especificamente esta decisão — não o problema de negócio inteiro (isso é PRD) nem
a proposta de arquitetura completa (isso é RFC). Referencie o RFC/PRD em uma frase se eles já
cobrem o pano de fundo, e vá direto à restrição ou pergunta que esta decisão respondeu.>

## 3. Decisão

<Uma frase clara e sem hedge: o que foi decidido. Não liste as opções aqui — isso vai na seção
seguinte.>

## 4. Alternativas Consideradas

*(Mínimo de 1 alternativa — real discutida na fonte ou, na ausência de debate registrado,
plausível e claramente marcada como tal.)*

### <Nome da alternativa>

- **Descrição:** <o que essa alternativa propunha>
- **Por que foi descartada:** <o trade-off específico>
- **Origem:** `[...]` *(ou `⚠️ Não discutida na fonte — incluída por ser tecnicamente plausível`)*

## 5. Consequências

*(Pelo menos 1 item positivo e 1 negativo — um ADR só com benefícios não está defendendo a
decisão, só elogiando.)*

**Positivas:**
- <consequência positiva concreta>

**Negativas:**
- <consequência negativa ou custo concreto>

**Origem:** `[...]`

## 6. Decisões relacionadas

*(Recomendada sempre que houver RFC, PRD, FDD ou outros ADRs relevantes para linkar — evita
repetir o conteúdo deles aqui.)*

- Implementa: `[RFC seção N](../RFC.md#...)` <ou "Nenhum RFC associado ainda">
- Detalhado em: `[FDD seção N](../FDD.md#...)` <se já existir>
- Substitui: `[ADR-NNN](./ADR-NNN-titulo-anterior.md)` <ou "Nenhum">
- Substituída por: `[ADR-NNN](./ADR-NNN-titulo-novo.md)` <preencher só quando este ADR for superado>
