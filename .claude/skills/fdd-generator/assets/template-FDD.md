# FDD — <Título da feature>

| Campo | Valor |
| --- | --- |
| **Status** | Rascunho / Em revisão / Aprovado |
| **Autor(es)** | <nomes> |
| **Data** | <AAAA-MM-DD> |
| **Revisores/Aprovadores** | <quem vai revisar antes da implementação começar> |
| **Documentos relacionados** | <RFC, ADRs — quando existirem> |

<!--
Preenchimento do cabeçalho quando o FDD é derivado de um RFC aprovado:
  Status    → "Rascunho" até alguém revisar; o documento nasce sem aprovação.
  Autor(es) → quem detalhou a implementação a partir do RFC/ADRs/reunião.
  Revisores/Aprovadores → normalmente quem vai implementar e um revisor técnico do RFC.
  Documentos relacionados → aponte o RFC que esta implementação detalha e os ADRs relevantes.
Remova este comentário no documento final.
-->

> Convenção de origem usada neste documento: `[RFC seção N]` para a proposta técnica aprovada,
> `[ADR-NNN]` para uma decisão já registrada, `[CODIGO caminho/do/arquivo::símbolo]` para o
> código-base existente (confirmado antes de citar), `[TRANSCRICAO hh:mm]` para falas da reunião.
> Itens marcados com ⚠️ PREMISSA não têm origem direta na fonte e precisam de confirmação.

---

## 1. Contexto e motivação técnica

<Ponte para o RFC/ADR que esta implementação detalha, e o que especificamente este FDD cobre que
eles deixaram em nível de componente. Não repita o problema de negócio do PRD.>

## 2. Objetivos técnicos

<O que precisa ser verdadeiro no sistema, em termos verificáveis no código, quando a
implementação estiver pronta.>

## 3. Escopo e exclusões

<O que este FDD implementa nesta fase e o que fica explicitamente fora — inclusive partes do RFC
adiadas para depois.>

## 4. Fluxos detalhados

*(Um bloco por mecanismo central — criação, processamento, retry, fallback/DLQ. Numere os passos;
cubra caminho feliz e caminho de falha de cada um.)*

### FLUXO-01 — <nome do mecanismo>

1. <passo>
2. <passo>
3. <passo>

**Origem:** `[...]`

### FLUXO-02 — <nome do mecanismo>

1. <passo>
2. <passo>

**Origem:** `[...]`

## 5. Contratos públicos

*(Um bloco por endpoint ou contrato exposto/consumido pela feature.)*

### CONTRATO-01 — `<MÉTODO> <caminho>`

**Headers:** `<headers relevantes — auth, idempotência, assinatura>`

**Request:**
```json
{}
```

**Response `<status>`:**
```json
{}
```

| Status | Semântica |
| --- | --- |
| <status> | <o que significa> |

**Origem:** `[...]`

## 6. Matriz de erros previstos

*(Use o prefixo de código já estabelecido pelo projeto/fonte — não invente um esquema novo.)*

| Código | Causa | Status | Ação esperada do cliente | Origem |
| --- | --- | --- | --- | --- |
| `<PREFIXO>_<NOME>` | <causa concreta> | <status HTTP> | <o que o cliente deve fazer> | `[...]` |

## 7. Estratégias de resiliência

<Timeout por tentativa, número máximo de tentativas, fórmula de backoff, critério de fallback ou
circuit breaker, garantia de idempotência — sempre com números concretos, nunca "resiliência
adequada".>

## 8. Observabilidade

<Métricas nomeadas (com tipo), campos de log específicos, spans de tracing — reaproveitando a
infraestrutura de observabilidade já existente quando houver.>

## 9. Dependências e compatibilidade

<Bibliotecas/serviços externos novos, com versão quando relevante, e o que não pode quebrar em
consumidores existentes.>

## 10. Critérios de aceite técnicos

*(Formato dado-quando-então; cada um verificável objetivamente.)*

- **CA-01.** Dado <contexto>, quando <ação>, então <comportamento esperado>.
- **CA-02.** Dado <contexto>, quando <ação>, então <comportamento esperado>.

## 11. Riscos e mitigação

*(Mínimo de 2 riscos concretos desta implementação — corrida, migração, comportamento sob carga —
não risco genérico de arquitetura ou de produto.)*

| ID | Risco | Cenário concreto | Mitigação | Origem |
| --- | --- | --- | --- | --- |
| RSC-01 | <risco> | <o que precisa acontecer para o risco se materializar> | <ação concreta> | `[...]` |

## 12. Integração com o sistema existente

*(Mínimo de 4 caminhos de arquivo reais, confirmados no código-base — não citados de memória.)*

| Arquivo | Ponto de integração | Mudança |
| --- | --- | --- |
| `<caminho>::<símbolo>` | <o que já existe ali> | <o que a feature nova faz nesse ponto> |
| `<caminho>::<símbolo>` | <o que já existe ali> | <o que a feature nova faz nesse ponto> |
| `<caminho>::<símbolo>` | <o que já existe ali> | <o que a feature nova faz nesse ponto> |
| `<caminho>::<símbolo>` | <o que já existe ali> | <o que a feature nova faz nesse ponto> |

## 13. Questões em aberto e premissas

*(Obrigatória sempre que houver ao menos uma lacuna ou premissa — o que é quase sempre neste
documento.)*

| ID | Questão em aberto | Quem decide / quando | Origem |
| --- | --- | --- | --- |
| Q-01 | <ponto levantado e não fechado> | <dono da decisão pendente, se souber> | `[...]` |

| ID | Premissa assumida | O que confirmaria | Impacto se errada |
| --- | --- | --- | --- |
| P-01 | <suposição feita na ausência de fonte> | <quem/o que confirma> | <o que muda na implementação> |
