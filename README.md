# Da Reunião ao Documento — Pacote de Design Docs gerado com IA

> Este README documenta **o processo de produção** da entrega.
> O enunciado original do desafio está preservado em [`docs/refinamento/history.md`](docs/refinamento/history.md).

---

## Sobre o desafio

A ideia aqui é simples de explicar e chata de fazer direito: existe uma call de 55 minutos gravada, cinco pessoas discutindo como construir um sistema de webhooks para notificar clientes B2B quando o status de um pedido muda. A reunião acabou, todo mundo saiu com a decisão na cabeça, e não sobrou nada além da transcrição literal. A tarefa é transformar isso em um pacote de documentação — PRD, RFC, FDD, ADRs e um tracker de rastreabilidade — bom o suficiente para um dev pegar e começar a codar sem precisar perguntar nada.

O ponto que faz o desafio ser um desafio não é escrever documento bonito. É que a IA escreve documento bonito com uma facilidade perigosa: se você pedir "gere um FDD de sistema de webhooks", ela devolve algo bem formatado, plausível, com números redondos e trade-offs genéricos — e metade daquilo não foi discutido na reunião nenhuma. O trabalho de verdade é o contrário disso: filtrar. Da transcrição sai o que foi **decidido**, o que foi **adiado** e o que foi **explicitamente descartado**, e essas três coisas vão para lugares diferentes do pacote. Diego sugeriu retry indefinido e o grupo fechou em 5 tentativas — então 5 é o requisito e o retry indefinido é alternativa descartada, não um detalhe a ser esquecido. Bruno propôs 3 tentativas e perdeu o argumento; isso também tem lugar. Identificar o que **não** entra vale tanto quanto identificar o que entra.

O segundo ponto é que a documentação não pode falar sozinha: ela tem que se conectar com o código que já existe. O repositório é um Order Management System em Node/TypeScript com Prisma e MySQL, com máquina de estados de pedido funcionando e zero mecanismo de notificação — o buraco é proposital, é exatamente o que a feature preenche. Então o FDD não pode dizer "integre com o service de pedidos"; ele precisa dizer que a escrita entra dentro do `prisma.$transaction` de `OrderService.changeStatus`, depois do ajuste de estoque, e que se ela falhar a transação inteira cai. Isso só sai abrindo o código.

---

## Ferramentas de IA utilizadas

| Ferramenta | Papel na produção |
| --- | --- |
| **Claude Code (Opus 5)** | Única ferramenta usada. Leitura da transcrição e do código-base, redação de todos os documentos do pacote, verificação cruzada das citações e correção das inconsistências entre documentos. |
| **Skills de projeto em [`.claude/skills/`](.claude/skills/)** | Seis skills dedicadas — uma por artefato — que carregam as regras de redação de cada documento no momento em que ele vai ser escrito. São elas que impõem o piso de qualidade (o que pertence a cada seção, o que é altura errada, quantas alternativas reais são exigidas, quando marcar premissa). |

As seis skills e o que cada uma governa:

| Skill | Governa |
| --- | --- |
| `adr-generator` | Formato MADR, um ADR por decisão, mínimo de 1 alternativa real, consequências obrigatoriamente com custo |
| `rfc-generator` | Altura de arquitetura, orçamento de 2–4 páginas, mínimo de 2 alternativas realmente debatidas |
| `fdd-generator` | Altura de implementação, código como fonte **primária**, mínimo de 4 arquivos reais na seção de integração |
| `prd-generator` | Altura de produto, teste de fronteira PRD × RFC, mínimo de 8 requisitos funcionais |
| `tracker-generator` | Resolução de origem em cadeia até `TRANSCRICAO` ou `CODIGO`, pisos de cobertura |
| `readme-processo-generator` | Este documento — processo real observado, nunca processo idealizado |

---

## Workflow adotado

A ordem **não** foi a intuitiva (PRD → RFC → FDD). Foi a inversa, e por um motivo: as decisões são o esqueleto. Só depois de saber exatamente o que foi decidido e por quê é que dá para escrever uma proposta coerente, e só com a proposta fechada é que o detalhamento não vira invenção.

**1. Leitura das fontes antes de escrever qualquer coisa.**
Transcrição inteira, do `[09:00]` ao `[09:53]` — inclusive o bloco final, depois que Marcos e Sofia saem da call, que é onde duas decisões reais aparecem. Depois, o código: `order.service.ts`, a hierarquia de erros, os middlewares, o logger, o `schema.prisma`, o `package.json`.

**2. ADRs primeiro** — 7 arquivos em `docs/adrs/`.
Cada decisão da reunião virou um arquivo, com a alternativa que perdeu e o custo assumido.

**3. RFC** — consolidação da proposta em cima das decisões já fechadas.
As alternativas descartadas e as questões em aberto têm lugar natural aqui; cada uma aponta para o ADR que a defende em detalhe, para não reargumentar.

**4. FDD** — o detalhamento, com o código aberto ao lado.
Modelo de dados, 7 fluxos, 8 contratos, matriz de erros `WEBHOOK_*`, e a seção obrigatória de integração com 12 arquivos reais.

**5. PRD** — por último entre os grandes, como consolidação.
Com RFC, FDD e ADRs prontos, o PRD vira tradução: cada decisão técnica descrita pela **consequência para o cliente**, não pelo mecanismo.

**6. Tracker** — varredura dos documentos prontos.
236 linhas ligando cada item à sua origem, mais uma segunda tabela isolando o que **não** é rastreável.

**7. Verificação automatizada e revisão final** contra a checklist de critérios de aceite.

---

## Prompts customizados

### Prompt 1 — invocação dirigida da skill de ADRs

O erro clássico aqui é pedir "gere os ADRs" e deixar a IA escolher o que é decisão. O prompt nomeia as decisões extraídas da leitura da transcrição e fixa as fontes, deixando para a skill apenas a redação:

```
Gerar 6 ADRs em docs/adrs/ a partir de TRANSCRICAO.md e do código.
Decisões: outbox MySQL, retry+backoff+DLQ, HMAC-SHA256 secret por endpoint
com rotação, at-least-once com X-Event-Id, worker separado em polling 2s,
reuso dos padrões existentes.
```

Resultado: pedi 6, entreguei 7 — a releitura do fecho da call revelou uma sétima decisão real, com alternativa debatida (ver "Iterações", item 4).

### Prompt 2 — FDD com a exigência de código real explícita

O que impede o FDD de virar prosa genérica é obrigar a seção de integração a apontar para arquivos que existem de fato:

```
Gerar docs/FDD.md a partir do RFC, dos 7 ADRs, da TRANSCRICAO.md e do código
existente. Seção obrigatória "Integração com o sistema existente" com 4+
arquivos reais.
```

### Prompt 3 — tracker com os pisos numéricos no próprio pedido

```
Gerar docs/TRACKER.md consolidando PRD, RFC, FDD e os 7 ADRs.
Formato obrigatório: ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização.
Mínimo 70% TRANSCRICAO com [hh:mm] Nome e 5+ linhas CODIGO.
```

### Prompt 4 — o mais útil de todos: verificar em vez de confiar

Este não é um prompt de geração, é de auditoria. Com 369 citações espalhadas por 11 arquivos, conferência manual não é confiável — então a conferência virou script, comparando cada `[TRANSCRICAO hh:mm Nome]` contra o arquivo real, **inclusive o nome do falante**:

```python
truth = {}
for line in open('TRANSCRICAO.md'):
    m = re.match(r'\[(\d\d:\d\d)\]\s+(\*?)([A-Za-zÀ-ú]+)', line)
    if m and not m.group(2):                    # ignora marcações tipo *Diego entrou*
        truth.setdefault(m.group(1), set()).add(m.group(3))

for d in glob.glob('docs/*.md') + glob.glob('docs/adrs/*.md'):
    for m in re.finditer(r'TRANSCRICAO (\d\d:\d\d)(?: ([A-Za-zÀ-ú]+))?', open(d).read()):
        ts, name = m.group(1), m.group(2)
        if ts not in truth:
            print(d, ts, 'TIMESTAMP INEXISTENTE')
        elif name and name not in truth[ts]:
            print(d, ts, name, f'falante errado; real={sorted(truth[ts])}')
```

Um script gêmeo extraiu todo caminho `src/…`, `prisma/…`, `tests/…` citado nos documentos e checou se o arquivo existe no repositório.

---

## Iterações e ajustes

Foram **7 ciclos** de geração → revisão crítica → correção. Os concretos:

**1. Âncoras cruzadas quebradas entre documentos.**
Os ADRs foram escritos primeiro e já linkavam para seções do RFC e do FDD que ainda não existiam — apostei na numeração. Errei: escrevi `../RFC.md#4-proposta-técnica`, mas no RFC pronto "Proposta técnica" ficou como seção **3**; no FDD, "Integração com o sistema existente" ficou na **12** (eu tinha chutado 7) e "Estratégias de resiliência" na **7** (chutei 8). Três âncoras mortas espalhadas por 7 arquivos, corrigidas em lote assim que cada documento de destino ficou pronto. Um link morto é pior que link nenhum: parece verificado e não é.

**2. Contradição entre RFC e FDD sobre o modelo de dados.**
O RFC afirmava "migration nova com **três** tabelas (configuração de webhook, outbox e dead letter)". Ao modelar o FDD, o histórico de entregas que Marcos pediu em `[09:34]` ("payload, response, tempo de resposta") não cabia em nenhuma das três — virou uma quarta tabela, `webhook_deliveries`. O RFC foi corrigido para quatro. Documento que contradiz outro documento do mesmo pacote é exatamente o tipo de erro que passa despercebido em leitura sequencial.

**3. Números do tracker estimados de cabeça — e errados.**
Escrevi a tabela de cobertura com "199 linhas, 77,4% TRANSCRICAO, 45 CODIGO". Ao contar por script: **236 linhas, 80,1%, 47**. Nenhum dos três números estava certo. Foram substituídos pelos valores contados. Afirmar percentual de cobertura sem contar é o mesmo vício que o tracker existe para combater.

**4. Um sétimo ADR que a lista do desafio não previa.**
O enunciado enumera 6 decisões principais. A releitura do trecho final da call — depois que Marcos e Sofia saem, entre `[09:50]` e `[09:53]` — mostrou Bruno perguntando se a outbox guarda o payload renderizado ou só o `order_id`, com Larissa e Diego fechando em "snapshot na inserção". É decisão fechada, com alternativa real debatida e consequência concreta (evento entregue 15h depois carrega dado velho). Virou o [ADR-006](docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md). Conversa de corredor depois que parte da call saiu é o lugar mais fácil de perder material real.

**5. Uma regra do enunciado que colidia com uma decisão da reunião.**
A primeira formulação do FDD colocava a validação do teto de 64KB na **inserção** do evento. Só que o [ADR-001](docs/adrs/ADR-001-outbox-no-mysql.md) estabelece que falha na inserção derruba a transação inteira — ou seja, um pedido deixaria de mudar de status porque o payload de notificação ficou grande. Absurdo operacional. A transcrição desfaz: Sofia diz "se por algum motivo o evento tiver 500KB, **a gente não envia**" `[09:23]` — a validação é no envio, e o evento vai para retry/DLQ. Adotei essa leitura e registrei a ambiguidade como questão em aberto (FDD Q-01) em vez de escolher em silêncio.

**6. Um achado que não está na transcrição, só no código.**
A reunião decidiu secret por endpoint, mas ninguém falou de log. Lendo `src/shared/logger/index.ts`, a lista de `redactPaths` do Pino cobre `password`, `passwordHash`, `token` e `accessToken` — e **não cobre `secret`**. Ou seja: da forma como está hoje, qualquer log do objeto de endpoint gravaria a credencial do cliente em texto. Não é hipótese, é o estado atual do arquivo. Virou risco no RFC (RSC-05), risco no FDD (RSC-06) e consequência negativa no [ADR-004](docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md). Foi o melhor argumento a favor de ler o código em vez de só a transcrição.

**7. Alternativas plausíveis que eu quase apresentei como debatidas.**
Três alternativas fortes não saíram da reunião: mTLS no lugar de HMAC, deduplicação do nosso lado, e hierarquia de erros própria do módulo. Todas soam bem e passariam despercebidas. Ficaram marcadas com ⚠️ **"não discutida na fonte"** dentro dos ADRs e isoladas na segunda tabela do tracker. Marcar isso é o que separa registro de decisão de teatro de decisão.

---

## Como navegar a entrega

Ordem sugerida de leitura — de cima para baixo em altura, que é o inverso da ordem em que foram escritos:

| # | Arquivo | O que responde |
| --- | --- | --- |
| 1 | [`docs/PRD.md`](docs/PRD.md) | Por que e o quê — problema, público, escopo, 15 requisitos funcionais, 11 não funcionais, métricas |
| 2 | [`docs/RFC.md`](docs/RFC.md) | Como pretendemos resolver — proposta, 4 alternativas descartadas, 4 questões em aberto |
| 3 | [`docs/adrs/`](docs/adrs/) | Por que decidimos exatamente assim — 7 ADRs, um por decisão ([índice](docs/adrs/README.md)) |
| 4 | [`docs/FDD.md`](docs/FDD.md) | Como construir — modelo de dados, 7 fluxos, 8 contratos, matriz `WEBHOOK_*`, integração com 12 arquivos reais |
| 5 | [`docs/TRACKER.md`](docs/TRACKER.md) | De onde veio cada coisa — 236 linhas rastreadas + 18 itens isolados como não rastreáveis |

Os 7 ADRs, em ordem de dependência:

| ADR | Decisão |
| --- | --- |
| [ADR-001](docs/adrs/ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL |
| [ADR-002](docs/adrs/ADR-002-worker-separado-em-polling.md) | Worker em processo separado, polling de 2s |
| [ADR-003](docs/adrs/ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff (5 tentativas) e DLQ |
| [ADR-004](docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md) | HMAC-SHA256, secret por endpoint, rotação com grace de 24h |
| [ADR-005](docs/adrs/ADR-005-entrega-at-least-once-com-event-id.md) | At-least-once com dedup pelo cliente via `X-Event-Id` |
| [ADR-006](docs/adrs/ADR-006-snapshot-do-payload-na-outbox.md) | Snapshot do payload na inserção da outbox |
| [ADR-007](docs/adrs/ADR-007-reuso-dos-padroes-existentes.md) | Webhooks como módulo convencional, reusando os padrões do projeto |

**Fontes:** [`TRANSCRICAO.md`](TRANSCRICAO.md) (reunião, 55 min) e o código em [`src/`](src/) e [`prisma/`](prisma/).

**Se você tem 5 minutos:** leia o TL;DR do [RFC](docs/RFC.md#1-resumo-executivo-tldr), depois a seção 12 do [FDD](docs/FDD.md#12-integração-com-o-sistema-existente) — é onde a documentação encosta no código de verdade.

---

## Nota sobre o código

A entrega é **puramente documental**. Nada em `src/`, `prisma/`, `tests/` ou nas configurações foi alterado — o código serviu de contexto e referência. Os únicos caminhos citados nos documentos que não existem no repositório são `src/worker.ts` e `src/modules/webhooks/webhook.worker.ts`, arquivos que a feature vai **criar**, nomeados pelo próprio time na reunião (`[09:11] Larissa`, `[09:28] Bruno`), e por isso nunca marcados como código existente.
