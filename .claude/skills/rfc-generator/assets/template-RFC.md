# RFC — <Título da proposta>

| Campo | Valor |
| --- | --- |
| **Status** | Rascunho / Em revisão / Aprovado |
| **Autor(es)** | <nomes> |
| **Data** | <AAAA-MM-DD> |
| **Revisores** | <participantes da reunião fonte> |
| **Documentos relacionados** | <PRD, FDD, ADRs — quando existirem> |

<!--
Preenchimento do cabeçalho quando o RFC é derivado de uma reunião:
  Status    → "Rascunho" até alguém da reunião revisar; o documento nasce sem aprovação.
  Autor(es) → quem conduziu a proposta técnica na fonte.
  Revisores → os participantes da reunião fonte, pelo nome ou papel. Ex.: "<Tech Lead>,
              <Eng. Backend>, <Segurança>" — são eles que precisam revisar antes do Status
              virar "Em revisão".
  Documentos relacionados → aponte o PRD que motivou esta proposta, quando existir, e o FDD
              que vai detalhar a implementação, quando existir.
Remova este comentário no documento final.
-->

> Convenção de origem usada neste documento: `[TRANSCRICAO hh:mm]` para falas da reunião,
> `[CODIGO caminho/do/arquivo]` para o código existente, `[PRD secção]` para o PRD relacionado,
> `[ADR-NNN]` para um ADR já existente. Itens marcados com ⚠️ PREMISSA não têm origem direta na
> fonte e precisam de confirmação.

---

## 1. Resumo executivo (TL;DR)

<3 a 5 frases: o que propomos, por que essa abordagem e não outra, e o que muda para o sistema.
Alguém que só ler esta seção precisa sair sabendo o essencial da proposta.>

## 2. Contexto e problema

<O que motiva tecnicamente esta proposta: limitação do sistema atual, requisito do PRD que exige
uma decisão de arquitetura, ou problema técnico identificado na reunião. Não repita o problema de
negócio inteiro do PRD — referencie-o e foque no que é específico da decisão técnica.>

## 3. Proposta técnica

<Visão geral da solução: componentes envolvidos, fluxo de dados, padrões adotados, como as peças
se encaixam. Nível de arquitetura — sem nome de classe, schema exato de tabela ou payload
completo de endpoint; isso é FDD.>

## 4. Alternativas consideradas

*(Mínimo de 2 alternativas reais, discutidas e descartadas na fonte — não hipóteses inventadas
para preencher a seção.)*

### ALT-01 — <alternativa>

- **Descrição:** <o que essa alternativa propunha>
- **Por que foi descartada:** <o trade-off específico que motivou o descarte>
- **Origem:** `[...]`

### ALT-02 — <alternativa>

- **Descrição:** <o que essa alternativa propunha>
- **Por que foi descartada:** <o trade-off específico que motivou o descarte>
- **Origem:** `[...]`

## 5. Questões em aberto

*(Mínimo de 2 pontos levantados na reunião e não decididos ou adiados.)*

| ID | Questão | Quem decide / quando | Origem |
| --- | --- | --- | --- |
| Q-01 | <ponto levantado e não fechado> | <dono da decisão pendente, se souber> | `[...]` |
| Q-02 | <ponto levantado e não fechado> | <dono da decisão pendente, se souber> | `[...]` |

*(Se você precisou assumir algo para o documento fechar, registre aqui também, marcado com
⚠️ PREMISSA, com o que confirmaria e o que muda se estiver errado. Remova a tabela abaixo se não
houver nenhuma premissa.)*

| ID | Premissa assumida | O que confirmaria | Impacto se errada |
| --- | --- | --- | --- |
| P-01 | <suposição feita na ausência de fonte> | <quem/o que confirma> | <o que muda na proposta> |

## 6. Impacto e riscos

**Impacto.** <sistemas, times ou fluxos existentes afetados pela mudança>

| ID | Risco | Probabilidade | Impacto | Mitigação | Origem |
| --- | --- | --- | --- | --- | --- |
| RSC-01 | <risco concreto desta proposta> | Baixa / Média / Alta | Baixo / Médio / Alto | <ação concreta> | `[...]` |

## 7. Decisões relacionadas

*(Mínimo de 2 ADRs linkados. Se o ADR ainda não existe no repositório, não invente o link — liste
a decisão como pendente de virar ADR.)*

| ADR | Decisão | Situação |
| --- | --- | --- |
| [ADR-001](../adrs/ADR-001-titulo-da-decisao.md) | <decisão registrada> | Existente |
| — | <decisão que ainda precisa virar ADR> | Pendente de criação |
