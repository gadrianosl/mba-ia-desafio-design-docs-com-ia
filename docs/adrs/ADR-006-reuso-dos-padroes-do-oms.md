# ADR-006 — Reuso dos padrões do OMS

## Status

Aceita na reunião; lacunas de implementação permanecem sujeitas à revisão no RFC.
Registro documental: 2026-10-14. A transcrição não informa a data calendário da reunião.

## Contexto

O OMS possui convenções de módulos, validação, autorização, erros e logging que o novo domínio deve respeitar.

## Decisão

Organizar o futuro módulo em src/modules/webhooks com controller, service, repository, routes e schemas Zod. Reutilizar AppError, Pino, middleware central e requireRole. Erros específicos do domínio usam WEBHOOK_.

## Alternativas Consideradas

Framework de erros ou logger novo é alternativa plausível, não uma proposta formal da reunião; foi preterida pelo reuso máximo decidido. A reunião explicitamente recusa adicionar logger ([09:29] Bruno).

## Consequências

Positivas: consistência de manutenção e respostas; menos dependências. Negativas: convenções atuais têm limites, incluindo erros genéricos em validação. O trade-off mantém compatibilidade e exige adaptação local onde WEBHOOK_ for necessário.

## Integração e limites

Arquivos atuais: `src/shared/errors/app-error.ts`; `src/middlewares/error.middleware.ts`; `src/middlewares/validate.middleware.ts`; `src/shared/logger/index.ts`; `src/middlewares/auth.middleware.ts`. validate converte Zod em VALIDATION_ERROR; não afirmar que já gera WEBHOOK_INVALID_URL. JWT tem sub/email/role, sem customerId.

## Rastreabilidade

ID: ADR-006. Fontes: [09:27] Bruno; [09:28] Bruno; [09:29] Larissa; [09:30] Larissa. Ver [Tracker](../TRACKER.md) e [RFC](../RFC.md).
