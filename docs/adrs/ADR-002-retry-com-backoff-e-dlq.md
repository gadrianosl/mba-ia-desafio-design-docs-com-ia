# ADR-002 — Retry com backoff e DLQ

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

Receptores podem ficar indisponíveis por horas. Eventos não podem permanecer em retry indefinidamente.

## Decisão

Usar retry limitado, progressão 1m/5m/30m/2h/12h e DLQ na tabela webhook_dead_letter, com payload, motivo e timestamp. Replay manual por POST /admin/webhooks/dead-letter/:id/replay, restrito a ADMIN e auditado.

## Alternativas Consideradas

Retry indefinido foi descartado por acumular eventos sem solução ([09:15] Diego). Três tentativas foram rejeitadas por não cobrir manutenção de duas horas ([09:16] Diego). Marcar falha na própria outbox foi contraposto à tabela separada, escolhida para leitura limpa e evidência ([09:17] Larissa; [09:18] Diego).

## Consequências

Positivas: tolera falhas temporárias e conserva evidência para recuperação. Negativas: entrega pode atrasar horas e exige operação manual após esgotamento. O trade-off limita o custo de retry e transfere recuperação tardia para a operação.

## Integração e limites

Ambiguidade preservada: a reunião confirma “5 tentativas” e cinco esperas, cuja soma é 14h36. Cinco chamadas totais só comportam quatro esperas. Resolver em RFC-OPEN-01 antes de implementar o contador; não afirmar que seis chamadas foram aprovadas.

## Rastreabilidade

ID: ADR-002. Fontes: [09:15] Diego; [09:16] Bruno; [09:17] Larissa; [09:18] Diego; [09:36] Sofia. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
