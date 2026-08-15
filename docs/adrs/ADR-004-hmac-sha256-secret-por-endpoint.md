# ADR-004 — Autenticação por HMAC-SHA256 com secret única por endpoint e rotação com grace period

## 1. Status

**Status:** Aceita — reunião técnica de quinta-feira, 09:00 (a fonte não registra a data completa; ADR redigido em 2026-08-14)
**Decisores:** Sofia (Eng. de Segurança, proponente), Larissa (Tech Lead), Diego (Eng. Sênior — Plataforma), Bruno (Eng. Pleno — Time de Pedidos)

## 2. Contexto

A feature expõe dados de pedidos para endpoints HTTP **fora da nossa infraestrutura**. Isso cria dois problemas que não existiam quando toda a comunicação era inbound e protegida por JWT `[CODIGO src/middlewares/auth.middleware.ts]`: o cliente precisa conseguir provar que a requisição **veio realmente de nós**, e que **ninguém adulterou o payload no caminho** `[TRANSCRICAO 09:19 Sofia]`.

O escopo é exclusivamente **outbound** — os clientes recebem, não enviam `[TRANSCRICAO 09:02 Marcos]`, `[TRANSCRICAO 09:03 Sofia]` — o que dispensa qualquer verificação de assinatura no sentido inverso.

## 3. Decisão

Assinar o corpo da requisição com **HMAC-SHA256** e enviar a assinatura no header `X-Signature`. Cada endpoint de webhook cadastrado tem uma **secret única**, gerada por nós e devolvida ao cliente no momento da criação — não existe secret global de plataforma `[TRANSCRICAO 09:21 Sofia]`.

A secret é **rotacionável pela API**. Ao rotacionar, a secret antiga permanece válida **em paralelo por 24 horas**, dando ao cliente tempo de migrar os sistemas dele; depois disso, ela deixa de ser aceita `[TRANSCRICAO 09:21 Sofia]`, `[TRANSCRICAO 09:22 Sofia]`.

A URL do webhook é obrigatoriamente **HTTPS**; cadastro com `http` é recusado com erro de validação. Isso não é decisão arquitetural, é validação no schema Zod `[TRANSCRICAO 09:23 Sofia]`, e fica detalhada no [FDD](../FDD.md#5-contratos-públicos).

## 4. Alternativas Consideradas

### Secret global única da plataforma

- **Descrição:** uma única chave compartilhada por todos os clientes, simplificando geração, armazenamento e rotação.
- **Por que foi descartada:** o raio de explosão de um vazamento seria total — "se vaza uma, vaza tudo". Com secret por endpoint, comprometer a chave de um cliente não compromete nenhum outro. O risco não é hipotético: o time já teve um cliente que vazou a própria secret em log de aplicação.
- **Origem:** `[TRANSCRICAO 09:21 Sofia]`, `[TRANSCRICAO 09:22 Diego]`

### Autenticação mútua por TLS (mTLS) ou assinatura assimétrica

- **Descrição:** provar a origem via certificado cliente/servidor ou assinar com chave privada nossa, verificável pela chave pública do cliente — elimina o segredo compartilhado.
- **Por que foi descartada:** ⚠️ **Não discutida na fonte** — incluída por ser a alternativa de mercado mais séria ao HMAC. Ela exigiria gestão de certificados ou de pares de chaves nos dois lados, elevando muito a barreira de integração para clientes B2B que hoje só sabem consumir uma API REST. O argumento que sustentou o HMAC-SHA256 na reunião foi exatamente esse: é o padrão de mercado e "todo cliente sério tem biblioteca pra isso" `[TRANSCRICAO 09:20 Sofia]`.
- **Origem:** ⚠️ Não discutida na fonte — incluída por ser tecnicamente plausível

## 5. Consequências

**Positivas:**

- **Raio de explosão de vazamento limitado a um cliente.** É o ganho central sobre a secret global.
- **Rotação sem downtime para o cliente:** as 24 horas de grace period permitem que ele troque a chave nos próprios sistemas sem uma janela em que os webhooks quebram.
- **Barreira de integração baixa:** HMAC-SHA256 é implementável com biblioteca padrão em qualquer linguagem, sem dependência exótica do lado do cliente.

**Negativas:**

- **O grace period é, ele próprio, uma janela de risco.** Durante 24 horas após a rotação, **duas** secrets são aceitas — inclusive a que motivou a rotação, se o motivo foi vazamento. É um trade-off consciente entre continuidade do cliente e fechamento imediato da exposição, e implica que rotação **não é** o mecanismo de resposta a incidente urgente: para isso é preciso desativar o endpoint.
- **As secrets viram um ativo sensível no nosso banco.** Isso exige cuidado explícito com logs: a lista de `redactPaths` do Pino hoje cobre `password`, `passwordHash`, `token` e `accessToken`, **mas não cobre `secret`** `[CODIGO src/shared/logger/index.ts]`. Manter a configuração como está registraria secrets em log — a lista precisa ser estendida na implementação.
- **Custo de calendário:** a revisão de segurança da geração de secret e do HMAC é pré-requisito de deploy e reserva pelo menos dois dias úteis no fim do cronograma `[TRANSCRICAO 09:46 Sofia]`.

## 6. Decisões relacionadas

- Implementa: [RFC — Proposta técnica](../RFC.md#3-proposta-técnica)
- Detalhado em: [FDD — Contratos públicos](../FDD.md#5-contratos-públicos) e [FDD — Matriz de erros](../FDD.md#6-matriz-de-erros-previstos)
- Relacionados: [ADR-005](./ADR-005-entrega-at-least-once-com-event-id.md) (demais headers do envio), [ADR-007](./ADR-007-reuso-dos-padroes-existentes.md) (validação via Zod e `requireRole`)
- Substitui: nenhum
