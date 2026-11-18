# Da Reunião ao Documento: Design Docs Gerados por IA

Documentação do processo de produção do pacote de design docs da feature **Sistema de Webhooks de Notificação de Pedidos** do OMS.

> O enunciado original do desafio está no [repositório base](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).

## Sobre o desafio

O desafio parte de uma situação comum: uma decisão técnica foi tomada numa reunião de 55 minutos (tech lead, PM, dois engenheiros e segurança), mas nada foi registrado além da transcrição (`TRANSCRICAO.md`). A tarefa foi transformar essa conversa e o código de um Order Management System já existente (Node.js, TypeScript, Prisma, MySQL) em documentação acionável: PRD, RFC, FDD, ADRs e um Tracker de rastreabilidade, sem mexer no código da aplicação.

A parte difícil não é escrever, é separar o que foi decidido do que foi só comentado, descartado ou adiado, e provar a origem de cada afirmação. Por isso toda informação precisa apontar para uma fala (`[hh:mm] Nome`) ou para um arquivo real do repositório, e o que não tem origem precisa ser declarado como proposta, não como fato.

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
| --- | --- |
| **Claude Code** (CLI, modelo Claude Sonnet) | Ferramenta principal. Leu `TRANSCRICAO.md` e o código em `src/` e `prisma/`, gerou e editou os Markdown em `docs/` e fez as varreduras de consistência (caminhos de arquivo, links entre documentos, cobertura do Tracker). |

A revisão crítica, a decisão do que entra ou não nos documentos e a aprovação final foram minhas. A IA propôs, eu conferi contra a transcrição e o código.

## Workflow adotado

Segui a ordem sugerida no enunciado, porque cada documento serve de base para o seguinte:

1. **Contextualização.** Pedi à IA um mapa do código (módulos, máquina de estados, transação do `changeStatus`, hierarquia de erros, `requireRole`, logger Pino) e uma leitura da transcrição separando decisões fechadas, requisitos, restrições, itens descartados e itens adiados.
2. **ADRs primeiro** (`docs/adrs/`). Seis ADRs, um por decisão principal da reunião: Outbox, worker em polling, retry/DLQ, HMAC, at-least-once com `X-Event-Id` e reuso dos padrões do projeto.
3. **RFC.** Consolidou a proposta em cima dos ADRs, com alternativas descartadas, questões em aberto e links para os ADRs.
4. **FDD.** Desenho técnico detalhado: modelo de dados, fluxos, contratos HTTP, matriz de erros `WEBHOOK_*`, resiliência, observabilidade e a seção "Integração com o sistema existente".
5. **PRD.** Por último entre os grandes documentos, como consolidação de alto nível.
6. **Tracker.** Montado varrendo os documentos prontos, item por item, ligando cada um à fala ou ao arquivo de origem.
7. **README do processo** (este arquivo), por último.
8. **Revisão final** pela checklist de critérios de aceite.

A interação foi sempre em duas etapas por documento: (a) um prompt dirigido, com o papel do documento, a "altura" dele e o que não deveria entrar; (b) uma rodada de revisão crítica minha, voltando à IA com correções pontuais em vez de pedir tudo de novo.

## Prompts customizados

### 1. Filtragem da transcrição (o que entra, o que não entra)

Usei antes de escrever qualquer documento, para evitar que ideias descartadas virassem requisito.

```text
Leia TRANSCRICAO.md inteira. Não resuma. Classifique cada ponto técnico em:
1. DECISÃO FECHADA (alguém com autoridade confirmou)
2. REQUISITO FUNCIONAL explícito
3. RESTRIÇÃO / REQUISITO NÃO FUNCIONAL
4. GANCHO COM O CÓDIGO EXISTENTE (cite o arquivo)
5. DESCARTADO (foi proposto e rejeitado)
6. ADIADO / EM ABERTO (ninguém decidiu ou ficou para depois)
7. DETALHE SECUNDÁRIO

Regras:
- Para cada item, traga o timestamp e o nome do falante no formato [hh:mm] Nome
  e a frase literal que sustenta a classificação.
- Se uma decisão foi tomada em um momento e revista depois, registre as duas
  falas e diga qual vale.
- Se algo parece decisão mas ninguém confirmou, classifique como EM ABERTO.
- Não invente nada que não esteja na transcrição. Se faltar informação, escreva
  "NÃO DEFINIDO NA REUNIÃO".
```

### 2. Geração do FDD com ancoragem no código

Usei para o documento mais técnico, com exigência explícita de separar fato de proposta.

```text
Escreva docs/FDD.md. Ele responde "como construir, em detalhe", em nível de
implementação. Não repita o RFC nem os ADRs: referencie-os por link.

Entradas: TRANSCRICAO.md, docs/adrs/*.md, docs/RFC.md e o código em src/ e prisma/.

Obrigatório:
- Fluxos: criação do evento na outbox (dentro da transação de changeStatus),
  processamento pelo worker, retry e DLQ.
- Contratos HTTP: pelo menos 4 endpoints com request, response, headers e status codes.
- Matriz de erros com prefixo WEBHOOK_, usando AppError e as classes de
  src/shared/errors/ (não crie hierarquia nova).
- Seção "Integração com o sistema existente" com caminhos de arquivo reais,
  dizendo o que muda em cada um. Antes de citar um caminho, confirme que ele existe.

Regras anti-alucinação:
- Todo detalhe que NÃO veio da reunião ou do código deve ser marcado como
  (proposta FDD) e listado numa seção "Premissas e pontos a validar", com
  quem deveria decidir.
- Se a transcrição for ambígua (ex.: "5 tentativas"), não escolha em silêncio:
  explicite as interpretações possíveis, escolha uma e justifique.
- Não inclua nada que a reunião descartou ou adiou.
```

## Iterações e ajustes

O processo teve **5 ciclos principais** de geração, revisão e correção (ADRs, RFC, FDD, PRD e Tracker), mais uma passada final pela checklist. Os principais problemas encontrados e como foram resolvidos:

1. **Detalhes inventados apresentados como decisão.** A primeira versão do FDD tratava como definidos pontos que a reunião não fechou: formato do header `X-Signature`, tamanho de lote do worker, índice composto, recuperação de eventos `PROCESSING` órfãos na partida, `redirect: 'manual'`, truncamento da resposta. Corrigi marcando esses itens como *(proposta FDD)* e criei a seção 13 do FDD, "Premissas de implementação e pontos a validar" (P-01 a P-10), indicando quem deve decidir cada um.
2. **Ambiguidade resolvida em silêncio.** A transcrição fala em "5 tentativas" de retry, mas o backoff citado só soma cerca de 15 horas se houver a tentativa inicial mais 5 retries. A IA escolheu uma leitura sem avisar. Registrei a ambiguidade como P-01 e deixei a confirmação com Larissa e Diego.
3. **Segredo armazenado.** A IA sugeriu guardar a secret do endpoint com hash. Isso é incompatível com HMAC, que precisa do valor utilizável para assinar. Corrigi o desenho e deixei como ponto para Sofia avaliar criptografia em repouso (P-04).
4. **Itens fora de escopo virando requisito.** Rascunhos do PRD incluíam coisas que a reunião adiou ou descartou (aviso por e-mail, rate limiting de saída, dashboard, arquivamento). Movi todos para "Fora de escopo" no PRD, com a fala de origem, e para "Questões em aberto" no RFC.
5. **SSRF.** Ao revisar o FDD, a IA propôs proteção contra SSRF como se tivesse sido discutida. Não foi. Mantive como ponto para a revisão de segurança (P-09) e o listei em "Itens sem origem rastreável" no Tracker, em vez de atribuí-lo a uma fala.
6. **Tracker contaminando a rastreabilidade.** A primeira versão atribuía falas aproximadas a itens que na verdade eram propostas. Passei a exigir timestamp e falante verificáveis em cada linha e criei a seção "Itens sem origem rastreável", para o Tracker nunca atribuir a alguém algo que ninguém disse.
7. **Sobreposição entre RFC e FDD.** O RFC saiu com nível de detalhe de implementação. Reduzi o RFC a decisões, alternativas e questões em aberto, deixando o detalhamento só no FDD.

## Como navegar a entrega

Ordem sugerida de leitura:

| # | Arquivo | O que responde |
| --- | --- | --- |
| 1 | [`docs/PRD.md`](docs/PRD.md) | Por que e o quê: problema, escopo, requisitos, métricas, riscos |
| 2 | [`docs/RFC.md`](docs/RFC.md) | Proposta técnica em alto nível, alternativas e questões em aberto |
| 3 | [`docs/adrs/README.md`](docs/adrs/README.md) | Índice das decisões, com um arquivo por ADR (`ADR-001` a `ADR-006`) |
| 4 | [`docs/FDD.md`](docs/FDD.md) | Como construir: modelo de dados, fluxos, contratos, erros, integração com o código |
| 5 | [`docs/TRACKER.md`](docs/TRACKER.md) | De onde veio cada item: fala da transcrição ou arquivo do código |

Para quem for implementar, a leitura mais direta é RFC, ADRs e FDD. Para auditar a fidelidade ao que foi discutido, use o Tracker junto com [`TRANSCRICAO.md`](TRANSCRICAO.md).

O código da aplicação (`src/`, `prisma/`, `tests/`) e a transcrição não foram alterados.
