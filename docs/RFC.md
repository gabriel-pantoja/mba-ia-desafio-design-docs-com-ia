# RFC-001: Sistema de Webhooks de Notificação de Pedidos

## Metadados

| Campo | Valor |
| --- | --- |
| **Autor** | Gabriel Pantoja (consolidação da reunião técnica conduzida por Larissa) |
| **Status** | Em revisão |
| **Data** | 18-11-2026 |
| **Revisores** | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenheiro Pleno, Pedidos), Diego (Engenheiro Sênior, Plataforma), Sofia (Engenheira de Segurança) |
| **Prazo alvo** | 3 sprints, incluindo ao menos 2 dias úteis de revisão de segurança pela Sofia antes do deploy ([09:46] Larissa, [09:46] Sofia) |
| **Documentos relacionados** | [PRD](./PRD.md) · [FDD](./FDD.md) · [ADRs](./adrs/README.md) · [Tracker](./TRACKER.md) |

## 1. Resumo executivo (TL;DR)

Propomos notificar clientes B2B por **webhooks outbound** sempre que o status de um pedido mudar. O evento é gravado numa tabela `webhook_outbox` **dentro da mesma transação** de `changeStatus` (padrão Outbox no MySQL existente). Um **worker em processo separado** lê a outbox por polling a cada 2 s e faz o POST HTTPS assinado com **HMAC-SHA256** (secret por endpoint). Falhas seguem retry com backoff exponencial (5 tentativas) e depois vão para uma **DLQ** em tabela própria. A entrega é **at-least-once**, e o cliente deduplica por `X-Event-Id`. Nenhuma infraestrutura nova é introduzida, e o módulo reaproveita os padrões já existentes no projeto.

## 2. Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo pediram formalmente para ser avisados quando o status de seus pedidos mudar. Hoje fazem polling em `GET /orders`, o que deixa a integração lenta e cara para eles, e a Atlas sinalizou risco de migrar para um concorrente se a entrega passar do fim do trimestre ([09:00] Marcos). Para esses clientes, "tempo real" significa **menos de 10 segundos** ([09:02] Marcos). O fluxo é só de saída: o sistema envia e o cliente não envia nada de volta ([09:02] Marcos).

O ponto crítico está no código existente. `OrderService.changeStatus` (`src/modules/orders/order.service.ts`) já roda uma transação que atualiza `orders`, insere em `order_status_history` e decrementa o estoque. Uma chamada HTTP ali dentro faria um cliente lento travar mudanças de status de outros pedidos, e um cliente fora do ar forçaria rollback de uma mudança de status válida ([09:04] Bruno). Hoje a aplicação não tem eventos, filas nem webhooks.

## 3. Proposta técnica

Visão geral do fluxo (detalhes de implementação ficam no FDD):

```
changeStatus (API)                       Worker (src/worker.ts)               Cliente
┌──────────────────────────┐             ┌────────────────────────┐
│ TX: update orders        │             │ poll a cada 2s         │
│     insert status_history│             │ lê PENDENTES (batch)   │   POST https
│     ajusta estoque       │──outbox───▶ │ assina HMAC-SHA256     │ ─────────────▶ valida X-Signature
│     publishWebhookEvent  │ (MySQL)     │ timeout 10s            │ ◀───────────── 2xx / erro
└──────────────────────────┘             │ ok → ENTREGUE          │                dedup por X-Event-Id
                                         │ falha → retry/backoff  │
                                         │ 5ª falha → DLQ         │
                                         └────────────────────────┘
```

Os pilares da proposta, cada um com seu ADR:

1. **Outbox transacional.** `publishWebhookEvent(tx, order, fromStatus, toStatus)` insere o evento na mesma transação de `changeStatus`. Se a inserção falhar, tudo sofre rollback. O payload é um snapshot renderizado na inserção, e o filtro de status assinados pelo webhook também é aplicado nessa hora ([ADR-001](./adrs/ADR-001-outbox-no-mysql.md)).
2. **Worker separado em polling.** Novo entry-point `src/worker.ts` (`npm run worker`), com seu próprio `PrismaClient`, mesmo banco. Polling de 2 s atende o requisito de menos de 10 s. Um único worker preserva ordem por `order_id` ([ADR-002](./adrs/ADR-002-worker-separado-em-polling.md)).
3. **Retry e DLQ.** 5 tentativas com backoff de 1 min, 5 min, 30 min, 2 h e 12 h. Depois disso o evento vai para `webhook_dead_letter`. O replay é manual, via endpoint restrito a `ADMIN` ([ADR-003](./adrs/ADR-003-retry-com-backoff-e-dlq.md)).
4. **Autenticação da entrega.** HMAC-SHA256 sobre o corpo, com secret única por endpoint. A rotação mantém a secret anterior válida por 24 h ([ADR-004](./adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md)).
5. **Semântica de entrega.** At-least-once. Cada evento tem um UUID enviado em `X-Event-Id` para deduplicação no cliente ([ADR-005](./adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).
6. **Reuso dos padrões do projeto.** Módulo `src/modules/webhooks` (controller, service, repository, routes, schemas), `AppError` com códigos prefixados `WEBHOOK_`, Pino, error middleware central, `requireRole` e UUIDs como chave ([ADR-006](./adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md)).

**Superfície de API proposta** (contratos completos no FDD): CRUD de configuração de webhook (`POST`, `PATCH`, `DELETE`, `GET` por customer), rotação de secret, `GET /webhooks/:id/deliveries` (histórico de entregas) e `POST /admin/webhooks/dead-letter/:id/replay` (`ADMIN`, com log de quem executou). O CRUD aceita qualquer role autenticada por enquanto ([09:37] Sofia). O `customer_id` vem no body ou no path, não do JWT ([09:32] Larissa).

**Requisitos não funcionais fixados na reunião:** URL obrigatoriamente `https`, validada no schema Zod ([09:23] Sofia); payload de no máximo 64 KB, com erro se ultrapassar ([09:24] Larissa); timeout de 10 s por chamada ([09:42] Diego); headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json` ([09:44] Diego, [09:44] Sofia).

## 4. Alternativas consideradas

| Alternativa | Por que foi descartada (trade-off) |
| --- | --- |
| **Disparo síncrono dentro de `changeStatus`** | Seria o mais simples e entregaria de imediato. Em troca, acopla a latência e a disponibilidade do cliente à transação de pedidos: um cliente lento trava mudanças de outros pedidos, e um cliente offline obrigaria a escolher entre dar rollback numa mudança válida ou perder o evento ([09:04] Bruno, [09:06] Diego). Descartada como "fora de questão". |
| **Fila externa (Redis Streams ou similar)** | Desacopla bem e oferece mais throughput. Em troca, exige subir e operar infraestrutura nova, e o evento deixaria de ser atômico com a transação do MySQL. Para um time pequeno foi considerado overengineering ([09:07] Larissa, [09:07] Diego). |
| **Trigger / notificação do banco para acordar o worker** | Seria mais reativo que polling. O MySQL não tem `LISTEN/NOTIFY`, e uma trigger só executa SQL, sem avisar um processo externo. Avisar o worker exigiria improvisos como escrever em arquivo ou chamar um endpoint ([09:09] Diego). Polling de 2 s já cumpre o requisito de menos de 10 s. |
| **Retry indefinido com backoff** | Nunca perde evento, mas deixa eventos pendurados para sempre quando o cliente some ([09:15] Diego). Optou-se por teto de tentativas e DLQ. |
| **Apenas 3 tentativas** | Mais agressivo, mas curto demais: cobriria só cerca de 30 minutos, e já houve cliente com 2 h de indisponibilidade planejada ([09:16] Diego). |
| **Exactly-once** | Eliminaria duplicatas no cliente. Exigiria coordenação entre os dois lados e muito mais complexidade. Por isso se adotou at-least-once com `X-Event-Id`, como Stripe e GitHub ([09:25] Diego). |

## 5. Questões em aberto

| # | Questão | Situação na reunião | Quem acompanha |
| --- | --- | --- | --- |
| 1 | **Rate limiting de saída por cliente.** Um cliente com 50 pedidos mudando de status em um minuto recebe 50 chamadas. | Fora do escopo desta fase. "Observar e decidir depois" ([09:38] Diego, [09:39] Larissa). | Diego / Larissa |
| 2 | **Notificação por e-mail ao cliente quando o webhook falha repetidamente** (por exemplo, 3 falhas seguidas). | Fora de escopo, talvez numa próxima fase, depois de medir o impacto ([09:37] Larissa). | Marcos |
| 3 | **Escala para múltiplos workers.** O ordering por `order_id` só vale com worker único. | Sem solução agora. As opções citadas foram particionar por `order_id` ou lock pessimista ([09:13] Diego). Registrado como limitação conhecida ([09:13] Larissa). | Diego |
| 4 | **Arquivamento de eventos entregues.** Citado como "depois de 30 dias ou assim". | Declarado fora do escopo da feature ([09:08] Diego). | Diego |
| 5 | **Endurecer permissões do CRUD de webhooks.** Hoje qualquer role autenticada pode configurar. | "Por enquanto sim. Mais pra frente a gente pode endurecer" ([09:37] Sofia). | Sofia |
| 6 | **Dashboard visual para o cliente.** | Descartado para esta fase. Só endpoints, e o painel é projeto do time de frontend ([09:40] Larissa). | Marcos |

Pedimos aos revisores atenção especial às questões 1 e 3, que são as que mais provavelmente mudam o desenho se o volume crescer.

## 6. Impacto e riscos

**Impacto no sistema existente**
- `OrderService.changeStatus` (`src/modules/orders/order.service.ts`) passa a chamar `publishWebhookEvent` dentro da transação. É a única alteração em código de domínio já existente.
- Novas tabelas no MySQL: `webhook_outbox`, `webhook_dead_letter` e a de configuração de endpoints. Um novo processo (`worker`) passa a precisar de deploy e monitoramento.
- Cada mudança de status ganha uma escrita extra na transação, mas só quando algum webhook do customer assina aquele status.

**Riscos**

| Risco | Mitigação |
| --- | --- |
| Falha na inserção da outbox bloqueia a mudança de status. | É intencional, pois a atomicidade é a garantia central ([09:40] Bruno). Testes ponta a ponta cobrem o rollback. |
| Atraso de até 2 s por causa do polling. | Aceito pelo PM e dentro do requisito de menos de 10 s ([09:10] Larissa, [09:10] Marcos). |
| Quebra de ordem se mais de um worker rodar em paralelo. | Operar um único worker e documentar a limitação (questão em aberto 3). |
| Cliente processa o mesmo evento duas vezes. | Dedup por `X-Event-Id`, documentado de forma destacada no portal de desenvolvedores ([09:26] Marcos). |
| Vazamento de secret pelo cliente. | Secret por endpoint, rotação com 24 h de convivência e revisão de segurança antes do deploy ([09:22] Diego, [09:46] Sofia). |
| Worker parado deixa eventos acumulando. | Eventos permanecem seguros na outbox e são processados quando o worker volta. A observabilidade do worker é detalhada no FDD. |
| Prazo de fim de novembro apertado para 3 sprints. | O escopo foi cortado no que a reunião adiou (e-mail, dashboard, rate limiting). |

## 7. Decisões relacionadas

| ADR | Decisão |
| --- | --- |
| [ADR-001](./adrs/ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL |
| [ADR-002](./adrs/ADR-002-worker-separado-em-polling.md) | Worker em processo separado, em polling de 2 s |
| [ADR-003](./adrs/ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff exponencial e DLQ em tabela separada |
| [ADR-004](./adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação de 24 h |
| [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) | At-least-once com `X-Event-Id` |
| [ADR-006](./adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) | Reuso dos padrões existentes do projeto |

O detalhamento de fluxos, contratos HTTP, matriz de erros `WEBHOOK_*` e integração com os arquivos do código está no [FDD](./FDD.md).
