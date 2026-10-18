# Tracker de rastreabilidade

Cada ID substantivo dos documentos está coberto. Fontes identificam evidências da
reunião ou padrões reais; elas não convertem propostas derivadas em decisões aprovadas.
Campos, códigos adicionais, testes e avaliações de risco são explicitamente marcados
como derivados nos documentos. Exemplos usam dados fictícios, não requisitos novos.

Cobertura verificável por IDs: **124/124 (100%)**. Esta contagem
mede unidades identificadas; a revisão semântica também verificou itens internos,
propostas e exclusões. Total: **131 linhas**, **107 TRANSCRICAO (81.7%)**,
**24 CODIGO**. Linhas EVID complementam as fontes de contratos e ADRs.
A porcentagem por IDs não substitui a exigência de pelo menos 80% de todos os itens
identificáveis: subitens da mesma unidade são cobertos pelas fontes agrupadas na linha.

Formato [hh:mm] preserva os horários literais da reunião; não são minutos transcorridos.
Caminhos CODIGO existem no repositório. Caminhos futuros são identificados como planos.
Para navegar, use o ID na busca do documento indicado. Datas de exemplos não são datas
prometidas de lançamento. Metadados autor/data e relato do processo não são decisões da reunião.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CTX-01 | docs/PRD.md | Contexto / escopo | Notificações outbound de mudanças de status | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos |
| PRD-PROB-01 | docs/PRD.md | Contexto / escopo | Três B2B, custo de polling e risco Atlas; prazo a confirmar | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos |
| PRD-USER-01 | docs/PRD.md | Contexto / escopo | Integração B2B, filtros, operadores e ADMIN | TRANSCRICAO | [09:00] Marcos; [09:32] Larissa; [09:33] Marcos; [09:36] Sofia |
| PRD-MET-01 | docs/PRD.md | Contexto / escopo | Expectativa abaixo de 10s; medição proposta, sem SLA definido | TRANSCRICAO | [09:02] Marcos |
| PRD-SCOPE-01 | docs/PRD.md | Contexto / escopo | API de configuração, filtros, entregas e recuperação | TRANSCRICAO | [09:31] Marcos; [09:34] Marcos; [09:35] Diego |
| PRD-OUT-01 | docs/PRD.md | Contexto / escopo | Email explicitamente adiado; esclarecimento após revisão do aluno, sem justificativa técnica definida na fala | TRANSCRICAO | [09:37] Marcos; [09:37] Larissa |
| PRD-OUT-02 | docs/PRD.md | Contexto / escopo | Dashboard fora da fase | TRANSCRICAO | [09:40] Larissa |
| PRD-OUT-03 | docs/PRD.md | Contexto / escopo | Rate limiting para observar e decidir | TRANSCRICAO | [09:39] Larissa |
| PRD-OUT-04 | docs/PRD.md | Contexto / escopo | Escala multi-worker futura | TRANSCRICAO | [09:13] Diego |
| PRD-OUT-05 | docs/PRD.md | Contexto / escopo | Arquivamento futuro; 30 dias não fechado | TRANSCRICAO | [09:08] Diego |
| PRD-OUT-06 | docs/PRD.md | Contexto / escopo | Somente outbound | TRANSCRICAO | [09:02] Marcos |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar URL/status e gerar secret | TRANSCRICAO | [09:31] Marcos; [09:32] Larissa |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Editar configuração; efeitos sobre pendentes não definidos na reunião | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Remover cadastro da seleção de novos eventos; destino dos pendentes em aberto | TRANSCRICAO | [09:33] Bruno; [09:34] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Listar por customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Filtrar na inserção; sem interessados não há evento, inclusive após remoção de todos os cadastros | TRANSCRICAO | [09:33] Marcos; [09:34] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Outbox no mesmo commit, rollback se falhar | TRANSCRICAO | [09:40] Bruno; [09:41] Diego |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Worker separado, polling 2s | TRANSCRICAO | [09:09] Diego; [09:11] Diego |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Retry/backoff/DLQ com ambiguidade explícita | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Replay ADMIN auditado | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Histórico de payload, resposta, resultado e duração | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Rotação com sobreposição 24h | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | At-least-once e deduplicação pelo cliente | TRANSCRICAO | [09:25] Diego; [09:26] Larissa |
| PRD-FR-13 | docs/PRD.md | Requisito Funcional | JWT de usuário; customer explícito; roles autenticadas | TRANSCRICAO | [09:32] Larissa; [09:37] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | HTTPS/HMAC e secret individual | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:23] Sofia |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | 64KB sem truncamento, unidade a confirmar | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Timeout de 10s gera falha | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Snapshot enxuto sem items | TRANSCRICAO | [09:43] Diego; [09:52] Larissa |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Ordenação por pedido no single-worker; lacuna de retry | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | Mesma stack, MySQL e padrões atuais | TRANSCRICAO | [09:07] Diego; [09:30] Larissa |
| PRD-DEC-01 | docs/PRD.md | Trade-off / dependência / risco derivado | Síntese das decisões e seus custos | TRANSCRICAO | [09:04] Bruno; [09:25] Diego; [09:48] Larissa |
| PRD-DEP-01 | docs/PRD.md | Trade-off / dependência / risco derivado | Receptor, portal e revisão de segurança em três sprints | TRANSCRICAO | [09:26] Marcos; [09:46] Sofia; [09:47] Larissa |
| PRD-RISK-01 | docs/PRD.md | Trade-off / dependência / risco derivado | Indisponibilidade: probabilidade/impacto são análise proposta | TRANSCRICAO | [09:16] Diego |
| PRD-RISK-02 | docs/PRD.md | Trade-off / dependência / risco derivado | Vazamento: avaliação qualitativa proposta | TRANSCRICAO | [09:21] Sofia; [09:22] Diego |
| PRD-RISK-03 | docs/PRD.md | Trade-off / dependência / risco derivado | Backlog: avaliação qualitativa proposta | TRANSCRICAO | [09:07] Bruno; [09:08] Diego; [09:38] Diego |
| PRD-RISK-04 | docs/PRD.md | Trade-off / dependência / risco derivado | Duplicatas: avaliação qualitativa proposta | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| PRD-AC-01 | docs/PRD.md | Critério / teste derivado | Validar CRUD e JWT | TRANSCRICAO | [09:31] Marcos; [09:33] Bruno; [09:37] Sofia |
| PRD-AC-02 | docs/PRD.md | Critério / teste derivado | Validar filtro e atomicidade | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno |
| PRD-AC-03 | docs/PRD.md | Critério / teste derivado | Medir envio assinado e latência | TRANSCRICAO | [09:02] Marcos; [09:20] Sofia |
| PRD-AC-04 | docs/PRD.md | Critério / teste derivado | Validar falhas, DLQ e replay ADMIN | TRANSCRICAO | [09:17] Larissa; [09:36] Sofia; [09:42] Diego |
| PRD-AC-05 | docs/PRD.md | Critério / teste derivado | Validar sobreposição na rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-06 | docs/PRD.md | Critério / teste derivado | Validar histórico e identidade | TRANSCRICAO | [09:25] Diego; [09:34] Marcos |
| PRD-AC-07 | docs/PRD.md | Critério / teste derivado | Rejeitar URL insegura e corpo excessivo | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa |
| PRD-TEST-01 | docs/PRD.md | Critério / teste derivado | Plano de testes derivado dos contratos e invariantes | TRANSCRICAO | [09:40] Bruno; [09:42] Diego; [09:46] Larissa |
| RFC-PROP-01 | docs/RFC.md | Proposta / contexto | Arquitetura consolidada, sem detalhes de implementação | TRANSCRICAO | [09:48] Larissa |
| RFC-CTX-01 | docs/RFC.md | Proposta / contexto | Dor B2B, meta e prazo comercial a confirmar | TRANSCRICAO | [09:00] Marcos; [09:02] Marcos; [09:45] Marcos |
| RFC-PROP-02 | docs/RFC.md | Proposta / contexto | Publicação tx, filtro e processo independente | TRANSCRICAO | [09:11] Diego; [09:34] Bruno; [09:41] Bruno |
| RFC-PROP-03 | docs/RFC.md | Proposta / contexto | Configuração, autenticação e limites de customer | TRANSCRICAO | [09:21] Sofia; [09:32] Larissa; [09:37] Sofia |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada / trade-off | HTTP síncrono bloqueia transação | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada / trade-off | Redis exige infraestrutura adicional | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada / trade-off | Trigger não notifica worker externo | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada / trade-off | Exactly-once exige coordenação bilateral | TRANSCRICAO | [09:25] Diego |
| RFC-OPEN-01 | docs/RFC.md | Questão aberta / lacuna identificada | Contagem de tentativas versus cinco esperas | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:17] Larissa |
| RFC-OPEN-02 | docs/RFC.md | Questão aberta / lacuna identificada | Ordenação com retries e desempate não definidos | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:17] Larissa |
| RFC-OPEN-03 | docs/RFC.md | Questão aberta / lacuna identificada | Protocolo de rotação/encoding ausente | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| RFC-OPEN-04 | docs/RFC.md | Questão aberta / lacuna identificada | Recuperação pós-crash não especificada | TRANSCRICAO | [09:11] Diego; [09:24] Diego; [09:25] Diego |
| RFC-OPEN-05 | docs/RFC.md | Questão aberta / lacuna identificada | Rate limiting explicitamente adiado | TRANSCRICAO | [09:38] Diego; [09:39] Larissa |
| RFC-OPEN-06 | docs/RFC.md | Questão aberta / lacuna identificada | Arquivamento explicitamente adiado | TRANSCRICAO | [09:08] Diego |
| RFC-OPEN-07 | docs/RFC.md | Questão aberta / lacuna identificada | Autorização por customer sem modelo definido | TRANSCRICAO | [09:32] Bruno; [09:32] Larissa; [09:37] Sofia |
| RFC-OPEN-08 | docs/RFC.md | Questão aberta / lacuna identificada | Detalhes HTTP, batch e remoção pendente sem definição | TRANSCRICAO | [09:08] Diego; [09:33] Bruno; [09:42] Diego |
| RFC-RISK-01 | docs/RFC.md | Risco / dependência derivada | Carga MySQL e latência sob backlog | TRANSCRICAO | [09:07] Bruno; [09:08] Diego; [09:09] Diego |
| RFC-RISK-02 | docs/RFC.md | Risco / dependência derivada | Deduplicação, segurança e revisão antes de deploy | TRANSCRICAO | [09:25] Diego; [09:46] Sofia; [09:47] Larissa |
| FDD-CTX-01 | docs/FDD.md | Contexto / objetivo / escopo | Preencher falta de notificações sem HTTP no commit | TRANSCRICAO | [09:00] Marcos; [09:04] Bruno; [09:40] Bruno |
| FDD-OBJ-01 | docs/FDD.md | Contexto / objetivo / escopo | Atomicidade, snapshot e envio assíncrono | TRANSCRICAO | [09:02] Marcos; [09:06] Diego; [09:52] Larissa |
| FDD-SCOPE-01 | docs/FDD.md | Contexto / objetivo / escopo | Somente outbound; criação não foi evento acordado | TRANSCRICAO | [09:02] Marcos; [09:37] Larissa; [09:40] Larissa; [09:40] Bruno |
| FDD-DATA-01 | docs/FDD.md | Detalhamento técnico proposto / derivado | Modelo proposto para configuração, outbox, tentativas e DLQ | TRANSCRICAO | [09:08] Diego; [09:18] Diego; [09:21] Bruno; [09:34] Marcos; [09:51] Larissa |
| FDD-DATA-02 | docs/FDD.md | Detalhamento técnico proposto / derivado | Fan-out e identidade de entrega/replay a revisar | TRANSCRICAO | [09:25] Diego; [09:33] Marcos; [09:34] Bruno; [09:18] Diego |
| FDD-FLOW-01 | docs/FDD.md | Detalhamento técnico proposto / derivado | Fluxo de publicação com tx e snapshot | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno; [09:41] Bruno; [09:52] Larissa |
| FDD-FLOW-02 | docs/FDD.md | Detalhamento técnico proposto / derivado | Polling, batch e processo próprio | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:11] Larissa; [09:30] Bruno |
| FDD-FLOW-03 | docs/FDD.md | Detalhamento técnico proposto / derivado | Seleção vencida, claim e conclusão são proposta derivada | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:42] Diego |
| FDD-FLOW-04 | docs/FDD.md | Detalhamento técnico proposto / derivado | Política HTTP 2xx/falha e redirects é proposta, não decisão | TRANSCRICAO | [09:14] Larissa; [09:15] Diego; [09:42] Diego |
| FDD-RETRY-01 | docs/FDD.md | Detalhamento técnico proposto / derivado | Backoff e DLQ; contador bloqueado por ambiguidade | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:18] Diego |
| FDD-RETRY-02 | docs/FDD.md | Detalhamento técnico proposto / derivado | Replay tx, identidade preservada e conflito são propostas | TRANSCRICAO | [09:18] Diego; [09:25] Diego; [09:36] Sofia |
| FDD-ORDER-01 | docs/FDD.md | Detalhamento técnico proposto / derivado | Ordem por pedido: bloqueio de sucessor é proposta pendente | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:17] Larissa |
| FDD-RECOVERY-01 | docs/FDD.md | Detalhamento técnico proposto / derivado | Recuperação de processando é proposta derivada de crash/duplicata | TRANSCRICAO | [09:11] Diego; [09:24] Diego; [09:25] Diego |
| FDD-API-00 | docs/FDD.md | Integração / padrão existente | Base /api/v1, convenções e exemplo de montagem | CODIGO | src/app.ts |
| FDD-API-01 | docs/FDD.md | Contrato proposto derivado | POST de cadastro, payload e geração de secret | TRANSCRICAO | [09:31] Marcos; [09:32] Larissa |
| FDD-API-02 | docs/FDD.md | Contrato proposto derivado | PATCH de edição; efeitos em pendentes não fechados | TRANSCRICAO | [09:33] Bruno |
| FDD-API-03 | docs/FDD.md | Contrato proposto derivado | GET por customer, paginação proposta | TRANSCRICAO | [09:33] Bruno |
| FDD-API-04 | docs/FDD.md | Contrato proposto derivado | DELETE; remoção lógica é proposta pendente | TRANSCRICAO | [09:33] Bruno |
| FDD-API-05 | docs/FDD.md | Contrato proposto derivado | Histórico com payload, resposta e duração | TRANSCRICAO | [09:34] Marcos |
| FDD-API-06 | docs/FDD.md | Contrato proposto derivado | Rotação e janela 24h; conflito adicional é proposta | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia |
| FDD-API-07 | docs/FDD.md | Contrato proposto derivado | Replay ADMIN auditado; 202 e concorrência propostos | TRANSCRICAO | [09:18] Diego; [09:35] Diego; [09:36] Sofia |
| FDD-PAYLOAD-01 | docs/FDD.md | Contrato proposto derivado | Campos, headers, sem items; timestamps ilustrativos | TRANSCRICAO | [09:43] Diego; [09:44] Diego; [09:44] Sofia |
| FDD-PAYLOAD-02 | docs/FDD.md | Contrato proposto derivado | Bytes imutáveis, unidade e rejeição no tx são propostas | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa; [09:40] Bruno; [09:52] Larissa |
| FDD-ERR-01 | docs/FDD.md | Contrato proposto derivado | Matriz de códigos específicos proposta a partir do padrão | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| FDD-ERR-02 | docs/FDD.md | Integração / padrão existente | AppError já serializa; erros Zod/Prisma são genéricos | CODIGO | src/middlewares/error.middleware.ts; src/middlewares/validate.middleware.ts; src/middlewares/auth.middleware.ts |
| FDD-SEC-01 | docs/FDD.md | Segurança / resiliência / observabilidade derivada | Secret individual, geração e revisão de armazenamento/destinos | TRANSCRICAO | [09:19] Sofia; [09:21] Sofia; [09:23] Sofia; [09:46] Sofia |
| FDD-SEC-02 | docs/FDD.md | Segurança / resiliência / observabilidade derivada | Proposta de encoding e transição de chave; revisão obrigatória | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| FDD-SEC-03 | docs/FDD.md | Segurança / resiliência / observabilidade derivada | Timestamp do envio não é coberto pelo HMAC do corpo | TRANSCRICAO | [09:22] Sofia; [09:44] Diego |
| FDD-RES-01 | docs/FDD.md | Segurança / resiliência / observabilidade derivada | Timeout, backoff, DLQ e ausência de email | TRANSCRICAO | [09:17] Larissa; [09:18] Diego; [09:37] Larissa; [09:42] Diego |
| FDD-OBS-01 | docs/FDD.md | Segurança / resiliência / observabilidade derivada | Métricas e correlação propostas; Pino e auditoria confirmados | TRANSCRICAO | [09:02] Marcos; [09:29] Bruno; [09:34] Marcos; [09:36] Sofia |
| FDD-INT-01 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/modules/orders/order.service.ts | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: prisma/schema.prisma | CODIGO | prisma/schema.prisma |
| FDD-INT-03 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/middlewares/auth.middleware.ts | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-04 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/shared/errors/app-error.ts | CODIGO | src/shared/errors/app-error.ts |
| FDD-INT-05 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/middlewares/error.middleware.ts | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-06 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/middlewares/validate.middleware.ts | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-INT-07 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/shared/logger/index.ts | CODIGO | src/shared/logger/index.ts |
| FDD-INT-08 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/config/database.ts | CODIGO | src/config/database.ts |
| FDD-INT-09 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/routes/index.ts | CODIGO | src/routes/index.ts |
| FDD-INT-10 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/app.ts | CODIGO | src/app.ts |
| FDD-INT-11 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/shared/http/response.ts | CODIGO | src/shared/http/response.ts |
| FDD-INT-12 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/modules/orders/order.status.ts | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-13 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: src/server.ts | CODIGO | src/server.ts |
| FDD-INT-14 | docs/FDD.md | Integração / padrão existente | Ponto real de integração e padrão observado: tests/orders.test.ts | CODIGO | tests/orders.test.ts |
| FDD-NEW-01 | docs/FDD.md | Caminho futuro previsto | Worker e módulo webhooks são planejados, ainda inexistentes | TRANSCRICAO | [09:11] Larissa; [09:27] Bruno; [09:28] Bruno |
| FDD-DEP-01 | docs/FDD.md | Integração / padrão existente | Versões e scripts declarados; compatibilidade atual | CODIGO | package.json |
| FDD-AC-01 | docs/FDD.md | Critério técnico / risco derivado | Testar filtro e atomicidade de pedido/histórico/estoque | TRANSCRICAO | [09:34] Bruno; [09:40] Bruno |
| FDD-AC-02 | docs/FDD.md | Critério técnico / risco derivado | Testar snapshot/HMAC/identidade | TRANSCRICAO | [09:22] Sofia; [09:25] Diego; [09:52] Larissa |
| FDD-AC-03 | docs/FDD.md | Critério técnico / risco derivado | Testar polling/timeout/retry após esclarecer contador | TRANSCRICAO | [09:09] Diego; [09:17] Larissa; [09:42] Diego |
| FDD-AC-04 | docs/FDD.md | Critério técnico / risco derivado | Testar API e limites | TRANSCRICAO | [09:23] Sofia; [09:24] Larissa; [09:31] Marcos; [09:34] Marcos |
| FDD-AC-05 | docs/FDD.md | Critério técnico / risco derivado | Testar autorização e auditoria; concorrência proposta | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| FDD-AC-06 | docs/FDD.md | Critério técnico / risco derivado | Testar recuperação, duplicata e regra de ordenação | TRANSCRICAO | [09:12] Diego; [09:13] Larissa; [09:25] Diego |
| FDD-AC-07 | docs/FDD.md | Critério técnico / risco derivado | Testar fronteira 24h e confidencialidade de chaves | TRANSCRICAO | [09:21] Sofia; [09:22] Sofia; [09:46] Sofia |
| FDD-AC-08 | docs/FDD.md | Critério técnico / risco derivado | Validar latência real com carga acordada | TRANSCRICAO | [09:02] Marcos; [09:09] Diego; [09:42] Diego |
| FDD-RISK-01 | docs/FDD.md | Critério técnico / risco derivado | Riscos de banco, capacidade, duplicata, chaves e customer | TRANSCRICAO | [09:08] Diego; [09:12] Diego; [09:21] Sofia; [09:25] Diego; [09:32] Larissa; [09:42] Diego |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão / alternativas / consequências | Outbox no MySQL; justificativas e limites no registro | TRANSCRICAO | [09:06] Diego; [09:07] Diego; [09:40] Bruno; [09:41] Diego |
| ADR-002 | docs/adrs/ADR-002-retry-com-backoff-e-dlq.md | Decisão / alternativas / consequências | Retry com backoff e DLQ; justificativas e limites no registro | TRANSCRICAO | [09:15] Diego; [09:16] Bruno; [09:17] Larissa; [09:18] Diego; [09:36] Sofia |
| ADR-003 | docs/adrs/ADR-003-hmac-e-rotacao-de-secret.md | Decisão / alternativas / consequências | HMAC e rotação de secret; justificativas e limites no registro | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:23] Sofia |
| ADR-004 | docs/adrs/ADR-004-at-least-once-e-event-id.md | Decisão / alternativas / consequências | At-least-once e identidade do evento; justificativas e limites no registro | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| ADR-005 | docs/adrs/ADR-005-worker-separado-com-polling.md | Decisão / alternativas / consequências | Worker separado com polling; justificativas e limites no registro | TRANSCRICAO | [09:08] Diego; [09:09] Diego; [09:10] Larissa; [09:11] Diego; [09:12] Diego; [09:13] Larissa; [09:30] Bruno |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Decisão / alternativas / consequências | Reuso dos padrões do OMS; justificativas e limites no registro | TRANSCRICAO | [09:27] Bruno; [09:28] Bruno; [09:29] Larissa; [09:30] Larissa |
| ADR-007 | docs/adrs/ADR-007-snapshot-do-payload-na-outbox.md | Decisão / alternativas / consequências | Snapshot do payload na outbox; justificativas e limites no registro | TRANSCRICAO | [09:43] Diego; [09:51] Larissa; [09:52] Larissa; [09:52] Diego |
| EVID-API-01 | docs/FDD.md | Integração / padrão existente | Status 201/200/204 e respostas diretas nos contratos propostos | CODIGO | src/modules/customers/customer.controller.ts |
| EVID-API-02 | docs/FDD.md | Integração / padrão existente | Envelope de paginação para listagem e histórico | CODIGO | src/shared/http/response.ts |
| EVID-API-03 | docs/FDD.md | Integração / padrão existente | Limites de paginação e enum reutilizáveis | CODIGO | src/modules/orders/order.schemas.ts |
| EVID-SEC-01 | docs/FDD.md | Integração / padrão existente | Pino atual não contém redação de secret | CODIGO | src/shared/logger/index.ts |
| EVID-AUTH-01 | docs/PRD.md | Integração / padrão existente | Token de usuário não contém customerId | CODIGO | src/middlewares/auth.middleware.ts |
| EVID-ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Integração / padrão existente | Transação real de changeStatus e provider MySQL | CODIGO | src/modules/orders/order.service.ts; prisma/schema.prisma |
| EVID-ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-oms.md | Integração / padrão existente | Classes de erro e padrões reutilizados | CODIGO | src/shared/errors/app-error.ts; src/middlewares/validate.middleware.ts; src/shared/logger/index.ts |
