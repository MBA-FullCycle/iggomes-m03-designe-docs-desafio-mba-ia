# PRD — <Nome da feature>

| Campo | Valor |
| --- | --- |
| **Status** | Rascunho / Em revisão / Aprovado |
| **Autor(es)** | <nomes> |
| **Data** | <AAAA-MM-DD> |
| **Fontes** | <arquivo(s), ticket(s), reunião e data> |
| **Documentos relacionados** | <RFC, FDD, ADRs — quando existirem> |

<!--
Preenchimento do cabeçalho quando o PRD é derivado de uma reunião:
  Status  → "Rascunho" até alguém da reunião revisar; o documento nasce sem aprovação.
  Autores → quem conduziu e decidiu na fonte, com os papéis. Ex.: "Derivado da reunião de
            2026-08-13, conduzida por <Tech Lead>, com <PM>, <Eng.>, <Segurança>."
  Fontes  → identifique a fonte primária e as secundárias separadamente.
Remova este comentário no documento final.
-->


> Convenção de origem usada neste documento: `[TRANSCRICAO hh:mm]` para falas da reunião,
> `[CODIGO caminho/do/arquivo]` para o código existente, `[TICKET ID]` para chamados.
> Itens marcados com ⚠️ PREMISSA não têm origem direta na fonte e precisam de confirmação.

---

## 1. Resumo e contexto da feature

<1 a 3 parágrafos: o que é, para quem, qual sistema toca e por que agora.>

## 2. Problema e motivação

**Problema.** <a dor, em termos de quem a sente hoje>

**Quem é afetado.** <perfis e volume, quando conhecido>

**Custo de não resolver.** <impacto de negócio, com números da fonte quando houver>

## 3. Público-alvo e cenários de uso

| Perfil | Necessidade | Cenário de uso | Origem |
| --- | --- | --- | --- |
| <perfil> | <o que precisa> | <situação concreta> | `[...]` |

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Valor atual | Meta | Como medir | Origem |
| --- | --- | --- | --- | --- | --- | --- |
| OBJ-01 | <objetivo> | <métrica> | <baseline ou "não medido"> | <meta quantitativa> | <instrumentação> | `[...]` |

## 5. Escopo

### 5.1 Incluso nesta fase

- <capacidade entregue> — `[...]`

### 5.2 Fora de escopo

| ID | Item | Motivo | Situação | Origem |
| --- | --- | --- | --- | --- |
| FE-01 | <item> | <por que ficou de fora> | Adiado / Descartado | `[...]` |
| FE-02 | <item> | <por que ficou de fora> | Adiado / Descartado | `[...]` |

## 6. Requisitos funcionais

| ID | Requisito | Prioridade | Origem |
| --- | --- | --- | --- |
| RF-01 | <enunciado verificável, sem detalhe de implementação> | Must | `[...]` |

## 7. Requisitos não funcionais

| ID | Requisito | Categoria | Origem |
| --- | --- | --- | --- |
| RNF-01 | <restrição de qualidade mensurável> | Desempenho / Confiabilidade / Segurança / Limites / Observabilidade / Compatibilidade | `[...]` |

## 8. Decisões e trade-offs principais

### DEC-01 — <decisão>

- **Decisão:** <o que foi fechado>
- **Alternativa considerada:** <o que foi colocado na mesa e rejeitado>
- **Motivo da rejeição:** <o trade-off que decidiu>
- **O que aceitamos perder:** <consequência assumida>
- **Origem:** `[...]`

## 9. Dependências

| ID | Dependência | Tipo | Impacto se ausente | Origem |
| --- | --- | --- | --- | --- |
| DEP-01 | <dependência> | Técnica / Organizacional / Externa / Dados | <o que acontece sem ela> | `[...]` |

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação | Responsável | Origem |
| --- | --- | --- | --- | --- | --- | --- |
| RSC-01 | <risco concreto desta feature> | Baixa / Média / Alta | Baixo / Médio / Alto | <ação concreta> | <quem> | `[...]` |

## 11. Critérios de aceitação

| ID | Critério | Requisitos cobertos |
| --- | --- | --- |
| CA-01 | <verificável por sim ou não> | RF-01, RNF-01 |

## 12. Estratégia de testes e validação

| Nível | O que valida nesta feature |
| --- | --- |
| Unitário | <...> |
| Integração | <...> |
| Ponta a ponta | <...> |
| Carga / resiliência | <...> |
| Segurança | <...> |

**Validação pós-deploy.** <métrica observada, janela de observação, critério de sucesso e de reversão>

## 13. Questões em aberto e premissas

*(Obrigatória sempre que houver ao menos uma lacuna ou premissa. Remova apenas se o documento
não tiver nenhuma das duas — o que é raro.)*

| ID | Item | Tipo | O que resolveria | Origem |
| --- | --- | --- | --- | --- |
| Q-01 | <ponto levantado e não decidido> | Questão em aberto | <quem decide, quando> | `[...]` |
| P-01 | <suposição que você precisou fazer> | ⚠️ Premissa | <confirmação necessária e o que muda se estiver errada> | — |
