# ADR-005: Garantia de entrega at-least-once com X-Event-Id para deduplicação

## Status

Aceito (17-11-2026). Decisão fechada em [09:26] Larissa.

## Contexto

Com outbox, worker em polling e retry ([ADR-001](./ADR-001-outbox-no-mysql.md), [ADR-002](./ADR-002-worker-separado-em-polling.md), [ADR-003](./ADR-003-retry-com-backoff-e-dlq.md)), um evento pode ser entregue e a confirmação se perder (timeout, falha de rede), levando a novo envio. É preciso definir a semântica de entrega.

## Decisão

- Garantir **at-least-once**: o cliente pode receber o mesmo evento mais de uma vez ([09:24] Diego).
- Cada evento recebe um **UUID gerado ao entrar na outbox**, enviado no header `X-Event-Id`. O cliente deduplica por esse identificador ([09:25] Diego).
- O payload inclui `event_id`, `event_type` (`order.status_changed`), timestamp ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos como `total_cents`, sem itens ([09:43] Diego).
- A responsabilidade de deduplicação é do cliente e será documentada em destaque no portal de desenvolvedor ([09:26] Marcos).

## Alternativas Consideradas

1. **Exactly-once.** Descartada: exige coordenação dos dois lados e é muito mais complexa ([09:25] Diego).
2. **At-most-once (sem retry).** Implicitamente descartada: contradiz a política de retry e perderia eventos.

## Consequências

**Positivas**
- Nenhum evento é perdido enquanto houver tentativas; segue o padrão de mercado (Stripe, GitHub).
- Implementação simples, sem coordenação distribuída.

**Negativas / trade-offs**
- Duplicatas são possíveis e a responsabilidade é transferida ao cliente ([09:25] Sofia).
- Sem garantia de ordering global; ordem apenas por `order_id` enquanto houver um único worker.
