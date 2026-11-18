# ADR-002: Worker em processo separado com polling de 2 segundos

## Status

Aceito (17-11-2026). Decisão fechada em [09:10] Larissa e [09:11] Diego.

## Contexto

Os eventos gravados em `webhook_outbox` ([ADR-001](./ADR-001-outbox-no-mysql.md)) precisam ser lidos e enviados por HTTP. O requisito de produto é entrega abaixo de 10 segundos ([09:02] Marcos). O MySQL não tem listener nativo equivalente ao `NOTIFY/LISTEN` do Postgres, e triggers só executam SQL, sem avisar um processo externo ([09:09] Diego). Se o worker rodasse dentro da API, um restart da API o derrubaria ([09:11] Diego).

## Decisão

- O worker faz **polling a cada 2 segundos**, buscando em lote pequeno os eventos pendentes mais antigos, processando-os e marcando o resultado.
- O worker roda como **processo separado**, com nova entry-point `src/worker.ts` (análoga a `src/server.ts`) e script `npm run worker` ([09:11] Larissa).
- A lógica de processamento fica no módulo, em `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts` ([09:28] Bruno).
- Usa o mesmo banco e a mesma `DATABASE_URL`, mas **uma instância própria do `PrismaClient`**, pois o cliente é por processo ([09:30] Bruno).
- Execução em **single-worker**. A ordenação por `created_at` na outbox dá ordem implícita por `order_id`; não há garantia de ordering global ([09:12] Diego, [09:13] Larissa).
- Timeout de 10 segundos por chamada HTTP; estouro conta como falha e vai para retry ([09:42] Diego).

## Alternativas Consideradas

1. **Trigger de banco ou mecanismo de notificação.** Descartada: não notifica processo externo, exigiria improviso (arquivo ou endpoint) ([09:09] Diego).
2. **Worker dentro do processo da API.** Descartada: restart da API perde o worker ([09:11] Diego).
3. **Múltiplos workers em paralelo.** Adiada: perderia a ordenação; seria necessário particionar por `order_id` ou usar lock pessimista ([09:13] Diego).

## Consequências

**Positivas**
- Simples, sem infraestrutura nova; o pior caso de latência mínima (2 s) atende ao limite de 10 s.
- Isolamento: a API não é afetada por lentidão de clientes.

**Negativas / trade-offs**
- Latência mínima de até 2 s e carga constante de leitura no banco.
- Single-worker: sem escala horizontal e sem ordering global, registrado como **limitação conhecida**.
- Mais um processo para operar e monitorar.
