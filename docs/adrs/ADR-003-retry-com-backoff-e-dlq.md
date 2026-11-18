# ADR-003: Política de retry com backoff exponencial e DLQ em tabela separada

## Status

Aceito (17-11-2026). Decisão fechada em [09:17] Larissa.

## Contexto

Endpoints de clientes podem ficar indisponíveis. Já houve cliente com duas horas de manutenção planejada ([09:16] Diego). É preciso uma política que cubra janelas longas sem manter eventos pendentes para sempre.

## Decisão

- **5 tentativas** com backoff exponencial de **1 min, 5 min, 30 min, 2 h e 12 h** (cerca de 15 horas entre a primeira falha e a última tentativa) ([09:17] Larissa).
- Esgotadas as tentativas, o evento é considerado falha permanente e movido para a tabela **`webhook_dead_letter`**, com payload, motivo da falha e timestamp ([09:18] Diego).
- O reprocessamento é manual por `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente ([09:18] Diego).
- O replay exige role `ADMIN`, reutilizando o middleware `requireRole` de `src/middlewares/auth.middleware.ts`, e registra quem o executou para auditoria ([09:36] Sofia, Larissa).

## Alternativas Consideradas

1. **Retry indefinido com backoff.** Descartada: eventos ficam pendurados se o cliente desaparecer ([09:15] Diego).
2. **3 tentativas.** Descartada: cobriria só cerca de 30 minutos e mataria o evento em indisponibilidades maiores ([09:16] Diego).
3. **Marcar como "failed" na própria outbox.** Descartada em favor de tabela separada, que mantém a leitura da outbox limpa e serve de evidência para debug ([09:18] Diego).

## Consequências

**Positivas**
- Cobre indisponibilidades de várias horas; DLQ preserva evidência e permite reprocessamento.
- A outbox principal permanece pequena.

**Negativas / trade-offs**
- Um cliente fora por mais de cerca de 15 h perde a entrega automática e depende de replay manual.
- A progressão fixa não é configurável por cliente.
- Replay manual é trabalho operacional; notificação por e-mail ao cliente ficou **fora de escopo** ([09:37] Larissa).
