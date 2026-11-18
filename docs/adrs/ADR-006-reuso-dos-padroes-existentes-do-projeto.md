# ADR-006: Reuso dos padrões existentes do projeto no módulo de webhooks

## Status

Aceito (17-11-2026). Decisão fechada em [09:30] Larissa.

## Contexto

A aplicação já tem convenções consolidadas: módulos por domínio, hierarquia de erros, logger centralizado e middlewares compartilhados. A feature de webhooks é nova e poderia introduzir padrões próprios, aumentando a carga de manutenção de um time pequeno.

## Decisão

Reutilizar o máximo do que já existe ([09:30] Larissa):

- **Estrutura de módulo:** novo `src/modules/webhooks` com controller, service, repository, routes e schemas, como os demais módulos em `src/modules/` ([09:27] Bruno). As rotas são registradas em `src/routes/index.ts`.
- **Erros:** classes derivadas de `AppError` (`src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`), como `InsufficientStockError` e `InvalidStatusTransitionError`, com códigos prefixados por **`WEBHOOK_`** (por exemplo `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`) ([09:28] Bruno, [09:29] Larissa).
- **Tratamento de erro:** `src/middlewares/error.middleware.ts` já trata `AppError`, Zod e Prisma, sem alterações ([09:29] Bruno).
- **Logger:** Pino em `src/shared/logger/index.ts`; nenhuma biblioteca nova ([09:29] Bruno).
- **Validação:** schemas Zod no padrão dos módulos, com `src/middlewares/validate.middleware.ts`.
- **Autorização:** `requireRole('ADMIN')` de `src/middlewares/auth.middleware.ts` no replay da DLQ; o CRUD de configuração exige apenas autenticação ([09:36] Larissa, [09:37] Sofia).
- **Acesso a dados e IDs:** Prisma e MySQL existentes, chaves UUID ([09:51] Larissa).
- **Integração com pedidos:** extensão de `changeStatus` em `src/modules/orders/order.service.ts` via `publishWebhookEvent(tx, ...)` ([ADR-001](./ADR-001-outbox-no-mysql.md)).

## Alternativas Consideradas

1. **Módulo com padrões próprios** (hierarquia de erros, logger ou formato de resposta distintos). Descartada: duplicaria convenções e aumentaria a manutenção.
2. **Serviço separado para webhooks.** Não discutida em detalhe; contrariada pela decisão de manter mesma stack, mesmo banco e mesmo Prisma ([09:11] Diego, Bruno).

## Consequências

**Positivas**
- Curva de aprendizado mínima e comportamento consistente de erros e logs.
- Menos código novo e nenhuma dependência adicional.

**Negativas / trade-offs**
- O módulo herda limitações dos padrões atuais.
- `OrderService` passa a depender do módulo de webhooks via `publishWebhookEvent`, criando acoplamento entre domínios.
- O CRUD aberto a qualquer role autenticada é provisório ([09:37] Sofia: "por enquanto").
