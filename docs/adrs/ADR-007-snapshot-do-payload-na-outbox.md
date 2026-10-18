# ADR-007 — Snapshot do payload na outbox

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

Um pedido pode mudar novamente antes de o worker entregar o evento anterior.

## Decisão

Persistir o payload já renderizado na inserção da outbox. Incluir event_id, event_type order.status_changed, timestamp ISO 8601, order_id, order_number, from_status, to_status, customer_id e total_cents. Não enviar items. IDs seguem UUID.

## Alternativas Consideradas

Guardar só order_id e renderizar no envio foi discutido e descartado porque produziria um retrato posterior, inconsistente com o evento ([09:51] Bruno; [09:52] Larissa).

## Consequências

Positivas: conteúdo estável em retries e representação do momento da transição. Negativas: mais armazenamento e dados duplicados. O trade-off troca espaço por fidelidade histórica e assinatura reproduzível.

## Integração e limites

O limite de 64KB é confirmado em [09:24] Larissa. Serialização em bytes, tamanho exato e persistência do corpo são detalhamentos derivados propostos no FDD, sem introduzir novos campos de negócio.

## Rastreabilidade

ID: ADR-007. Fontes: [09:43] Diego; [09:51] Larissa; [09:52] Larissa; [09:52] Diego. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
