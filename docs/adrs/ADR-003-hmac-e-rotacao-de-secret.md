# ADR-003 — HMAC e rotação de secret

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

O receptor precisa verificar origem e integridade de dados de pedidos enviados para fora da infraestrutura.

## Decisão

Assinar o corpo com HMAC-SHA256 em X-Signature. Cada endpoint tem secret própria, gerada pela plataforma e devolvida na criação. Permitir rotação mantendo a antiga válida por 24 horas. Exigir HTTPS.

## Alternativas Consideradas

Secret global é a alternativa explicitamente rejeitada: seu vazamento comprometeria todos os clientes ([09:21] Sofia).

## Consequências

Positivas: integridade verificável e impacto de vazamento limitado ao endpoint. Negativas: distribuição, armazenamento e rotação exigem cuidado; HMAC não impede sozinho replay. O trade-off compartilha segredo com o cliente para evitar infraestrutura de chaves públicas.

## Integração e limites

Encoding da assinatura, armazenamento da secret e modo de assinar durante a janela de duas chaves não foram definidos. Ver RFC-OPEN-03 e FDD-SEC-02; não tratar o grace period como protocolo já resolvido.

## Rastreabilidade

ID: ADR-003. Fontes: [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:23] Sofia. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
