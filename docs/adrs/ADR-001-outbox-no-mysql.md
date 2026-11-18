# ADR-001: Padrão Outbox no MySQL para publicação de eventos de webhook

## Status

Aceito (17-11-2026). Decisão fechada na reunião técnica de webhooks ([09:08] Larissa).

## Contexto

Clientes B2B (Atlas Comercial, MaxDistribuição, Nova Cargo) querem ser notificados em menos de 10 segundos quando o status de um pedido muda ([09:02] Marcos). Hoje fazem polling em `GET /orders`.

O método `changeStatus` em `src/modules/orders/order.service.ts` já executa uma transação pesada: atualiza `orders`, insere em `order_status_history` e ajusta `stock_quantity`. Uma chamada HTTP dentro dela travaria mudanças de status por causa de clientes lentos, e um cliente fora do ar forçaria rollback da mudança ([09:04] Bruno). É preciso garantir que não exista caso de status alterado sem evento, nem evento sem status alterado.

## Decisão

Adotar o padrão **Outbox** no MySQL existente:

- Na mesma transação de `changeStatus`, inserir uma linha em `webhook_outbox`. Se a inserção falhar, a transação inteira sofre rollback ([09:40] Bruno, [09:41] Diego).
- A inserção é feita pela função `publishWebhookEvent(tx, order, fromStatus, toStatus)`, que recebe o cliente `tx` da transação corrente. Nenhum repository é injetado no `OrderService` ([09:41] Bruno).
- O payload é um **snapshot renderizado no momento da inserção**, não apenas o `order_id` ([09:52] Larissa).
- O filtro por status assinado pelo webhook é aplicado **na inserção**; se nenhum webhook do customer quer o status, nada é inserido ([09:34] Bruno).
- A chave primária é UUID, como no resto do projeto ([09:51] Larissa).
- A tabela tem índices em status (pendente, processando, falhou, entregue) e em `created_at` ([09:08] Diego). Arquivar linhas entregues fica fora do escopo.

## Alternativas Consideradas

1. **Disparo HTTP síncrono dentro de `changeStatus`.** Descartada: cliente lento bloqueia outras mudanças e cliente offline obrigaria rollback ([09:04] Bruno).
2. **Fila externa (Redis Streams ou similar).** Descartada: exige subir mais infraestrutura e é excessiva para um time pequeno ([09:07] Larissa, Diego).
3. **Gravar só `order_id` e renderizar o payload no envio.** Descartada: o evento deixaria de refletir o estado do pedido no momento da mudança ([09:52] Larissa).

## Consequências

**Positivas**
- Consistência atômica: se a transação commitou, o evento existe; se deu rollback, ele some junto.
- Sem nova infraestrutura; reaproveita MySQL e Prisma (`prisma/schema.prisma`, `src/config/database.ts`).
- O snapshot torna o evento imutável e fácil de reenviar.

**Negativas / trade-offs**
- `changeStatus` fica acoplado à inserção na outbox, e a transação cresce.
- A tabela cresce e precisa de arquivamento futuro.
- A latência passa a depender do polling do worker (ver [ADR-002](./ADR-002-worker-separado-em-polling.md)).
