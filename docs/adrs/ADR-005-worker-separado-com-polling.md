# ADR-005 — Worker separado com polling

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

Envio assíncrono precisa consumir a outbox com a stack atual e sem infraestrutura nova.

## Decisão

Criar entrypoint futuro src/worker.ts, separado da API, com polling de 2 segundos, batches pequenos dos pendentes mais antigos e PrismaClient próprio por processo, usando a mesma DATABASE_URL. Operar inicialmente um único worker.

## Alternativas Consideradas

Trigger/listener de banco foi descartado porque trigger MySQL não notifica nativamente processo externo ([09:09] Diego). Worker no processo da API foi descartado pelo acoplamento aos reinícios ([09:11] Diego). Múltiplos workers e particionamento foram adiados ([09:13] Diego).

## Consequências

Positivas: implantação simples e ciclo de vida separado da API. Negativas: polling adiciona espera e single-worker limita vazão. O trade-off reduz complexidade inicial, aceitando limitação de escala. O intervalo de 2s não garante sozinho entrega em menos de 10s se houver backlog ou receptor lento.

## Integração e limites

A intenção aprovada é ordem por order_id, sem ordem global. A regra durante retry e empate de created_at não foi especificada; RFC-OPEN-02 pede validação antes de prometer essa propriedade.

## Rastreabilidade

ID: ADR-005. Fontes: [09:08] Diego; [09:09] Diego; [09:10] Larissa; [09:11] Diego; [09:12] Diego; [09:13] Larissa; [09:30] Bruno. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
