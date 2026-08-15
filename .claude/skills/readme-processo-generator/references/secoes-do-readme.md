# O que pertence a cada seção do README de processo

Referência de redação. Cada seção traz o propósito, o que precisa estar lá e um par de exemplos
de conteúdo forte e fraco.

> Ao contrário dos exemplos do `prd-generator` e do `rfc-generator`, aqui não há domínio fictício
> para ilustrar — o conteúdo bom e o conteúdo ruim se distinguem pela mesma coisa em toda seção:
> se aconteceu de verdade ou se foi reconstruído de memória para preencher espaço. Os exemplos
> abaixo usam trechos genéricos só para mostrar a *forma* esperada.

## Índice

1. [Sobre o desafio](#1-sobre-o-desafio)
2. [Ferramentas de IA utilizadas](#2-ferramentas-de-ia-utilizadas)
3. [Workflow adotado](#3-workflow-adotado)
4. [Prompts customizados](#4-prompts-customizados)
5. [Iterações e ajustes](#5-iterações-e-ajustes)
6. [Como navegar a entrega](#6-como-navegar-a-entrega)

---

## 1. Sobre o desafio

**Propósito:** orientar quem abre o repositório pela primeira vez — o que foi pedido e o que foi
entregue — antes de ele mergulhar nos documentos.

Um parágrafo basta. Não repita a descrição inteira do desafio original; resuma o suficiente para
dar contexto e aponte para onde o pacote de documentos vive.

- ✅ "Este repositório documenta o design de um Sistema de Webhooks de Notificação de Pedidos,
  produzido a partir da transcrição de uma reunião técnica (`TRANSCRICAO.md`) e do código
  existente do OMS. O pacote entregue cobre PRD, RFC, FDD, ADRs e um tracker de rastreabilidade."
- ❌ Colar a descrição completa do desafio original sem resumir — o README de processo não é o
  enunciado do desafio, é o relato de como ele foi resolvido.

## 2. Ferramentas de IA utilizadas

**Propósito:** dizer quais ferramentas de IA entraram no processo e para que exatamente, não só
listar nomes.

Cada ferramenta precisa de uma frase dizendo a etapa em que ela atuou. "Usada para tudo" não
diferencia nada e não ajuda quem for repetir o processo.

- ✅ "**Claude Code** — usado para ler o código existente (`src/modules/orders`), identificar os
  pontos de integração da nova feature e gerar o RFC e o FDD com base na transcrição e no
  código."
- ❌ "**ChatGPT** — ajudou bastante no processo." (não diz em qual etapa nem para produzir o quê)

Nunca liste uma ferramenta que não foi de fato usada. Se você não tem certeza se algo contou como
"ferramenta de IA utilizada" (por exemplo, autocomplete do editor), pergunte ao usuário antes de
incluir.

## 3. Workflow adotado

**Propósito:** relatar a sequência real de etapas seguida na produção da documentação.

O material do curso mostra um workflow de exemplo (analisar transcrição → analisar código →
PRD → RFC → FDD → ADRs → Tracker → README → revisão final). É um bom ponto de partida, mas é um
exemplo, não uma prescrição — confirme com o usuário se foi essa a ordem real ou se ela foi
diferente (por exemplo, ADRs escritos em paralelo ao RFC, ou uma etapa de setup de skills/ferramentas
antes de produzir qualquer documento) antes de reproduzir a lista como se fosse a verdade do
projeto.

- ✅ "1. Leitura da transcrição e do código. 2. Criação de skills dedicadas para gerar PRD e RFC
  de forma consistente. 3. Produção do PRD. 4. Teste da skill de RFC contra a transcrição real,
  com ajustes. 5. Produção do RFC. [...]" — ordem específica, com uma etapa (criação de skills)
  que não está no exemplo do curso porque é real deste projeto.
- ❌ Copiar as nove etapas do material de referência palavra por palavra sem checar se bate com o
  que aconteceu.

## 4. Prompts customizados

**Propósito:** mostrar, literalmente, o que foi pedido à IA — para que o processo seja auditável
e repetível por outra pessoa.

Esta é a seção com maior risco de invenção: é tentador escrever um prompt "de exemplo" que soa
bem em vez de recuperar o que foi realmente digitado. Um prompt reconstruído de memória, mesmo
que capture a intenção certa, não é o mesmo documento — perde detalhes que na hora fizeram
diferença (uma restrição específica, uma seção que foi pedida explicitamente).

Fontes válidas, em ordem de preferência:
1. O texto literal de uma mensagem desta própria sessão de trabalho, se a produção aconteceu
   aqui.
2. O texto que o usuário cola de uma conversa em outra ferramenta (ChatGPT, etc.).
3. Na ausência das duas anteriores, pergunte ao usuário — não improvise um prompt plausível.

- ✅ Colar o prompt exatamente como foi enviado, em bloco de código, com uma linha antes dizendo
  o que ele produziu.
- ❌ "Prompt: pedimos para a IA gerar um PRD bem completo e detalhado." (paráfrase vaga, não é o
  prompt, é um resumo do prompt)

## 5. Iterações e ajustes

**Propósito:** mostrar que o resultado final não foi aceito de primeira — que houve revisão
crítica, o que é exatamente o que o desafio pede do "papel de maestro".

Cada iteração precisa de um antes e depois concretos: o que a primeira versão tinha de errado,
específico o bastante para ser verificável, e o ajuste que resolveu. Iterações genéricas
("revisamos e melhoramos a redação") não diferenciam de não ter revisado nada.

Um bom lugar para encontrar material real para esta seção é o histórico de trabalho em si: se uma
skill foi testada contra uma fonte real e o teste revelou um problema que foi corrigido antes da
entrega, isso é uma iteração legítima e concreta — muitas vezes melhor que uma reconstrução vaga
de "revisamos o PRD".

- ✅ "Na primeira versão do RFC, a seção de alternativas citava apenas uma opção descartada. Ao
  reler a transcrição inteira (em vez de só o resumo final), foram encontradas mais três
  alternativas discutidas e rejeitadas, que foram adicionadas com o trade-off de cada uma."
- ❌ "Revisamos todos os documentos e corrigimos o que precisava ser corrigido." (não diz o quê,
  não é verificável)

## 6. Como navegar a entrega

**Propósito:** dar ao leitor uma ordem de leitura e o caminho real de cada arquivo, para que ele
não precise adivinhar por onde começar.

Liste apenas os documentos que de fato existem no repositório, com o caminho real. Se um
documento do pacote (por exemplo, os ADRs) ainda não foi criado no momento em que este README é
escrito, não o omita — registre o caminho esperado e o estado atual, para que a lista continue
confiável.

- ✅ "4. **ADRs** — registro das decisões arquiteturais. `docs/adrs/` *(ainda não criado nesta
  entrega — ver Questões em aberto do RFC)*"
- ❌ Listar `docs/adrs/ADR-001-....md` como se existisse quando o diretório só tem o README de
  convenção.
