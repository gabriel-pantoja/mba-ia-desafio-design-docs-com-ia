# PRD-001: Sistema de Webhooks de Notificação de Pedidos

## Metadados

| Campo | Valor |
| --- | --- |
| **Autor** | Gabriel Pantoja (consolidação da reunião técnica conduzida por Larissa) |
| **Status** | Rascunho para revisão |
| **Data** | 18-11-2026 |
| **Dono do produto** | Marcos (Product Manager) |
| **Stakeholders** | Larissa (Tech Lead), Bruno (Engenheiro Pleno, Pedidos), Diego (Engenheiro Sênior, Plataforma), Sofia (Engenheira de Segurança) |
| **Prazo alvo** | Fim de novembro, pedido da Atlas ([09:45] Marcos); estimativa de 3 sprints, com revisão de segurança incluída ([09:47] Larissa) |
| **Documentos relacionados** | [RFC](./RFC.md) · [FDD](./FDD.md) · [ADRs](./adrs/README.md) · [Tracker](./TRACKER.md) |

**Convenção de rastreabilidade.** Itens da reunião citam `[hh:mm] Nome` (fonte: `TRANSCRICAO.md`); itens do código citam o caminho do arquivo. Este PRD responde *por que* e *o quê*. O *como* está no [RFC](./RFC.md), nos [ADRs](./adrs/README.md) e no [FDD](./FDD.md), e não é repetido aqui.

---

## 1. Resumo e contexto da feature

Hoje, quando o status de um pedido muda no OMS, o cliente B2B só descobre consultando `GET /orders` repetidamente. A feature entrega **webhooks de saída**: o OMS passa a avisar, por HTTPS, o endpoint cadastrado pelo cliente sempre que o status de um pedido dele mudar, em menos de 10 segundos.

O cliente cadastra e gerencia seus webhooks pela API do OMS, escolhe quais status quer ouvir, valida a autenticidade de cada notificação por assinatura e consulta o histórico de entregas. A plataforma garante que nenhuma mudança de status deixe de gerar notificação, repete entregas que falharem e dá à operação um caminho de reprocessamento.

A aplicação existente não tem eventos, filas nem webhooks (`src/modules/orders/order.service.ts`, `README.md`), então esta é a primeira capacidade de notificação externa do produto.

## 2. Problema e motivação

**Problema.** Atlas Comercial, MaxDistribuição e Nova Cargo fazem polling em `GET /orders` para saber se um pedido mudou. Isso deixa a integração **lenta e cara para eles** ([09:00] Marcos). Esse polling é exatamente o que a feature elimina.

**Motivação de negócio.**

- Os três clientes fizeram **pedido formal** na semana anterior à reunião ([09:00] Marcos).
- A Atlas sinalizou que, se a entrega passar do fim do trimestre, **pode migrar para um concorrente** ([09:00] Marcos). A Atlas quer a feature para **fim de novembro** ([09:45] Marcos).
- Para esses clientes, "tempo real" significa **qualquer coisa abaixo de 10 segundos**; o essencial é não ficarem atualizando manualmente ([09:02] Marcos).

**Por que não é trivial.** A mudança de status roda numa transação que já atualiza o pedido, grava o histórico e debita estoque (`OrderService.changeStatus`). Avisar o cliente dentro dessa transação faria um cliente lento travar mudanças de status de outros pedidos, e um cliente fora do ar forçaria desfazer uma mudança válida ([09:04] Bruno). A feature precisa avisar o cliente **sem** comprometer a operação de pedidos.

## 3. Público-alvo e cenários de uso

### 3.1 Públicos

| Público | Quem é | O que precisa |
| --- | --- | --- |
| **Cliente B2B integrador** | Sistemas de Atlas Comercial, MaxDistribuição e Nova Cargo, via usuários do OMS que os representam ([09:32] Marcos) | Receber a notificação, validar que veio do OMS, não processar duplicatas, ver o que foi enviado |
| **Operador do OMS** | Usuário autenticado que opera pedidos e, por ora, também configura webhooks ([09:37] Sofia) | Cadastrar e manter webhooks de um customer |
| **Administrador do OMS** | Usuário com role `ADMIN` | Reprocessar entregas que esgotaram as tentativas, com auditoria ([09:36] Sofia) |
| **Time de engenharia / segurança** | Larissa, Bruno, Diego, Sofia | Operar o worker e revisar a segurança antes do deploy |
| **Produto / documentação** | Marcos | Documentar a integração no portal do desenvolvedor ([09:26] Marcos, [09:40] Marcos) |

### 3.2 Cenários de uso

| ID | Cenário |
| --- | --- |
| CU-01 | **Caminho feliz.** Um pedido da Atlas passa para `SHIPPED`. Em menos de 10 s, o endpoint da Atlas recebe a notificação assinada e responde 2xx. |
| CU-02 | **Cadastro.** Um integrador cadastra uma URL `https` informando que só quer `SHIPPED` e `DELIVERED`. Recebe a secret na criação ([09:31] Marcos, [09:33] Marcos). |
| CU-03 | **Cliente fora do ar.** O endpoint da Nova Cargo está em manutenção. O OMS tenta de novo em intervalos crescentes; ao voltar, o cliente recebe o evento pendente ([09:16] Diego). |
| CU-04 | **Entrega esgotada.** Após todas as tentativas sem sucesso, o evento fica guardado para reprocessamento. Um `ADMIN` o reenvia manualmente e a ação é registrada ([09:18] Diego, [09:36] Sofia). |
| CU-05 | **Secret vazada.** A MaxDistribuição vazou a secret num log. Ela gera uma nova pela API e tem 24 h para migrar os sistemas, com a antiga ainda válida ([09:21] Sofia, [09:22] Diego). |
| CU-06 | **Auditoria da integração.** O cliente consulta os últimos 100 envios (sucesso ou falha, payload, resposta, tempo) para diagnosticar um problema do lado dele ([09:34] Marcos). |
| CU-07 | **Evento duplicado.** O cliente recebe o mesmo evento duas vezes e o descarta pelo identificador de evento ([09:24] Diego). |
| CU-08 | **Status não assinado.** O pedido muda para um status que o cliente não escolheu; nada é enviado a ele ([09:33] Marcos, [09:34] Bruno). |

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta | Fonte |
| --- | --- | --- | --- | --- |
| PRD-OBJ-01 | Notificar o cliente em "tempo real" | Tempo entre o commit da mudança de status e a chamada ao endpoint, no caminho feliz | **< 10 s** (p95) | [09:02] Marcos; critério técnico CA-04 do FDD |
| PRD-OBJ-02 | Nunca perder uma mudança de status | Mudanças de status commitadas sem evento correspondente | **0** (atomicidade total) | [09:40] Bruno |
| PRD-OBJ-03 | Eliminar o polling nos clientes que pediram a feature | Clientes solicitantes com webhook ativo | **3 de 3** (Atlas, MaxDistribuição, Nova Cargo) *(derivada do pedido formal; a medição do volume residual de `GET /orders` é proposta deste PRD)* | [09:00] Marcos |
| PRD-OBJ-04 | Entregar no prazo comercial | Feature em produção, com revisão de segurança concluída | **Fim de novembro**, em até **3 sprints** | [09:45] Marcos, [09:47] Larissa |
| PRD-OBJ-05 | Absorver indisponibilidade real de clientes | Janela coberta antes de a entrega ser dada como falha | **≈ 15 h** entre a primeira falha e a última tentativa | [09:17] Diego |
| PRD-OBJ-06 | Não degradar a operação de pedidos | Falha ou lentidão de um endpoint de cliente afetando `PATCH /orders/:id/status` | **0 casos** | [09:04] Bruno |
| PRD-OBJ-07 | Sem custo adicional de infraestrutura | Novos serviços além de MySQL (um processo `worker` à parte) | **0 novos serviços** de infra | [09:07] Larissa, [09:07] Diego |

**Observação.** A reunião fixou explicitamente o limiar de 10 s, a janela de retry e o prazo. A meta de adoção 3 de 3 deriva do pedido formal dos três clientes; o percentual de redução de chamadas a `GET /orders` **não** foi definido na reunião e fica como acompanhamento pós-lançamento.

## 5. Escopo

### 5.1 Incluso

- Cadastro, edição, remoção e listagem de webhooks por customer, com escolha dos status a receber.
- Notificação de saída (`order.status_changed`) a cada mudança de status do pedido assinada pelo webhook.
- Assinatura de autenticidade em cada envio e rotação de secret com período de convivência.
- Novas tentativas automáticas em caso de falha, com armazenamento das entregas esgotadas.
- Reprocessamento manual dessas entregas por `ADMIN`, com auditoria.
- Histórico de entregas consultável pelo cliente.
- Documentação de integração para o portal do desenvolvedor (responsabilidade de Marcos).

### 5.2 Fora de escopo

Itens descartados ou adiados na reunião:

| Item | Situação | Fonte |
| --- | --- | --- |
| **Aviso por e-mail ao cliente quando o webhook falha repetidamente** | Adiado; "talvez próxima fase, depois de medir o impacto" | [09:37] Larissa, [09:38] Marcos |
| **Rate limiting de saída por cliente** | Adiado; "observar e decidir depois" | [09:39] Diego, [09:39] Larissa |
| **Dashboard visual para o cliente** | Descartado nesta fase; só endpoints, o painel é projeto do time de frontend | [09:40] Larissa |
| **Arquivamento de eventos entregues (~30 dias)** | Fora do escopo da feature | [09:08] Diego |
| **Escala para múltiplos workers / ordenação global** | Limitação conhecida; hoje ordem por pedido com um único worker | [09:13] Larissa, [09:14] Marcos |
| **Endurecimento das permissões do CRUD de webhooks** | Adiado; "mais pra frente" | [09:37] Sofia |
| **Webhooks de entrada (cliente envia para o OMS)** | Fora do escopo; somente saída | [09:02] Marcos, [09:03] Sofia |
| **Itens do pedido no payload** | Não enviados para manter o payload enxuto; o cliente consulta `GET /orders/:id` | [09:43] Diego |

## 6. Requisitos funcionais

Cada requisito tem ID `PRD-FR-xx`, usado no [Tracker](./TRACKER.md). A especificação de contrato e fluxo de cada um está no [FDD](./FDD.md).

| ID | Requisito | Fonte |
| --- | --- | --- |
| PRD-FR-01 | O cliente pode **cadastrar** um webhook informando URL e lista de status que deseja receber; a **secret é gerada pelo OMS** e devolvida na criação. | [09:31] Marcos, [09:21] Sofia |
| PRD-FR-02 | O cliente pode **editar** (`PATCH`), **remover** (`DELETE`) e **listar** (`GET`) os webhooks de um customer. O `customer_id` é informado no body ou no path, não extraído do JWT. | [09:33] Bruno, [09:32] Larissa |
| PRD-FR-03 | Cada webhook tem um **filtro de eventos**: lista dos status que ele quer ouvir. O filtro é aplicado quando o evento é registrado; se nenhum webhook do customer assina o status, nada é registrado. | [09:33] Marcos, [09:34] Bruno |
| PRD-FR-04 | Toda **mudança de status de pedido** gera um evento para cada webhook ativo que assina o novo status, e o evento é registrado **de forma atômica** com a mudança: se a mudança sofre rollback, o evento não existe; se ela commita, o evento existe. | [09:06] Diego, [09:40] Bruno |
| PRD-FR-05 | O OMS **envia o evento por HTTPS** ao endpoint cadastrado, de forma assíncrona à mudança de status, com **timeout de 10 s** por chamada. Resposta fora de 2xx ou estouro do timeout é falha. | [09:11] Diego, [09:42] Diego |
| PRD-FR-06 | O payload é JSON e contém `event_id`, `event_type` (`order.status_changed`), timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e `total_cents`. **Não inclui itens.** O payload é um **snapshot do momento da mudança**. | [09:43] Diego, [09:52] Larissa |
| PRD-FR-07 | Cada envio leva os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`. | [09:44] Diego, [09:44] Sofia |
| PRD-FR-08 | O envio é **assinado com HMAC-SHA256** sobre o corpo, com **secret única por endpoint**, para o cliente validar origem e integridade. | [09:20] Sofia, [09:22] Sofia |
| PRD-FR-09 | O cliente pode **rotacionar a secret** pela API; a anterior permanece válida por **24 h** e depois deixa de valer. | [09:21] Sofia |
| PRD-FR-10 | Em caso de falha, o OMS **tenta novamente** com intervalos crescentes de **1 min, 5 min, 30 min, 2 h e 12 h**, no máximo 5 retentativas. | [09:17] Larissa |
| PRD-FR-11 | Esgotadas as tentativas, o evento é **persistido numa tabela de entregas falhas (DLQ)** com payload, motivo e timestamp, sem ficar pendente para sempre. | [09:15] Diego, [09:18] Diego |
| PRD-FR-12 | Um usuário com role **`ADMIN`** pode **reprocessar** um evento da DLQ (`POST /admin/webhooks/dead-letter/:id/replay`); o OMS **registra em log quem executou**. Os demais papéis não podem. | [09:35] Diego, [09:36] Sofia, [09:36] Larissa |
| PRD-FR-13 | O cliente pode consultar o **histórico de entregas** de um webhook (`GET /webhooks/:id/deliveries`): sucesso ou falha, payload, resposta e tempo de resposta, das últimas 100 entregas. | [09:34] Marcos |
| PRD-FR-14 | A entrega tem semântica **at-least-once**: o mesmo evento pode chegar mais de uma vez, sempre com o mesmo `X-Event-Id` para deduplicação pelo cliente. | [09:24] Diego, [09:26] Larissa |
| PRD-FR-15 | Eventos do **mesmo pedido** são entregues na **ordem** em que ocorreram (garantia por `order_id`, com um único worker). | [09:12] Diego, [09:13] Larissa |
| PRD-FR-16 | A URL do webhook **deve ser `https`**; `http` é recusado com erro de validação. | [09:23] Sofia |

## 7. Requisitos não funcionais

| ID | Requisito | Categoria | Fonte |
| --- | --- | --- | --- |
| PRD-NFR-01 | **Latência:** notificação em menos de 10 s no caminho feliz; o polling do worker acrescenta no máximo 2 s. | Desempenho | [09:02] Marcos, [09:10] Larissa |
| PRD-NFR-02 | **Isolamento:** falha ou lentidão do cliente não pode bloquear nem desfazer mudanças de status; o envio roda em **processo separado** da API. | Disponibilidade | [09:04] Bruno, [09:11] Diego |
| PRD-NFR-03 | **Consistência:** o registro do evento participa da mesma transação da mudança de status. | Integridade | [09:40] Bruno, [09:41] Diego |
| PRD-NFR-04 | **Segurança de transporte:** somente URLs `https`. | Segurança | [09:23] Sofia |
| PRD-NFR-05 | **Segurança de autenticidade:** HMAC-SHA256, secret por endpoint, rotação com 24 h de convivência. | Segurança | [09:22] Sofia |
| PRD-NFR-06 | **Controle de acesso:** replay restrito a `ADMIN`, com auditoria; demais operações exigem usuário autenticado. | Segurança | [09:36] Sofia, [09:37] Sofia |
| PRD-NFR-07 | **Revisão de segurança:** mínimo de 2 dias úteis da Sofia, sobre HMAC e geração de secret, antes do deploy. | Segurança / processo | [09:46] Sofia |
| PRD-NFR-08 | **Tamanho de payload:** máximo de 64 KB; acima disso o OMS retorna erro e não envia truncado. | Limite | [09:23] Sofia, [09:24] Larissa |
| PRD-NFR-09 | **Timeout** de 10 s por chamada HTTP. | Resiliência | [09:42] Diego |
| PRD-NFR-10 | **Resiliência a falhas do cliente:** janela de retry de ≈ 15 h antes da entrega ser dada como falha. | Resiliência | [09:17] Diego |
| PRD-NFR-11 | **Escalabilidade (limitação conhecida):** um único worker; ordem garantida só por pedido, não global. | Limitação | [09:13] Larissa |
| PRD-NFR-12 | **Sem nova infraestrutura:** apenas MySQL, Prisma, Express, Zod e Pino já existentes; um processo `worker` adicional. | Operação | [09:07] Diego, [09:30] Larissa |
| PRD-NFR-13 | **Consistência com o projeto:** módulo no padrão dos demais, erros `AppError` com prefixo `WEBHOOK_`, logs Pino e error middleware centralizado. | Manutenibilidade | [09:27] Bruno, [09:29] Larissa, [09:30] Larissa |
| PRD-NFR-14 | **Compatibilidade:** a API de pedidos existente não muda de contrato. | Compatibilidade | `src/modules/orders/order.service.ts`; [FDD §11](./FDD.md) |
| PRD-NFR-15 | **Identificadores:** UUID em todas as chaves, como no resto do banco. | Consistência | [09:51] Larissa |

## 8. Decisões e trade-offs principais

Cada decisão foi fechada na reunião e está registrada em um ADR. Aqui ficam o impacto para o produto e o que se abriu mão.

| Decisão | O que ganhamos | O que aceitamos em troca | ADR |
| --- | --- | --- | --- |
| **Outbox no MySQL**, em vez de disparo síncrono ou fila externa | Consistência total entre status e evento; zero infraestrutura nova; time pequeno consegue operar | Uma escrita a mais na transação de mudança de status; o evento vive no mesmo banco dos pedidos | [ADR-001](./adrs/ADR-001-outbox-no-mysql.md) |
| **Worker separado com polling de 2 s**, em vez de reação imediata | Atende o limite de 10 s; a API reiniciar não derruba o envio | Até 2 s de atraso no pior caso (aceito pelo PM); um processo a mais para operar | [ADR-002](./adrs/ADR-002-worker-separado-em-polling.md) |
| **5 retentativas com backoff e DLQ**, em vez de 3 tentativas ou retry infinito | Cobre manutenções planejadas de ≈ 2 h e quedas longas; nada fica pendente para sempre | Cliente fora do ar por mais de ≈ 15 h perde a entrega automática e depende de replay manual | [ADR-003](./adrs/ADR-003-retry-com-backoff-e-dlq.md) |
| **HMAC-SHA256 com secret por endpoint** e rotação de 24 h | Vazamento de uma secret não compromete as outras; migração sem janela de corte | Mais responsabilidade de segurança do lado do cliente (guardar e validar a secret) | [ADR-004](./adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md) |
| **At-least-once com `X-Event-Id`**, em vez de exactly-once | Simplicidade; é o padrão de mercado (Stripe, GitHub) | O cliente precisa deduplicar; duplicatas são possíveis | [ADR-005](./adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| **Reuso dos padrões do projeto** | Entrega mais rápida e menor custo de manutenção | Segue as convenções existentes, mesmo onde haveria alternativa mais enxuta | [ADR-006](./adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) |

**Trade-offs de produto que merecem atenção:**

- **Responsabilidade do cliente.** Duplicatas e validação de assinatura ficam com o cliente. Por isso a documentação no portal é parte da entrega, não um complemento ([09:26] Marcos).
- **Falha silenciosa.** Sem aviso por e-mail nesta fase, o cliente só descobre uma entrega esgotada consultando o histórico ou pelo contato da operação.
- **Prazo × escopo.** O prazo de fim de novembro só cabe em 3 sprints porque e-mail, dashboard e rate limiting ficaram de fora.

## 9. Dependências

| Dependência | Tipo | Situação |
| --- | --- | --- |
| **Revisão de segurança da Sofia** (≥ 2 dias úteis, antes do deploy) | Processo | Precisa ser **agendada**; sem ela não há deploy ([09:46] Sofia, [09:49] Sofia) |
| **Documentação no portal do desenvolvedor** (integração, deduplicação por `X-Event-Id`, validação de assinatura) | Entrega de Produto | Responsabilidade de Marcos ([09:26] Marcos, [09:40] Marcos) |
| **Comunicação com os clientes** e confirmação do prazo com a Atlas | Negócio | Marcos ([09:47] Marcos, [09:49] Marcos) |
| **Sessão de revisão do design** com Bruno e Diego antes de iniciar a implementação | Processo | Larissa ([09:50] Larissa) |
| **`OrderService.changeStatus`** (`src/modules/orders/order.service.ts`) | Código existente | Único ponto de domínio alterado; deve manter o contrato atual |
| **Usuários do OMS que representam os clientes** (JWT do próprio sistema) | Código existente | Clientes cadastram pela API, autenticados ([09:32] Marcos) |
| **`requireRole`, `AppError`, error middleware, Pino** | Código existente | Reaproveitados sem alteração ([09:29] Bruno, [09:36] Larissa) |
| **MySQL 8.0 e Node ≥ 20** | Infraestrutura | Já em uso (`docker-compose.yml`, `package.json`) |
| **Um processo `worker` em produção** | Operação | Novo; deploy depois da migration e **uma única réplica** ([09:11] Larissa, [09:13] Larissa) |

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- | --- |
| PRD-R-01 | **Prazo apertado:** 3 sprints para o fim de novembro, com a Atlas ameaçando migrar | Média | Alto | Escopo já cortado (e-mail, dashboard, rate limiting); ordem de entrega definida: modelagem → worker/retry → CRUD/histórico → integração e testes ([09:46] Larissa) |
| PRD-R-02 | **Cliente lento atrasa as entregas de outros** e quebra o limite de 10 s | Média | Alto | Timeout fixo de 10 s e envio concorrente entre pedidos distintos; alerta de idade do evento mais antigo pendente ([FDD R-02](./FDD.md)) |
| PRD-R-03 | **Worker parado acumula eventos** e o cliente deixa de receber notificações | Média | Alto | Eventos ficam seguros no banco e são enviados ao retomar; alerta de último ciclo do worker ([FDD R-04](./FDD.md)) |
| PRD-R-04 | **Vazamento de secret** por um cliente | Média | Alto | Secret única por endpoint, rotação com 24 h, secret fora dos logs e revisão de segurança antes do deploy ([09:22] Diego, [09:46] Sofia) |
| PRD-R-05 | **Cliente processa o evento em duplicidade** | Alta | Médio | `X-Event-Id` estável entre tentativas e documentação destacada no portal ([09:26] Marcos) |
| PRD-R-06 | **Falha na gravação do evento impede a mudança de status** | Baixa | Alto | Comportamento intencional: a atomicidade é a garantia central; coberto por testes de rollback ([09:40] Bruno) |
| PRD-R-07 | **Mais de um worker em produção** quebra a ordenação por pedido | Baixa | Médio | Limitação documentada; deploy com réplica única ([09:13] Larissa) |
| PRD-R-08 | **Cliente não percebe entregas esgotadas**, por falta de aviso | Média | Médio | Alerta interno sobre eventos na DLQ; aviso por e-mail fora desta fase ("talvez" na próxima, depois de medir o impacto) ([09:37] Larissa) |
| PRD-R-09 | **Rajada de chamadas** quando muitos pedidos de um cliente mudam juntos | Média | Baixo | Rate limiting adiado; acompanhar o volume e decidir depois ([09:39] Larissa) |

Os riscos técnicos detalhados (R-01 a R-12) estão no [FDD](./FDD.md). A proteção contra URLs que apontem para a rede interna (SSRF) **não foi discutida na reunião** e foi levada como ponto para a revisão de segurança ([FDD, P-09](./FDD.md)).

## 11. Critérios de aceitação

A feature é aceita quando todos os critérios abaixo forem verdadeiros. A versão verificável, item a item, está no [FDD §12](./FDD.md).

| ID | Critério |
| --- | --- |
| PRD-CA-01 | Uma mudança de status de um pedido cujo customer tem webhook assinando o novo status gera a notificação, e **nenhuma** mudança de status commitada fica sem evento (PRD-OBJ-02). |
| PRD-CA-02 | Se o registro do evento falhar, a mudança de status **não é aplicada** (pedido, histórico e estoque permanecem inalterados). |
| PRD-CA-03 | Mudança para um status que nenhum webhook assina **não gera** evento nem envio. |
| PRD-CA-04 | No caminho feliz, o endpoint do cliente é chamado em **menos de 10 s** (p95) após a mudança de status (PRD-OBJ-01). |
| PRD-CA-05 | O cliente consegue validar a assinatura do corpo com a secret e recebe os 5 headers definidos. |
| PRD-CA-06 | Endpoint que falha recebe novas tentativas nos intervalos 1 min, 5 min, 30 min, 2 h e 12 h; depois, o evento aparece na DLQ. |
| PRD-CA-07 | Endpoint que não responde em 10 s é tratado como falha e reagendado. |
| PRD-CA-08 | Cadastro com URL `http` é recusado; com `https` é aceito e devolve a secret **apenas na criação e na rotação**. |
| PRD-CA-09 | Após rotacionar a secret, a anterior e a nova validam por 24 h; depois, só a nova. |
| PRD-CA-10 | Replay por `OPERATOR` é negado; por `ADMIN` funciona, reaproveita o mesmo `X-Event-Id` e registra quem executou. |
| PRD-CA-11 | O cliente consulta o histórico das últimas 100 entregas com sucesso/falha, payload, resposta e tempo. |
| PRD-CA-12 | Eventos de um mesmo pedido chegam na ordem em que ocorreram (um único worker, sem falhas). |
| PRD-CA-13 | Payload acima de 64 KB resulta em erro e não é enviado truncado. |
| PRD-CA-14 | Os testes existentes de pedidos continuam passando sem alterar suas expectativas (PRD-NFR-14). |
| PRD-CA-15 | A **revisão de segurança da Sofia** (≥ 2 dias úteis) está concluída **antes** do deploy. |
| PRD-CA-16 | A documentação de integração (deduplicação e validação de assinatura) está disponível no portal para os clientes. |

## 12. Estratégia de testes e validação

A estratégia detalhada está no [FDD §12](./FDD.md); aqui estão os níveis de validação sob a ótica de produto.

| Nível | O que valida | Cobre |
| --- | --- | --- |
| **Testes de integração da API** (padrão de `tests/orders.test.ts`, com banco real) | Cadastro, edição, remoção, listagem, rotação de secret, histórico de entregas, controle de acesso do replay | PRD-FR-01 a 03, 09, 12, 13, 16; PRD-CA-08 a 11 |
| **Testes de atomicidade** | Mudança de status com e sem webhook assinando; rollback quando o registro do evento falha; payload acima de 64 KB | PRD-FR-03, 04; PRD-NFR-03, 08; PRD-CA-01 a 03, 13 |
| **Testes do worker** (servidor HTTP local simulando 2xx, 500 e atraso maior que o timeout; relógio injetado para o backoff) | Envio, assinatura, retry nos intervalos corretos, DLQ, timeout, ordem por pedido, recuperação após queda do worker | PRD-FR-05, 07, 08, 10, 11, 14, 15; PRD-CA-04 a 07, 12 |
| **Testes unitários** | Assinatura HMAC e política de retry | PRD-FR-08, 10 |
| **Teste de regressão** | Suíte existente de pedidos permanece verde; `lint`, `build` e `test` passam | PRD-NFR-14; PRD-CA-14 |
| **Validação de segurança** | Revisão manual de HMAC, geração de secret e ausência de secret em logs | PRD-NFR-05, 07; PRD-CA-15 |
| **Validação com clientes** | Ambiente de teste com Atlas, MaxDistribuição e Nova Cargo recebendo notificações reais antes do go-live; confirmação do recebimento em menos de 10 s *(proposta deste PRD: a reunião não define o piloto)* | PRD-OBJ-01, 03 |
| **Acompanhamento pós-lançamento** | Tempo de entrega, volume de eventos na DLQ e idade do evento pendente mais antigo, para decidir sobre rate limiting e e-mail | PRD-OBJ-01; itens adiados da seção 5.2 |

**Critério de saída:** todos os critérios de aceitação da seção 11 atendidos, suíte verde e revisão de segurança concluída.

## 13. Questões em aberto

Adiadas na reunião e acompanhadas no [RFC §5](./RFC.md): rate limiting de saída (Diego/Larissa), e-mail ao cliente em caso de falhas (Marcos), escala para múltiplos workers (Diego), arquivamento de eventos entregues (Diego), endurecimento de permissões do CRUD (Sofia) e dashboard visual (Marcos). Pontos de implementação sem decisão na reunião estão no [FDD §13](./FDD.md).
