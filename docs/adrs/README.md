# Architectural Decision Records

Decisões arquiteturais da feature de Sistema de Webhooks de Notificação de Pedidos, nomeadas `ADR-NNN-titulo-em-kebab-case.md`.

| ADR | Decisão |
| --- | --- |
| [ADR-001](./ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL |
| [ADR-002](./ADR-002-worker-separado-em-polling.md) | Worker em processo separado, em polling |
| [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff e DLQ |
| [ADR-004](./ADR-004-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint |
| [ADR-005](./ADR-005-entrega-at-least-once-com-x-event-id.md) | At-least-once com `X-Event-Id` |
| [ADR-006](./ADR-006-reuso-dos-padroes-existentes-do-projeto.md) | Reuso dos padrões existentes |
