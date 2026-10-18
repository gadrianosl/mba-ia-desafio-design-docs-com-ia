# ADR-004 — At-least-once e identidade do evento

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

O cliente pode receber o evento e a confirmação se perder antes de o worker persistir o sucesso.

## Decisão

Adotar at-least-once com UUID em X-Event-Id, gerado ao entrar na outbox. Reutilizar o identificador nas retentativas; cliente deduplica. Explicitar a possibilidade de duplicatas na integração.

## Alternativas Consideradas

Exactly-once foi rejeitado por exigir coordenação dos dois lados e maior complexidade ([09:25] Diego).

## Consequências

Positivas: recuperação com duplicatas identificáveis. Negativas: o consumidor precisa deduplicar e a DLQ pode terminar sem entrega. O trade-off prioriza recuperação sobre eliminação de duplicatas; at-least-once não é promessa de sucesso eterno apesar do limite de retries.

## Integração e limites

Identidade em fan-out e replay não foi detalhada na call. O FDD apresenta uma proposta derivada, sujeita a revisão, que mantém o event_id do evento e identifica a entrega separadamente.

## Rastreabilidade

ID: ADR-004. Fontes: [09:24] Diego; [09:25] Diego; [09:26] Larissa. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
