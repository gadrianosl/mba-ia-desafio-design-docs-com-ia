# ADR-001 — Outbox no MySQL

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

O status do pedido não pode depender da disponibilidade do receptor. O OMS já mantém pedido, histórico e estoque em transação MySQL.

## Decisão

Persistir o evento na webhook_outbox dentro da mesma transação de changeStatus. Se a inserção falhar, reverter toda a transação. Não fazer chamadas HTTP nessa transação.

## Alternativas Consideradas

Envio HTTP síncrono foi descartado por bloquear a transação e acoplar o OMS à indisponibilidade do cliente ([09:04] Bruno). Redis Streams foi descartado pela infraestrutura adicional para um time pequeno ([09:07] Larissa e Diego).

## Consequências

Positivas: atomicidade entre alteração e registro do evento; reuso do MySQL. Negativas: mais escrita e leitura no banco transacional; exige worker e manutenção da tabela. O trade-off troca infraestrutura adicional por carga no banco existente.

## Integração e limites

Integração: `src/modules/orders/order.service.ts`, método `changeStatus`, já usa `Prisma.TransactionClient`; `prisma/schema.prisma` confirma MySQL. A função proposta na reunião é `publishWebhookEvent(tx, order, fromStatus, toStatus)`.

## Rastreabilidade

ID: ADR-001. Fontes: [09:06] Diego; [09:07] Diego; [09:40] Bruno; [09:41] Diego. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
