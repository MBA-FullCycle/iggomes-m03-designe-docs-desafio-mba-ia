---
name: readme-processo-generator
description: Gera o README de processo (README.md na raiz do repositório) que documenta como um pacote de documentação técnica (PRD, RFC, FDD, ADRs) foi produzido com apoio de IA — ferramentas usadas, workflow adotado, prompts customizados e iterações de refinamento reais. Use sempre que o usuário pedir para gerar, escrever, atualizar ou revisar o "README do processo", o README.md da raiz, quiser documentar como usou IA para produzir a documentação, listar ferramentas e prompts de IA usados no desafio, ou registrar as iterações e ajustes feitos durante a produção — mesmo que não peça explicitamente um "README". Diferente do prd-generator e do rfc-generator, este documento não nasce de uma transcrição de reunião: a fonte é o histórico real de trabalho desta sessão e o relato do usuário sobre o que aconteceu fora dela (ex.: ChatGPT, Copilot). Nunca invente ferramenta, prompt ou iteração — pergunte o que não for observável na sessão.
---

# Geração do README de processo

## O que este documento é — e por que ele é diferente dos outros

O PRD, o RFC e o FDD documentam a **feature**: o que ela faz e como foi projetada. Este README
documenta o **processo**: como a documentação em si foi produzida, com quais ferramentas, com
quais prompts, e com que revisões. É a diferença entre o produto do trabalho e o relato do
trabalho.

Isso muda a natureza da fonte. PRD e RFC rastreiam para `TRANSCRICAO.md` e para o código — fontes
estáticas que você pode ler inteiras. Este README rastreia para algo mais escorregadio: o que
realmente aconteceu na produção dos outros documentos. Parte disso é observável nesta própria
sessão de trabalho (se o PRD e o RFC foram gerados aqui, os prompts que os produziram estão no
histórico da conversa). Parte não é observável de jeito nenhum — se o usuário usou ChatGPT ou
Copilot em outra janela, você não tem acesso a isso a menos que ele conte.

## A regra inegociável: nada sem origem — aplicada ao processo, não ao conteúdo

O risco de invenção aqui é diferente do PRD/RFC, mas não é menor. Lá, o risco é inventar um
requisito ou uma alternativa. Aqui, o risco é inventar um prompt "de exemplo" que soa bem, uma
ferramenta que não foi usada, ou uma iteração genérica que nunca aconteceu de fato — coisas fáceis
de fabricar porque ninguém vai conferir contra uma transcrição.

Toda afirmação sobre o processo vem de uma destas três fontes, nesta ordem de preferência:

1. **O histórico observável desta sessão** — se você gerou o PRD, o RFC ou uma skill de apoio
   nesta própria conversa, o prompt real que pediu isso está ali. Use o texto literal, não uma
   paráfrase.
2. **O relato direto do usuário** — para tudo que aconteceu fora desta sessão (outra ferramenta,
   outra conversa, decisões tomadas antes de você entrar). Pergunte especificamente: quais
   ferramentas, para que etapa, quais prompts (peça para colar o texto literal quando possível),
   quais ajustes concretos.
3. **Evidência indireta do repositório** — histórico do `git log`, arquivos que existem em
   `docs/`, skills presentes em `.claude/skills/`. Isso serve para reconstruir uma linha do tempo
   e para não esquecer uma etapa, mas não substitui perguntar o "porquê" e o "como" ao usuário.

O que nunca é aceitável: escrever um prompt de exemplo plausível e apresentá-lo como se tivesse
sido realmente enviado, listar uma ferramenta de IA "porque é comum usar", ou descrever uma
iteração genérica ("revisamos e ajustamos") sem o ajuste concreto que ela representa.

## Fluxo

### 1. Reconstruir a linha do tempo real

Antes de perguntar qualquer coisa ao usuário, junte o que já é observável:

- Releia esta conversa (ou as partes relevantes dela) em busca de prompts reais que produziram
  PRD, RFC, FDD, ADRs, ou skills de apoio a essa produção.
- Rode `git log --oneline` no repositório para ver a ordem real em que os documentos foram
  commitados — isso costuma revelar a sequência real, que pode diferir do workflow "ideal" do
  material de referência.
- Liste o que existe hoje em `docs/` e `.claude/skills/` — cada skill de apoio (como um gerador
  de PRD ou de RFC dedicado) é, ela mesma, uma etapa real do workflow e vale a pena aparecer na
  seção 3.

Isso reduz drasticamente o que você precisa perguntar, e o que sobrar são perguntas que só o
usuário pode responder.

### 2. Perguntar o que não é observável

Pergunte de forma objetiva, cobrindo o que falta depois do passo 1:

- Quais ferramentas de IA foram usadas fora desta sessão (ChatGPT, Copilot, outra) e para qual
  etapa especificamente.
- Se algum prompt relevante foi enviado fora desta sessão, peça o texto literal — se o usuário só
  lembra a ideia geral, registre isso como aproximação e diga no relatório final que não é o
  texto exato.
- Quais ajustes concretos foram feitos entre a primeira versão de um documento e a versão final,
  e o que motivou cada um.

Não pergunte o que já ficou claro no passo 1.

### 3. Escrever o README

Use `assets/template-README.md` como esqueleto e `references/secoes-do-readme.md` para saber o
que pertence a cada seção, com exemplos de conteúdo forte e fraco. As seis seções obrigatórias
são:

1. Sobre o desafio
2. Ferramentas de IA utilizadas
3. Workflow adotado
4. Prompts customizados
5. Iterações e ajustes
6. Como navegar a entrega

Este documento **substitui** a descrição original do desafio no `README.md` da raiz do
repositório — não convive com ela. Confirme com o usuário que é isso que ele quer antes de
sobrescrever, se o `README.md` atual ainda for a descrição original do desafio.

O workflow de exemplo do material de referência (ler transcrição → código → PRD → RFC → FDD →
ADRs → Tracker → README → revisão) é um ponto de partida útil, não um roteiro a copiar. Reproduza
a ordem real reconstruída no passo 1, mesmo que ela inclua etapas que o exemplo não prevê (como
a criação de uma skill de apoio antes de produzir o primeiro documento).

### 4. Auditar antes de entregar

Releia o documento pronto contra este piso de qualidade. Se algum item falhar, corrija antes
de responder ao usuário:

- [ ] Arquivo contém as seis seções obrigatórias
- [ ] Lista no mínimo 1 ferramenta de IA, cada uma com a etapa em que foi usada
- [ ] Mostra no mínimo 2 prompts customizados, em blocos de código, com texto literal (ou
      marcados como aproximação, se o usuário só lembrou a ideia)
- [ ] Descreve no mínimo 2 iterações ou ajustes concretos — com o "antes" e o "depois"
      específicos, não genéricos
- [ ] "Como navegar a entrega" lista o caminho real de cada documento do pacote, incluindo os que
      ainda não foram criados (marcados como tal)
- [ ] Nenhum prompt, ferramenta ou iteração foi inventado; tudo tem origem na sessão, no relato do
      usuário, ou no histórico do repositório

## Antipadrões

| Antipadrão | Por que dói | O que fazer |
| --- | --- | --- |
| Prompt "de exemplo" que soa bem mas não foi realmente enviado | Quebra a auditabilidade — é o mesmo tipo de invenção que o PRD proíbe para requisitos | Use o texto literal da sessão ou peça ao usuário; nunca reconstrua de memória |
| Copiar o workflow de 9 passos do material de referência sem checar | Documenta um processo que talvez não seja o que aconteceu | Reconstrua a ordem real via `git log` e histórico da sessão |
| Listar ferramenta de IA sem dizer para que foi usada | Não ajuda quem quer repetir o processo | Uma frase por ferramenta, nomeando a etapa |
| Iteração genérica ("revisamos e melhoramos") | Indistinguível de não ter revisado nada | Antes e depois concretos, com o que mudou de fato |
| Manter a descrição original do desafio ao lado do processo | O README de processo substitui, não convive com o enunciado | Confirme e sobrescreva, referenciando o desafio em 1 parágrafo |
| Listar ADR ou documento como existente quando é só um stub | Quebra a confiança de quem navega pela lista | Marque explicitamente o que ainda não foi criado |

## Ao entregar

Diga onde o arquivo foi escrito e reporte, em poucas linhas:

- quais ferramentas, prompts e iterações vieram do histórico observável desta sessão versus do
  relato do usuário;
- se algum prompt foi registrado como aproximação por falta do texto literal;
- quais seções ficaram mais fracas por falta de informação real disponível, e o que resolveria
  isso (geralmente, uma pergunta específica ao usuário).

Não afirme ter documentado o processo completo se partes relevantes dele não foram observadas nem
relatadas.

## Arquivos de apoio

- `references/secoes-do-readme.md` — o que pertence a cada uma das 6 seções, com exemplos de
  conteúdo forte e fraco. Leia antes de redigir.
- `assets/template-README.md` — esqueleto Markdown pronto.
