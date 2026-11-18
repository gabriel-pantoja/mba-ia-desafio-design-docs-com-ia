# Tracker de Rastreabilidade

Referência cruzada entre cada item registrado em [PRD](./PRD.md), [RFC](./RFC.md), [FDD](./FDD.md) e [ADRs](./adrs/README.md) e a sua origem em `TRANSCRICAO.md` (reunião técnica de webhooks) ou no código do repositório.

**Como ler**

- **Fonte = TRANSCRICAO**: `Localização` traz `[hh:mm] Nome`, o timestamp e o falante da linha da transcrição. Quando o item vem de mais de uma fala, as referências são separadas por `;`.
- **Fonte = CODIGO**: `Localização` traz o caminho real do arquivo no repositório.
- IDs de itens de ADR seguem `ADR-00N-NN` (o ADR em si é `ADR-00N`). Itens do FDD seguem os identificadores do próprio documento (`OT-`, `CA-`, `R-`, `P-`, seções `FDD-`).
- Itens marcados como *(proposta FDD)* não foram decididos na reunião. Estão no tracker apontando a fala que originou a lacuna, e não como decisão tomada.
- Itens sem nenhuma origem na reunião nem no código estão listados na seção [Itens sem origem rastreável](#itens-sem-origem-rastreável).

## Tabela de rastreabilidade

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | docs/PRD.md | Objetivo | Notificar o cliente em menos de 10 s (p95), considerado "tempo real" | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Objetivo | Zero mudanças de status commitadas sem evento correspondente | TRANSCRICAO | [09:40] Bruno |
| PRD-OBJ-03 | docs/PRD.md | Objetivo | Eliminar o polling nos 3 clientes solicitantes (Atlas, MaxDistribuição, Nova Cargo) | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-04 | docs/PRD.md | Objetivo | Entregar até o fim de novembro em até 3 sprints, com revisão de segurança | TRANSCRICAO | [09:45] Marcos; [09:47] Larissa |
| PRD-OBJ-05 | docs/PRD.md | Objetivo | Janela de retry de cerca de 15 h antes de dar a entrega como falha | TRANSCRICAO | [09:17] Diego |
| PRD-OBJ-06 | docs/PRD.md | Objetivo | Falha ou lentidão de endpoint de cliente não pode afetar `PATCH /orders/:id/status` | TRANSCRICAO | [09:04] Bruno |
| PRD-OBJ-07 | docs/PRD.md | Objetivo | Nenhum serviço novo de infraestrutura além de um processo worker | TRANSCRICAO | [09:07] Diego; [09:07] Larissa |
| PRD-CTX-01 | docs/PRD.md | Contexto | Pedido formal de Atlas, MaxDistribuição e Nova Cargo; Atlas pode migrar para concorrente | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Clientes hoje fazem polling em `GET /orders`, integração lenta e cara | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-03 | docs/PRD.md | Contexto | Aplicação atual não tem eventos, filas nem webhooks | CODIGO | src/modules/orders/order.service.ts |
| PRD-CTX-04 | docs/PRD.md | Contexto | Transação de mudança de status já é pesada (pedido, histórico, estoque); HTTP dentro dela travaria outros pedidos | TRANSCRICAO | [09:04] Bruno |
| PRD-CU-02 | docs/PRD.md | Cenário de uso | Cadastro de URL https com filtro SHIPPED/DELIVERED e secret devolvida na criação | TRANSCRICAO | [09:31] Marcos; [09:33] Marcos |
| PRD-CU-03 | docs/PRD.md | Cenário de uso | Cliente fora do ar recebe o evento pendente após o retorno, via retries | TRANSCRICAO | [09:16] Diego |
| PRD-CU-04 | docs/PRD.md | Cenário de uso | Entrega esgotada fica guardada; ADMIN reenvia manualmente com registro | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-CU-05 | docs/PRD.md | Cenário de uso | Secret vazada: nova secret pela API e 24 h de convivência com a antiga | TRANSCRICAO | [09:21] Sofia; [09:22] Diego |
| PRD-CU-06 | docs/PRD.md | Cenário de uso | Cliente consulta os últimos 100 envios (sucesso/falha, payload, resposta, tempo) | TRANSCRICAO | [09:34] Marcos |
| PRD-CU-07 | docs/PRD.md | Cenário de uso | Evento duplicado descartado pelo cliente via identificador de evento | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| PRD-CU-08 | docs/PRD.md | Cenário de uso | Status não assinado pelo cliente não gera envio | TRANSCRICAO | [09:34] Bruno |
| PRD-ESC-01 | docs/PRD.md | Restrição | Fora de escopo: e-mail ao cliente quando o webhook falha repetidamente | TRANSCRICAO | [09:37] Larissa; [09:38] Marcos |
| PRD-ESC-02 | docs/PRD.md | Restrição | Fora de escopo: rate limiting de saída ("observar e decidir depois") | TRANSCRICAO | [09:39] Larissa |
| PRD-ESC-03 | docs/PRD.md | Restrição | Fora de escopo: dashboard visual; só endpoints | TRANSCRICAO | [09:40] Larissa |
| PRD-ESC-04 | docs/PRD.md | Restrição | Fora de escopo: arquivamento de eventos entregues (~30 dias) | TRANSCRICAO | [09:08] Diego |
| PRD-ESC-05 | docs/PRD.md | Restrição | Limitação conhecida: um único worker, sem ordenação global | TRANSCRICAO | [09:13] Larissa; [09:14] Marcos |
| PRD-ESC-06 | docs/PRD.md | Restrição | Endurecer permissões do CRUD fica para depois | TRANSCRICAO | [09:37] Sofia |
| PRD-ESC-07 | docs/PRD.md | Restrição | Fora de escopo: webhooks de entrada (somente outbound) | TRANSCRICAO | [09:02] Marcos; [09:03] Sofia |
| PRD-ESC-08 | docs/PRD.md | Restrição | Itens do pedido não vão no payload; cliente usa `GET /orders/:id` | TRANSCRICAO | [09:43] Diego |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL e lista de status; secret gerada pelo OMS e devolvida na criação | TRANSCRICAO | [09:31] Marcos; [09:21] Sofia |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Editar (PATCH), remover (DELETE) e listar (GET) webhooks de um customer; `customer_id` no body ou path | TRANSCRICAO | [09:33] Bruno; [09:32] Larissa |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Filtro de eventos por webhook, aplicado na inserção na outbox | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Evento registrado de forma atômica com a mudança de status | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Envio HTTPS assíncrono por worker, timeout 10 s, não-2xx ou timeout é falha | TRANSCRICAO | [09:11] Diego; [09:42] Diego |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Payload JSON com event_id, event_type, timestamp, order_id, order_number, from/to_status, customer_id, total_cents; snapshot, sem itens | TRANSCRICAO | [09:43] Diego; [09:52] Larissa |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Headers X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id e Content-Type | TRANSCRICAO | [09:44] Diego; [09:44] Sofia |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Assinatura HMAC-SHA256 sobre o corpo, secret única por endpoint | TRANSCRICAO | [09:20] Sofia; [09:22] Sofia |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Rotação de secret pela API, anterior válida por 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Retry com backoff 1 min, 5 min, 30 min, 2 h, 12 h | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Esgotadas as tentativas, evento vai para tabela DLQ com payload, motivo e timestamp | TRANSCRICAO | [09:15] Diego; [09:18] Diego |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Replay de DLQ por `ADMIN` em `POST /admin/webhooks/dead-letter/:id/replay`, com log de quem executou | TRANSCRICAO | [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| PRD-FR-13 | docs/PRD.md | Requisito Funcional | Histórico de entregas em `GET /webhooks/:id/deliveries` (últimas 100) | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-14 | docs/PRD.md | Requisito Funcional | Entrega at-least-once com mesmo `X-Event-Id` para deduplicação | TRANSCRICAO | [09:24] Diego; [09:26] Larissa |
| PRD-FR-15 | docs/PRD.md | Requisito Funcional | Ordem de eventos por `order_id` com um único worker | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| PRD-FR-16 | docs/PRD.md | Requisito Funcional | URL do webhook deve ser https; http recusado com erro de validação | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Latência < 10 s; polling acrescenta no máximo 2 s | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Isolamento: falha do cliente não bloqueia a API; envio em processo separado | TRANSCRICAO | [09:04] Bruno; [09:11] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Consistência: evento na mesma transação da mudança de status | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Somente URLs https | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | HMAC-SHA256, secret por endpoint, rotação com 24 h de convivência | TRANSCRICAO | [09:22] Sofia |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Replay restrito a ADMIN com auditoria; demais operações exigem autenticação | TRANSCRICAO | [09:36] Sofia; [09:37] Sofia |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | Revisão de segurança de no mínimo 2 dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | Payload máximo de 64 KB; acima disso erro, sem truncar | TRANSCRICAO | [09:23] Sofia; [09:24] Diego; [09:24] Larissa |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 s por chamada HTTP | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-10 | docs/PRD.md | Requisito Não Funcional | Janela de retry de cerca de 15 h | TRANSCRICAO | [09:17] Diego |
| PRD-NFR-11 | docs/PRD.md | Requisito Não Funcional | Um único worker; ordem só por pedido, não global | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-12 | docs/PRD.md | Requisito Não Funcional | Sem nova infra: apenas MySQL, Prisma, Express, Zod, Pino; um processo worker | TRANSCRICAO | [09:07] Diego; [09:30] Larissa |
| PRD-NFR-13 | docs/PRD.md | Requisito Não Funcional | Módulo no padrão dos demais, `AppError` com prefixo `WEBHOOK_`, Pino, error middleware | TRANSCRICAO | [09:27] Bruno; [09:29] Larissa; [09:30] Larissa |
| PRD-NFR-14 | docs/PRD.md | Requisito Não Funcional | API de pedidos existente não muda de contrato | CODIGO | src/modules/orders/order.service.ts |
| PRD-NFR-15 | docs/PRD.md | Requisito Não Funcional | UUID em todas as chaves | TRANSCRICAO | [09:51] Larissa |
| PRD-DEC-01 | docs/PRD.md | Decisão | Outbox no MySQL em vez de disparo síncrono ou fila externa | TRANSCRICAO | [09:08] Larissa |
| PRD-DEC-02 | docs/PRD.md | Decisão | Worker separado com polling de 2 s | TRANSCRICAO | [09:10] Larissa |
| PRD-DEC-03 | docs/PRD.md | Decisão | 5 tentativas com backoff e DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-DEC-04 | docs/PRD.md | Decisão | HMAC-SHA256, secret por endpoint, rotação 24 h | TRANSCRICAO | [09:22] Sofia |
| PRD-DEC-05 | docs/PRD.md | Decisão | At-least-once com `X-Event-Id` em vez de exactly-once | TRANSCRICAO | [09:26] Larissa |
| PRD-DEC-06 | docs/PRD.md | Decisão | Reuso máximo dos padrões do projeto | TRANSCRICAO | [09:30] Larissa |
| PRD-TO-01 | docs/PRD.md | Trade-off | Responsabilidade de deduplicar e validar assinatura fica com o cliente; documentação no portal é parte da entrega | TRANSCRICAO | [09:26] Marcos; [09:25] Sofia |
| PRD-TO-02 | docs/PRD.md | Trade-off | Sem e-mail nesta fase, cliente só descobre falha pelo histórico ou contato | TRANSCRICAO | [09:37] Larissa |
| PRD-TO-03 | docs/PRD.md | Trade-off | Prazo de 3 sprints só cabe com e-mail, dashboard e rate limiting fora | TRANSCRICAO | [09:46] Larissa |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança da Sofia precisa ser agendada antes do deploy | TRANSCRICAO | [09:49] Sofia; [09:46] Sofia |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação no portal do desenvolvedor, de responsabilidade de Marcos | TRANSCRICAO | [09:26] Marcos; [09:40] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Comunicação com clientes e confirmação de prazo com a Atlas | TRANSCRICAO | [09:47] Marcos; [09:49] Marcos |
| PRD-DEP-04 | docs/PRD.md | Dependência | Sessão de revisão do design com Bruno e Diego antes de codar | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-05 | docs/PRD.md | Dependência | `OrderService.changeStatus` é o único ponto de domínio alterado | CODIGO | src/modules/orders/order.service.ts |
| PRD-DEP-06 | docs/PRD.md | Dependência | Usuários do OMS representam os clientes; JWT do próprio sistema | TRANSCRICAO | [09:32] Marcos |
| PRD-DEP-07 | docs/PRD.md | Dependência | `requireRole`, `AppError`, error middleware e Pino reaproveitados sem alteração | TRANSCRICAO | [09:29] Bruno; [09:36] Larissa |
| PRD-DEP-08 | docs/PRD.md | Dependência | MySQL 8.0 e Node ≥ 20 já em uso | CODIGO | docker-compose.yml |
| PRD-DEP-09 | docs/PRD.md | Dependência | Um processo worker em produção, uma única réplica, deploy depois da migration | TRANSCRICAO | [09:11] Larissa; [09:13] Larissa |
| PRD-R-01 | docs/PRD.md | Risco | Prazo apertado de 3 sprints com a Atlas ameaçando migrar | TRANSCRICAO | [09:46] Larissa; [09:00] Marcos |
| PRD-R-02 | docs/PRD.md | Risco | Cliente lento atrasa entregas de outros e quebra o limite de 10 s | TRANSCRICAO | [09:42] Diego |
| PRD-R-03 | docs/PRD.md | Risco | Worker parado acumula eventos | TRANSCRICAO | [09:11] Diego |
| PRD-R-04 | docs/PRD.md | Risco | Vazamento de secret por um cliente | TRANSCRICAO | [09:22] Diego |
| PRD-R-05 | docs/PRD.md | Risco | Cliente processa evento em duplicidade | TRANSCRICAO | [09:26] Marcos |
| PRD-R-06 | docs/PRD.md | Risco | Falha na gravação do evento impede mudança de status (intencional) | TRANSCRICAO | [09:40] Bruno |
| PRD-R-07 | docs/PRD.md | Risco | Mais de um worker quebra a ordenação por pedido | TRANSCRICAO | [09:13] Larissa |
| PRD-R-08 | docs/PRD.md | Risco | Cliente não percebe entregas esgotadas sem aviso | TRANSCRICAO | [09:37] Larissa |
| PRD-R-09 | docs/PRD.md | Risco | Rajada de chamadas quando muitos pedidos mudam juntos | TRANSCRICAO | [09:38] Diego |
| PRD-CA-01 | docs/PRD.md | Critério de Aceitação | Mudança de status gera notificação e nenhuma mudança commitada fica sem evento | TRANSCRICAO | [09:40] Bruno |
| PRD-CA-02 | docs/PRD.md | Critério de Aceitação | Se o registro do evento falhar, a mudança de status não é aplicada | TRANSCRICAO | [09:40] Bruno |
| PRD-CA-03 | docs/PRD.md | Critério de Aceitação | Status que nenhum webhook assina não gera evento | TRANSCRICAO | [09:34] Bruno |
| PRD-CA-04 | docs/PRD.md | Critério de Aceitação | Chamada ao cliente em menos de 10 s (p95) no caminho feliz | TRANSCRICAO | [09:02] Marcos |
| PRD-CA-05 | docs/PRD.md | Critério de Aceitação | Cliente valida a assinatura e recebe os 5 headers | TRANSCRICAO | [09:44] Diego |
| PRD-CA-06 | docs/PRD.md | Critério de Aceitação | Retries em 1 min, 5 min, 30 min, 2 h, 12 h e depois DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-CA-07 | docs/PRD.md | Critério de Aceitação | Sem resposta em 10 s é falha e reagendada | TRANSCRICAO | [09:42] Diego |
| PRD-CA-08 | docs/PRD.md | Critério de Aceitação | URL http recusada; https aceita e secret devolvida na criação e rotação | TRANSCRICAO | [09:23] Sofia; [09:31] Marcos |
| PRD-CA-09 | docs/PRD.md | Critério de Aceitação | Após rotação, antiga e nova validam por 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-CA-10 | docs/PRD.md | Critério de Aceitação | Replay negado para OPERATOR; ADMIN funciona, mesmo X-Event-Id, com log | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| PRD-CA-11 | docs/PRD.md | Critério de Aceitação | Histórico das últimas 100 entregas com sucesso/falha, payload, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-12 | docs/PRD.md | Critério de Aceitação | Eventos do mesmo pedido chegam em ordem | TRANSCRICAO | [09:12] Diego |
| PRD-CA-13 | docs/PRD.md | Critério de Aceitação | Payload acima de 64 KB resulta em erro e não é enviado | TRANSCRICAO | [09:24] Larissa |
| PRD-CA-14 | docs/PRD.md | Critério de Aceitação | Testes existentes de pedidos continuam passando | CODIGO | tests/orders.test.ts |
| PRD-CA-15 | docs/PRD.md | Critério de Aceitação | Revisão de segurança (≥ 2 dias úteis) concluída antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-CA-16 | docs/PRD.md | Critério de Aceitação | Documentação de integração disponível no portal | TRANSCRICAO | [09:40] Marcos |
| PRD-TEST-01 | docs/PRD.md | Estratégia de Testes | Testes de integração com banco real no padrão de `tests/orders.test.ts` | CODIGO | tests/orders.test.ts |
| RFC-PROP-01 | docs/RFC.md | Decisão | Outbox transacional: `publishWebhookEvent(tx, ...)` na mesma transação de `changeStatus` | TRANSCRICAO | [09:41] Bruno; [09:40] Bruno |
| RFC-PROP-02 | docs/RFC.md | Decisão | Worker separado (`src/worker.ts`, `npm run worker`) com PrismaClient próprio e polling de 2 s | TRANSCRICAO | [09:11] Larissa; [09:30] Bruno; [09:09] Diego |
| RFC-PROP-03 | docs/RFC.md | Decisão | Retry 5 tentativas com backoff e DLQ em `webhook_dead_letter`, replay por ADMIN | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| RFC-PROP-04 | docs/RFC.md | Decisão | HMAC-SHA256 sobre o corpo, secret por endpoint, rotação com 24 h | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-05 | docs/RFC.md | Decisão | At-least-once com UUID em `X-Event-Id` | TRANSCRICAO | [09:26] Larissa |
| RFC-PROP-06 | docs/RFC.md | Decisão | Módulo `src/modules/webhooks` com `AppError` `WEBHOOK_*`, Pino, `requireRole`, UUIDs | TRANSCRICAO | [09:30] Larissa |
| RFC-API-01 | docs/RFC.md | Requisito Funcional | CRUD aceita qualquer role autenticada por enquanto; `customer_id` no body ou path | TRANSCRICAO | [09:37] Sofia; [09:32] Larissa |
| RFC-NFR-01 | docs/RFC.md | Requisito Não Funcional | URL https validada no schema Zod; payload ≤ 64 KB; timeout 10 s; 5 headers | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa; [09:42] Diego; [09:44] Diego |
| RFC-ALT-01 | docs/RFC.md | Trade-off | Descartado: disparo síncrono dentro de `changeStatus` | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | docs/RFC.md | Trade-off | Descartado: fila externa (Redis Streams), overengineering para time pequeno | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Trade-off | Descartado: trigger/notificação do banco; MySQL não tem LISTEN/NOTIFY | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Trade-off | Descartado: retry indefinido com backoff | TRANSCRICAO | [09:15] Diego |
| RFC-ALT-05 | docs/RFC.md | Trade-off | Descartado: apenas 3 tentativas (cobriria ~30 min) | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-06 | docs/RFC.md | Trade-off | Descartado: exactly-once, por complexidade | TRANSCRICAO | [09:25] Diego |
| RFC-OPEN-01 | docs/RFC.md | Questão em aberto | Rate limiting de saída por cliente | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| RFC-OPEN-02 | docs/RFC.md | Questão em aberto | Notificação por e-mail ao cliente em falhas repetidas | TRANSCRICAO | [09:37] Marcos; [09:37] Larissa |
| RFC-OPEN-03 | docs/RFC.md | Questão em aberto | Escala para múltiplos workers (particionar por `order_id` ou lock pessimista) | TRANSCRICAO | [09:13] Diego; [09:13] Larissa |
| RFC-OPEN-04 | docs/RFC.md | Questão em aberto | Arquivamento de eventos entregues | TRANSCRICAO | [09:08] Diego |
| RFC-OPEN-05 | docs/RFC.md | Questão em aberto | Endurecer permissões do CRUD de webhooks | TRANSCRICAO | [09:37] Sofia |
| RFC-OPEN-06 | docs/RFC.md | Questão em aberto | Dashboard visual para o cliente | TRANSCRICAO | [09:39] Marcos; [09:40] Larissa |
| RFC-IMP-01 | docs/RFC.md | Restrição | `changeStatus` passa a chamar `publishWebhookEvent` dentro da transação; única alteração em domínio existente | CODIGO | src/modules/orders/order.service.ts |
| RFC-IMP-02 | docs/RFC.md | Restrição | Novas tabelas `webhook_outbox`, `webhook_dead_letter` e de configuração; novo processo worker | TRANSCRICAO | [09:06] Diego; [09:18] Diego; [09:21] Bruno |
| RFC-IMP-03 | docs/RFC.md | Restrição | Escrita extra na transação só quando algum webhook assina o status | TRANSCRICAO | [09:34] Bruno |
| RFC-R-01 | docs/RFC.md | Risco | Falha na inserção da outbox bloqueia mudança de status (intencional) | TRANSCRICAO | [09:40] Bruno |
| RFC-R-02 | docs/RFC.md | Risco | Atraso de até 2 s por causa do polling | TRANSCRICAO | [09:10] Larissa; [09:10] Marcos |
| RFC-R-03 | docs/RFC.md | Risco | Quebra de ordem com mais de um worker | TRANSCRICAO | [09:12] Diego |
| RFC-R-04 | docs/RFC.md | Risco | Duplicidade de processamento no cliente | TRANSCRICAO | [09:26] Marcos |
| RFC-R-05 | docs/RFC.md | Risco | Vazamento de secret pelo cliente | TRANSCRICAO | [09:22] Diego |
| RFC-R-06 | docs/RFC.md | Risco | Worker parado acumula eventos na outbox | TRANSCRICAO | [09:11] Diego |
| RFC-R-07 | docs/RFC.md | Risco | Prazo de fim de novembro apertado para 3 sprints | TRANSCRICAO | [09:46] Larissa |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Padrão Outbox no MySQL para publicação de eventos | TRANSCRICAO | [09:08] Larissa |
| ADR-001-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Inserir na outbox na mesma transação de `changeStatus`; falha causa rollback | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| ADR-001-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | `publishWebhookEvent(tx, order, fromStatus, toStatus)` recebe o `tx`, sem injetar repository | TRANSCRICAO | [09:41] Bruno |
| ADR-001-03 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Payload é snapshot renderizado na inserção | TRANSCRICAO | [09:52] Larissa; [09:52] Diego |
| ADR-001-04 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Filtro de status aplicado na inserção | TRANSCRICAO | [09:34] Bruno; [09:34] Diego |
| ADR-001-05 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Chave primária UUID | TRANSCRICAO | [09:51] Larissa |
| ADR-001-06 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Índices em status e `created_at`; arquivamento fora do escopo | TRANSCRICAO | [09:08] Diego |
| ADR-001-07 | docs/adrs/ADR-001-outbox-no-mysql.md | Trade-off | `changeStatus` fica acoplado à outbox e a transação cresce | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-08 | docs/adrs/ADR-001-outbox-no-mysql.md | Restrição | Reaproveita MySQL e Prisma existentes, sem nova infra | CODIGO | prisma/schema.prisma |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Worker em processo separado com polling de 2 s | TRANSCRICAO | [09:10] Larissa; [09:11] Diego |
| ADR-002-01 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Polling a cada 2 s, lote pequeno dos pendentes mais antigos | TRANSCRICAO | [09:09] Diego; [09:08] Diego |
| ADR-002-02 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Nova entry-point `src/worker.ts` análoga a `src/server.ts` e script `npm run worker` | TRANSCRICAO | [09:11] Larissa |
| ADR-002-03 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Lógica em `webhook.worker.ts` ou `webhook.processor.ts` no módulo | TRANSCRICAO | [09:28] Bruno |
| ADR-002-04 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Mesmo banco e `DATABASE_URL`, mas `PrismaClient` próprio por processo | TRANSCRICAO | [09:30] Bruno |
| ADR-002-05 | docs/adrs/ADR-002-worker-separado-em-polling.md | Restrição | Single-worker; ordem por `created_at`, sem ordering global (limitação conhecida) | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| ADR-002-06 | docs/adrs/ADR-002-worker-separado-em-polling.md | Requisito Não Funcional | Timeout de 10 s por chamada; estouro vai para retry | TRANSCRICAO | [09:42] Diego |
| ADR-002-07 | docs/adrs/ADR-002-worker-separado-em-polling.md | Trade-off | Latência mínima de até 2 s e carga constante de leitura | TRANSCRICAO | [09:10] Larissa |
| ADR-002-08 | docs/adrs/ADR-002-worker-separado-em-polling.md | Restrição | Worker modelado em `src/server.ts` (entry-point existente) | CODIGO | src/server.ts |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Retry com backoff exponencial e DLQ em tabela separada | TRANSCRICAO | [09:17] Larissa |
| ADR-003-01 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | 5 tentativas com backoff 1 min, 5 min, 30 min, 2 h, 12 h (~15 h) | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| ADR-003-02 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | DLQ em `webhook_dead_letter` com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| ADR-003-03 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Replay manual via `POST /admin/webhooks/dead-letter/:id/replay` | TRANSCRICAO | [09:18] Diego; [09:35] Diego |
| ADR-003-04 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Replay exige role ADMIN via `requireRole` e registra quem executou | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| ADR-003-05 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Trade-off | Cliente fora por mais de ~15 h depende de replay manual; e-mail fora de escopo | TRANSCRICAO | [09:37] Larissa |
| ADR-003-06 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Restrição | `requireRole` já existe em `auth.middleware.ts` e é reaproveitado | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 com secret por endpoint e rotação com grace period | TRANSCRICAO | [09:22] Sofia |
| ADR-004-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Assinar o corpo com HMAC-SHA256 no header `X-Signature` | TRANSCRICAO | [09:20] Sofia |
| ADR-004-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Secret única por endpoint, gerada pela plataforma, nunca global | TRANSCRICAO | [09:21] Sofia; [09:31] Marcos |
| ADR-004-03 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Rotação por endpoint da API; antiga válida por 24 h | TRANSCRICAO | [09:21] Sofia |
| ADR-004-04 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Requisito Funcional | Headers X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id, Content-Type | TRANSCRICAO | [09:44] Diego; [09:44] Sofia |
| ADR-004-05 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Requisito Não Funcional | URL https (schema Zod) e payload ≤ 64 KB, tratados como requisitos | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| ADR-004-06 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Trade-off | Armazenar secrets utilizáveis (não só hash) e duas durante a rotação | TRANSCRICAO | [09:21] Sofia |
| ADR-004-07 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Restrição | Código de HMAC e geração de secret exige revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia |
| ADR-004-08 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Cliente já vazou secret em log de aplicação | TRANSCRICAO | [09:22] Diego |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | Entrega at-least-once com `X-Event-Id` para deduplicação | TRANSCRICAO | [09:26] Larissa |
| ADR-005-01 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | Cliente pode receber o mesmo evento mais de uma vez | TRANSCRICAO | [09:24] Diego |
| ADR-005-02 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | UUID gerado ao entrar na outbox, enviado em `X-Event-Id` | TRANSCRICAO | [09:25] Diego |
| ADR-005-03 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Requisito Funcional | Formato do payload do evento `order.status_changed`, sem itens | TRANSCRICAO | [09:43] Diego |
| ADR-005-04 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Restrição | Deduplicação é do cliente, documentada em destaque no portal | TRANSCRICAO | [09:26] Marcos |
| ADR-005-05 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Trade-off | Duplicatas possíveis; responsabilidade transferida ao cliente | TRANSCRICAO | [09:25] Sofia; [09:25] Diego |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Reuso máximo dos padrões existentes do projeto | TRANSCRICAO | [09:30] Larissa |
| ADR-006-01 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Módulo `src/modules/webhooks` (controller, service, repository, routes, schemas) | TRANSCRICAO | [09:27] Bruno |
| ADR-006-02 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Erros derivados de `AppError` com prefixo `WEBHOOK_` | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| ADR-006-03 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Error middleware centralizado já trata AppError, Zod e Prisma | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-04 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Logger Pino sem biblioteca nova | TRANSCRICAO | [09:29] Bruno |
| ADR-006-05 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Schemas Zod com `validate.middleware.ts` | CODIGO | src/middlewares/validate.middleware.ts |
| ADR-006-06 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | `requireRole('ADMIN')` só no replay; CRUD exige apenas autenticação | TRANSCRICAO | [09:36] Larissa; [09:37] Sofia |
| ADR-006-07 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Trade-off | `OrderService` passa a depender do módulo de webhooks via `publishWebhookEvent` | TRANSCRICAO | [09:41] Bruno |
| ADR-006-08 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Trade-off | CRUD aberto a qualquer role autenticada é provisório | TRANSCRICAO | [09:37] Sofia |
| ADR-006-09 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Restrição | Classes `AppError`, `InsufficientStockError`, `InvalidStatusTransitionError` existentes como molde | CODIGO | src/shared/errors/http-errors.ts |
| FDD-OT-01 | docs/FDD.md | Objetivo | Evento e mudança de status atômicos | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| FDD-OT-02 | docs/FDD.md | Objetivo | Entrega em < 10 s (até 2 s de polling + chamada HTTP) | TRANSCRICAO | [09:02] Marcos; [09:10] Larissa |
| FDD-OT-03 | docs/FDD.md | Objetivo | Latência ou queda de cliente não afeta a API de pedidos | TRANSCRICAO | [09:04] Bruno; [09:11] Diego |
| FDD-OT-04 | docs/FDD.md | Objetivo | Autenticidade verificável: HMAC-SHA256, secret por endpoint, rotação 24 h | TRANSCRICAO | [09:19] Sofia; [09:22] Sofia |
| FDD-OT-05 | docs/FDD.md | Objetivo | At-least-once com `X-Event-Id` | TRANSCRICAO | [09:26] Larissa |
| FDD-OT-06 | docs/FDD.md | Objetivo | 5 retries com backoff, DLQ e replay manual por ADMIN | TRANSCRICAO | [09:17] Larissa; [09:36] Larissa |
| FDD-OT-07 | docs/FDD.md | Objetivo | Zero dependências e infra novas | TRANSCRICAO | [09:07] Diego; [09:30] Larissa |
| FDD-OT-08 | docs/FDD.md | Objetivo | Módulo no padrão dos demais com códigos `WEBHOOK_*` | TRANSCRICAO | [09:27] Bruno; [09:29] Larissa |
| FDD-CTX-01 | docs/FDD.md | Contexto | `changeStatus` abre `$transaction` que lê o pedido, valida `canTransition`, ajusta estoque, atualiza `orders` e grava histórico | CODIGO | src/modules/orders/order.service.ts |
| FDD-CTX-02 | docs/FDD.md | Contexto | Matriz de transições válidas de status em `canTransition` | CODIGO | src/modules/orders/order.status.ts |
| FDD-ESC-01 | docs/FDD.md | Restrição | Somente `changeStatus` emite evento; `create` não emite *(proposta FDD)* | CODIGO | src/modules/orders/order.service.ts |
| FDD-ESC-02 | docs/FDD.md | Restrição | Evento único `order.status_changed` | TRANSCRICAO | [09:43] Diego |
| FDD-DATA-01 | docs/FDD.md | Decisão | Chaves UUID `Char(36)` e `snake_case` via `@@map`, como o restante do schema | CODIGO | prisma/schema.prisma |
| FDD-DATA-02 | docs/FDD.md | Decisão | `webhook_outbox.id` é o `event_id` e `X-Event-Id` | TRANSCRICAO | [09:25] Diego |
| FDD-DATA-03 | docs/FDD.md | Decisão | Um registro de outbox por endpoint inscrito; `X-Webhook-Id` identifica o cadastro | TRANSCRICAO | [09:44] Sofia |
| FDD-DATA-04 | docs/FDD.md | Decisão | Índices em status e `created_at` na outbox | TRANSCRICAO | [09:08] Diego |
| FDD-DATA-05 | docs/FDD.md | Decisão | `orderId` sem FK, pois `OrderService.delete` permite apagar pedidos PENDING/CANCELLED | CODIGO | src/modules/orders/order.service.ts |
| FDD-DATA-06 | docs/FDD.md | Decisão | DLQ guarda payload, motivo e timestamp; `replayedById` sustenta auditoria | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-DATA-07 | docs/FDD.md | Decisão | Estados da outbox: pendente, processando, entregue, falhou | TRANSCRICAO | [09:08] Diego |
| FDD-DATA-08 | docs/FDD.md | Decisão | Config de webhook armazena url, secret, customer_id e estado ativo | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia |
| FDD-FLOW-01 | docs/FDD.md | Decisão | `publishWebhookEvent` em `webhook.publisher.ts`: função pura recebendo `tx` | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| FDD-FLOW-02 | docs/FDD.md | Decisão | Filtrar endpoints por `toStatus` antes de inserir; sem assinantes, nada é inserido | TRANSCRICAO | [09:34] Bruno; [09:34] Diego |
| FDD-FLOW-03 | docs/FDD.md | Decisão | `eventId` UUID gerado com a dependência `uuid` já usada no projeto | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-FLOW-04 | docs/FDD.md | Requisito Não Funcional | Payload > 65.536 bytes lança erro e nada é enviado truncado | TRANSCRICAO | [09:24] Larissa; [09:23] Sofia |
| FDD-FLOW-05 | docs/FDD.md | Decisão | Exceção no publisher causa rollback da transação inteira | TRANSCRICAO | [09:40] Bruno |
| FDD-FLOW-06 | docs/FDD.md | Decisão | Ponto de inserção: após `orderStatusHistory.create` e antes da releitura `refreshed` | CODIGO | src/modules/orders/order.service.ts |
| FDD-FLOW-07 | docs/FDD.md | Decisão | Worker em `src/worker.ts` com lógica em `webhook.processor.ts` | TRANSCRICAO | [09:11] Larissa; [09:28] Bruno |
| FDD-FLOW-08 | docs/FDD.md | Decisão | Worker cria `PrismaClient` próprio via `createPrismaClient()` | CODIGO | src/config/database.ts |
| FDD-FLOW-09 | docs/FDD.md | Decisão | `SIGINT`/`SIGTERM` com `shutdown_initiated` e `$disconnect`, como `server.ts` | CODIGO | src/server.ts |
| FDD-FLOW-10 | docs/FDD.md | Decisão | Ciclo de polling a cada 2 s | TRANSCRICAO | [09:09] Diego |
| FDD-FLOW-11 | docs/FDD.md | Decisão | Lote pequeno de pendentes por ciclo | TRANSCRICAO | [09:08] Diego |
| FDD-FLOW-12 | docs/FDD.md | Decisão | Consulta por `createdAt ASC` e eventos do mesmo `orderId` em sequência preservam ordem | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| FDD-FLOW-13 | docs/FDD.md | Decisão | `fetch` global do Node ≥ 20 com `AbortSignal.timeout(10_000)` | CODIGO | package.json |
| FDD-FLOW-14 | docs/FDD.md | Requisito Não Funcional | Timeout de 10 s; sucesso = qualquer 2xx | TRANSCRICAO | [09:42] Diego |
| FDD-FLOW-15 | docs/FDD.md | Decisão | Cada tentativa grava linha em `webhook_delivery_attempts`, base do histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| FDD-RETRY-01 | docs/FDD.md | Decisão | `RETRY_DELAYS_MS` = 1 min, 5 min, 30 min, 2 h, 12 h | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| FDD-RETRY-02 | docs/FDD.md | Premissa | Interpretação 1 tentativa inicial + 5 retries (6 chamadas), pois só assim a soma ≈ 15 h fecha *(proposta FDD)* | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| FDD-RETRY-03 | docs/FDD.md | Requisito Não Funcional | Falha = não 2xx, 3xx, erro de rede/DNS/TLS ou timeout de 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-DLQ-01 | docs/FDD.md | Decisão | 6ª falha move para `webhook_dead_letter` com payload, motivo e timestamps, numa transação | TRANSCRICAO | [09:18] Diego |
| FDD-DLQ-02 | docs/FDD.md | Decisão | Replay em `POST /api/v1/admin/webhooks/dead-letter/:id/replay` com `requireRole('ADMIN')` | TRANSCRICAO | [09:35] Diego; [09:36] Larissa |
| FDD-DLQ-03 | docs/FDD.md | Decisão | Replay recoloca a mesma linha como PENDING, mantendo o `event_id` | TRANSCRICAO | [09:18] Diego; [09:25] Diego |
| FDD-DLQ-04 | docs/FDD.md | Decisão | Log `webhook_dlq_replayed` com `userId`, `deadLetterId`, `eventId` para auditoria | TRANSCRICAO | [09:36] Sofia |
| FDD-SEC-01 | docs/FDD.md | Decisão | Rotação: `previousSecret` válida por 24 h; secret nova | TRANSCRICAO | [09:21] Sofia |
| FDD-SEC-02 | docs/FDD.md | Premissa | Assinatura dupla em `X-Signature` durante a rotação *(proposta FDD, decorre da convivência de 24 h)* | TRANSCRICAO | [09:21] Sofia |
| FDD-API-01 | docs/FDD.md | Restrição | Prefixo `/api/v1` e todos os endpoints exigem `authenticate` | CODIGO | src/app.ts |
| FDD-API-02 | docs/FDD.md | Restrição | Formato de erro `{ error: { code, message, details } }` do error middleware | CODIGO | src/middlewares/error.middleware.ts |
| FDD-API-03 | docs/FDD.md | Restrição | Resposta da API em camelCase; payload do webhook em snake_case | TRANSCRICAO | [09:43] Diego |
| FDD-API-04 | docs/FDD.md | Decisão | `customerId` vem no body ou query, nunca do JWT | TRANSCRICAO | [09:32] Larissa |
| FDD-API-05 | docs/FDD.md | Restrição | Qualquer role autenticada nos endpoints de CRUD; ADMIN no replay | TRANSCRICAO | [09:36] Larissa; [09:37] Sofia |
| FDD-API-06 | docs/FDD.md | Requisito Funcional | `POST /api/v1/webhooks`: cadastra, valida https, devolve secret só na criação | TRANSCRICAO | [09:31] Marcos; [09:23] Sofia |
| FDD-API-07 | docs/FDD.md | Requisito Funcional | `events` é array de `OrderStatus` | CODIGO | prisma/schema.prisma |
| FDD-API-08 | docs/FDD.md | Requisito Funcional | `GET /api/v1/webhooks?customerId=` com paginação no molde de `listCustomersQuerySchema` e `paginated()` | CODIGO | src/modules/customers/customer.schemas.ts |
| FDD-API-09 | docs/FDD.md | Requisito Funcional | `paginated()` usado na listagem e no histórico | CODIGO | src/shared/http/response.ts |
| FDD-API-10 | docs/FDD.md | Requisito Funcional | `PATCH /api/v1/webhooks/:id`: editar webhook | TRANSCRICAO | [09:33] Bruno |
| FDD-API-11 | docs/FDD.md | Requisito Funcional | Filtro `events` e `active` valem para eventos futuros; filtro aplicado na inserção | TRANSCRICAO | [09:34] Bruno |
| FDD-API-12 | docs/FDD.md | Requisito Funcional | `DELETE /api/v1/webhooks/:id`: remover webhook (204, sem corpo como `DELETE /orders/:id`) | TRANSCRICAO | [09:33] Bruno |
| FDD-API-13 | docs/FDD.md | Requisito Funcional | `POST /api/v1/webhooks/:id/rotate-secret`: gera nova secret, anterior válida por 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-API-14 | docs/FDD.md | Requisito Funcional | `GET /api/v1/webhooks/:id/deliveries`: últimas 100 entregas com sucesso/falha, payload, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| FDD-API-15 | docs/FDD.md | Requisito Funcional | `POST /api/v1/admin/webhooks/dead-letter/:id/replay` restrito a ADMIN | TRANSCRICAO | [09:35] Diego; [09:36] Sofia |
| FDD-API-16 | docs/FDD.md | Restrição | `FORBIDDEN` 403 vem de `requireRole` | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-OUT-01 | docs/FDD.md | Requisito Funcional | Header `Content-Type: application/json` | TRANSCRICAO | [09:44] Diego |
| FDD-OUT-02 | docs/FDD.md | Requisito Funcional | `X-Event-Id` com UUID estável entre retries | TRANSCRICAO | [09:25] Diego |
| FDD-OUT-03 | docs/FDD.md | Requisito Funcional | `X-Webhook-Id` com id do endpoint de destino | TRANSCRICAO | [09:44] Sofia |
| FDD-OUT-04 | docs/FDD.md | Requisito Funcional | `X-Timestamp` com instante do envio, para detecção de replay pelo cliente | TRANSCRICAO | [09:44] Diego |
| FDD-OUT-05 | docs/FDD.md | Requisito Funcional | `X-Signature` com o HMAC do corpo | TRANSCRICAO | [09:20] Sofia |
| FDD-OUT-06 | docs/FDD.md | Requisito Funcional | Corpo do webhook: event_id, event_type, timestamp, order_id, order_number, from/to_status, customer_id, total_cents | TRANSCRICAO | [09:43] Diego |
| FDD-OUT-07 | docs/FDD.md | Decisão | HMAC sobre os bytes exatos do corpo, com `node:crypto` | TRANSCRICAO | [09:22] Sofia |
| FDD-OUT-08 | docs/FDD.md | Restrição | Cliente deve deduplicar por `X-Event-Id`; orientação vai ao portal do desenvolvedor | TRANSCRICAO | [09:26] Marcos |
| FDD-OUT-09 | docs/FDD.md | Premissa | `X-Timestamp` fora do HMAC; decisão foi assinar o corpo *(P-03)* | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
| FDD-ERR-01 | docs/FDD.md | Decisão | Códigos `WEBHOOK_*` para erros do módulo (`WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`) | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| FDD-ERR-02 | docs/FDD.md | Decisão | Classes novas no molde de `InsufficientStockError` (estende `UnprocessableEntityError` com código próprio) | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERR-03 | docs/FDD.md | Restrição | `NotFoundError` fixa o código `NOT_FOUND`; por isso `WebhookNotFoundError` estende `AppError` direto | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERR-04 | docs/FDD.md | Requisito Funcional | `WEBHOOK_INVALID_URL` (400) quando URL não é https | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno |
| FDD-ERR-05 | docs/FDD.md | Requisito Não Funcional | `WEBHOOK_PAYLOAD_TOO_LARGE` (422) acima de 64 KB | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-06 | docs/FDD.md | Restrição | Erros genéricos `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN` e Prisma `P2002`/`P2025` reaproveitados | CODIGO | src/middlewares/error.middleware.ts |
| FDD-ERR-07 | docs/FDD.md | Requisito Funcional | `WEBHOOK_DEAD_LETTER_NOT_FOUND` e `WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED` no replay | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-08 | docs/FDD.md | Requisito Funcional | `WEBHOOK_SECRET_REQUIRED` quando endpoint não tem secret utilizável para assinar | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-09 | docs/FDD.md | Requisito Não Funcional | `WEBHOOK_DELIVERY_TIMEOUT` quando não há resposta em 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-10 | docs/FDD.md | Requisito Não Funcional | `WEBHOOK_RETRIES_EXHAUSTED` registra a movimentação para a DLQ | TRANSCRICAO | [09:15] Diego |
| FDD-RES-01 | docs/FDD.md | Restrição | Worker isolado da API: a API nunca espera HTTP externo | TRANSCRICAO | [09:11] Diego |
| FDD-RES-02 | docs/FDD.md | Restrição | Crash entre 2xx e UPDATE reenvia o evento; cliente deduplica | TRANSCRICAO | [09:24] Diego |
| FDD-RES-03 | docs/FDD.md | Restrição | Ordem só por `order_id` e só com worker único; evento em backoff pode ser ultrapassado | TRANSCRICAO | [09:13] Larissa |
| FDD-RES-04 | docs/FDD.md | Restrição | Teto de 64 KB na inserção; payload sem itens | TRANSCRICAO | [09:24] Larissa; [09:43] Diego |
| FDD-OBS-01 | docs/FDD.md | Restrição | Projeto só tem Pino; observabilidade baseada em logs estruturados, sem lib nova | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-02 | docs/FDD.md | Restrição | `redactPaths` hoje não cobre `secret`; precisa incluir `*.secret`, `*.previousSecret`, `x-signature` | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-03 | docs/FDD.md | Restrição | Nunca logar `secret`, por histórico de cliente que vazou secret em log | TRANSCRICAO | [09:22] Diego |
| FDD-OBS-04 | docs/FDD.md | Requisito Funcional | Log `webhook_dlq_replayed` com quem executou o replay | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-05 | docs/FDD.md | Restrição | Nomes de evento de log em snake_case como `http_request` e `server_started` | CODIGO | src/server.ts |
| FDD-OBS-06 | docs/FDD.md | Requisito Não Funcional | Indicador de SLO: idade do pendente mais antigo < 10 s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-07 | docs/FDD.md | Restrição | `X-Request-Id` da API não é propagado à outbox, pois a assinatura de `publishWebhookEvent` foi fixada | TRANSCRICAO | [09:41] Bruno |
| FDD-INT-01 | docs/FDD.md | Restrição | `order.service.ts` chama `publishWebhookEvent(tx, ...)` na `$transaction` de `changeStatus` | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Restrição | Registrar rotas `/webhooks` e `/admin/webhooks` no router principal | CODIGO | src/routes/index.ts |
| FDD-INT-03 | docs/FDD.md | Restrição | Instanciar repository/service/controller de webhooks em `buildControllers` | CODIGO | src/app.ts |
| FDD-INT-04 | docs/FDD.md | Restrição | Adicionar enum e 4 models Prisma e relação em `Customer`; nova migration | CODIGO | prisma/schema.prisma |
| FDD-INT-05 | docs/FDD.md | Restrição | `authenticate` no router e `requireRole('ADMIN')` no replay; `req.user.id` alimenta `replayedById` | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-06 | docs/FDD.md | Restrição | `validate({ body, params, query })` repassa erros não-Zod a `next(err)` | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-INT-07 | docs/FDD.md | Restrição | Singleton `prisma` é o da API; worker usa `createPrismaClient()` próprio | CODIGO | src/config/database.ts |
| FDD-INT-08 | docs/FDD.md | Restrição | Novas variáveis de ambiente do worker (poll 2000 ms, batch, concorrência, timeout 10000 ms) | CODIGO | src/config/env.ts |
| FDD-INT-09 | docs/FDD.md | Restrição | `OrderStatus` do `@prisma/client` valida `events`; `canTransition` barra eventos inválidos | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-10 | docs/FDD.md | Restrição | Scripts `worker` e `start:worker` no `package.json`, sem dependência nova | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-11 | docs/FDD.md | Restrição | `tests/setup.ts` limpa tabelas na ordem das FKs; incluir as tabelas de webhook | CODIGO | tests/setup.ts |
| FDD-INT-12 | docs/FDD.md | Restrição | Factories de teste no estilo de `createTestCustomer` | CODIGO | tests/helpers/factories.ts |
| FDD-INT-13 | docs/FDD.md | Restrição | Estrutura do módulo novo segue a de `src/modules/orders/` | TRANSCRICAO | [09:27] Bruno |
| FDD-DEP-01 | docs/FDD.md | Restrição | Runtime Node ≥ 20 (`engines`) com `fetch`, `AbortSignal.timeout` e `node:crypto` | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Restrição | MySQL 8.0 do `docker-compose.yml` | CODIGO | docker-compose.yml |
| FDD-DEP-03 | docs/FDD.md | Restrição | Compatibilidade aditiva: nenhuma rota, resposta ou schema existente muda | CODIGO | src/modules/orders/order.routes.ts |
| FDD-DEP-04 | docs/FDD.md | Restrição | Deploy do worker depois da migration, com um único worker | TRANSCRICAO | [09:13] Larissa |
| FDD-DEP-05 | docs/FDD.md | Restrição | Cadência de 3 sprints com revisão da Sofia ao fim | TRANSCRICAO | [09:47] Larissa |
| FDD-CA-01 | docs/FDD.md | Critério de Aceitação | 1 linha na outbox por endpoint inscrito, na mesma transação | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-02 | docs/FDD.md | Critério de Aceitação | Falha em `publishWebhookEvent` mantém `orders`, histórico e estoque inalterados | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-03 | docs/FDD.md | Critério de Aceitação | Status não assinado não insere na outbox | TRANSCRICAO | [09:34] Bruno |
| FDD-CA-04 | docs/FDD.md | Critério de Aceitação | Entrega em < 10 s após o COMMIT (p95) | TRANSCRICAO | [09:02] Marcos |
| FDD-CA-05 | docs/FDD.md | Critério de Aceitação | Corpo confere com `HMAC_SHA256(secret, corpo)`; 5 headers presentes | TRANSCRICAO | [09:44] Diego; [09:20] Sofia |
| FDD-CA-06 | docs/FDD.md | Critério de Aceitação | Retries nos 5 intervalos e DLQ após a última falha | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| FDD-CA-07 | docs/FDD.md | Critério de Aceitação | Sem resposta em 10 s é falha e reagendada | TRANSCRICAO | [09:42] Diego |
| FDD-CA-08 | docs/FDD.md | Critério de Aceitação | http retorna 400 `WEBHOOK_INVALID_URL`; https retorna 201 com secret | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-09 | docs/FDD.md | Critério de Aceitação | `GET /webhooks` nunca devolve `secret` | TRANSCRICAO | [09:31] Marcos |
| FDD-CA-10 | docs/FDD.md | Critério de Aceitação | Após rotação, `X-Signature` com as duas assinaturas por 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-CA-11 | docs/FDD.md | Critério de Aceitação | Replay: OPERATOR recebe 403; ADMIN recebe 200 com mesmo `event_id` e log | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| FDD-CA-12 | docs/FDD.md | Critério de Aceitação | Retry reutiliza mesmo `X-Event-Id` e mesmo corpo (snapshot) | TRANSCRICAO | [09:52] Larissa; [09:25] Diego |
| FDD-CA-13 | docs/FDD.md | Critério de Aceitação | Eventos de um mesmo `order_id` entregues em ordem, com um worker | TRANSCRICAO | [09:12] Diego |
| FDD-CA-14 | docs/FDD.md | Critério de Aceitação | Após `kill -9`, eventos PROCESSING voltam a PENDING e são entregues (possível duplicata) | TRANSCRICAO | [09:11] Diego |
| FDD-CA-15 | docs/FDD.md | Critério de Aceitação | `secret`/`previousSecret` não aparecem em logs | TRANSCRICAO | [09:22] Diego |
| FDD-CA-16 | docs/FDD.md | Critério de Aceitação | `npm run lint`, `build` e `test` passam; testes de pedidos seguem verdes | CODIGO | package.json |
| FDD-CA-17 | docs/FDD.md | Critério de Aceitação | Revisão de segurança da Sofia (≥ 2 dias úteis) antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-TEST-01 | docs/FDD.md | Estratégia de Testes | Seguir o padrão de `tests/orders.test.ts` (supertest + banco real) | CODIGO | tests/orders.test.ts |
| FDD-P-01 | docs/FDD.md | Premissa | Contagem de tentativas: "5 tentativas" vs. 5 intervalos de backoff *(proposta FDD)* | TRANSCRICAO | [09:15] Diego; [09:17] Diego |
| FDD-P-02 | docs/FDD.md | Premissa | Formato de `X-Signature` e assinatura dupla na rotação *(proposta FDD)* | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia |
| FDD-P-03 | docs/FDD.md | Premissa | `X-Timestamp` fora do HMAC *(proposta FDD)* | TRANSCRICAO | [09:44] Diego |
| FDD-P-04 | docs/FDD.md | Premissa | Secret armazenada utilizável (não hash), pois o HMAC exige o valor *(proposta FDD)* | TRANSCRICAO | [09:21] Sofia |
| FDD-P-05 | docs/FDD.md | Premissa | Métricas/tracing sem biblioteca nova *(proposta FDD)* | TRANSCRICAO | [09:29] Bruno |
| FDD-P-06 | docs/FDD.md | Premissa | Payload > 64 KB aborta a transação de `changeStatus` *(proposta FDD)* | TRANSCRICAO | [09:24] Larissa; [09:40] Bruno |
| FDD-P-07 | docs/FDD.md | Premissa | Concorrência entre pedidos distintos no worker *(proposta FDD)* | TRANSCRICAO | [09:42] Diego |
| FDD-P-08 | docs/FDD.md | Premissa | `DELETE` remove em cascata a outbox pendente do endpoint *(proposta FDD)* | TRANSCRICAO | [09:33] Bruno |
| FDD-P-10 | docs/FDD.md | Premissa | Reenvio sem checar se o cliente já processou, aceito por at-least-once | TRANSCRICAO | [09:24] Diego |
| FDD-R-01 | docs/FDD.md | Risco | Falha na inserção da outbox bloqueia mudança de status (intencional) | TRANSCRICAO | [09:40] Bruno |
| FDD-R-02 | docs/FDD.md | Risco | Cliente lento atrasa a fila de outros clientes e viola o limite de 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-R-03 | docs/FDD.md | Risco | Mais de um worker quebra a ordenação | TRANSCRICAO | [09:13] Larissa |
| FDD-R-04 | docs/FDD.md | Risco | Worker parado acumula eventos | TRANSCRICAO | [09:11] Diego |
| FDD-R-05 | docs/FDD.md | Risco | Vazamento de secret pelo cliente | TRANSCRICAO | [09:22] Diego |
| FDD-R-06 | docs/FDD.md | Risco | Cliente processa evento em duplicidade | TRANSCRICAO | [09:26] Marcos |
| FDD-R-07 | docs/FDD.md | Risco | Crescimento sem limite da outbox e das tentativas | TRANSCRICAO | [09:08] Diego |
| FDD-R-08 | docs/FDD.md | Risco | Rajada de chamadas para cliente com muitas mudanças | TRANSCRICAO | [09:38] Diego |
| FDD-R-09 | docs/FDD.md | Risco | Falhas silenciosas na DLQ sem aviso ao cliente | TRANSCRICAO | [09:37] Larissa |
| FDD-R-10 | docs/FDD.md | Risco | Ordem quebrada por evento em backoff | TRANSCRICAO | [09:14] Marcos |
| FDD-R-11 | docs/FDD.md | Risco | Falha de segurança em HMAC/geração de secret | TRANSCRICAO | [09:46] Sofia |
| FDD-R-12 | docs/FDD.md | Risco | Prazo de 3 sprints apertado para fim de novembro | TRANSCRICAO | [09:46] Larissa |

## Itens sem origem rastreável

Itens dos documentos que **não** têm origem na reunião nem no código existente. Foram deixados de fora da tabela de propósito, para o tracker não atribuir a uma fala algo que ninguém disse.

| Item | Documento | Observação |
| --- | --- | --- |
| P-09 (proteção contra SSRF) | docs/FDD.md | Não foi discutida na reunião; o FDD a leva como ponto para a revisão de segurança |
| Detalhes de implementação marcados *(proposta FDD)* sem fala correspondente | docs/FDD.md | Formato `sha256=<hex>`, recuperação de órfãos `PROCESSING` na partida, `redirect: 'manual'`, índice composto `(status, nextAttemptAt)`, tamanhos de lote e concorrência, métricas e alertas sugeridos, truncamento de 2 KB da resposta, filtro `success` no histórico |
| Meta de adoção "3 de 3" e piloto com clientes | docs/PRD.md | Marcados como proposta do PRD; derivam do pedido formal ([09:00] Marcos), mas a medição e o piloto não foram definidos |
