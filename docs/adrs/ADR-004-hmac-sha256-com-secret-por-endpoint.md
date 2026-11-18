# ADR-004: Assinatura HMAC-SHA256 com secret por endpoint e rotação com grace period

## Status

Aceito (17-11-2026). Decisão fechada em [09:22] Sofia.

## Contexto

Os eventos carregam dados de pedidos para um endpoint fora da nossa infraestrutura. O cliente precisa validar que a requisição veio de nós e que o payload não foi adulterado ([09:19] Sofia). Já houve cliente que vazou uma secret em log de aplicação ([09:22] Diego).

## Decisão

- Assinar o **corpo do request** com **HMAC-SHA256**, enviando a assinatura no header `X-Signature` ([09:20] Sofia).
- Cada endpoint de webhook tem **sua própria secret**, gerada pela plataforma e devolvida na criação; nunca uma secret global ([09:21] Sofia, [09:31] Marcos).
- A secret é **rotacionável** por endpoint da API. Após a rotação, a antiga continua válida por **24 horas** e depois expira ([09:21] Sofia).
- Headers enviados: `X-Event-Id`, `X-Signature`, `X-Timestamp` (permite ao cliente detectar replay) e `X-Webhook-Id`, além de `Content-Type: application/json` ([09:44] Diego, Sofia).
- URL `https` obrigatória (validação no schema Zod) e payload limitado a 64 KB, com erro se exceder; ambos tratados como requisitos e não como decisão arquitetural ([09:23] Sofia, [09:24] Larissa).

## Alternativas Consideradas

1. **Secret global da plataforma.** Descartada: o vazamento de uma comprometeria todos os clientes ([09:21] Sofia).
2. **Rotação com corte imediato.** Descartada implicitamente: o grace period de 24 h dá tempo para o cliente migrar seus sistemas ([09:21] Sofia).

## Consequências

**Positivas**
- Autenticidade e integridade verificáveis com bibliotecas padrão de mercado.
- Vazamento de uma secret afeta um único endpoint.

**Negativas / trade-offs**
- É preciso armazenar secrets utilizáveis (não só hash) e guardar duas secrets durante a rotação.
- O cliente precisa implementar a verificação.
- Código de HMAC e geração de secret exige revisão de segurança antes do deploy (mínimo de dois dias úteis reservados pela Sofia).
